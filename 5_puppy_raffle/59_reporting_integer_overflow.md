# Integer Overflow dan Unsafe Casting di PuppyRaffle

## Pendahuluan

Pada audit Puppy Raffle, ditemukan dua kerentanan terkait penanganan angka dan tipe data di fungsi `selectWinner`:

1. **Integer Overflow** pada penjumlahan `totalFees`
2. **Unsafe Casting** dari `uint256` ke `uint64`

Keduanya berpotensi menyebabkan kehilangan fee yang terkumpul di kontrak.

---

## Kerentanan 1: Integer Overflow

### Kode Rentan

```solidity
totalFees = totalFees + uint64(fee);
```

### Root Cause

- Kontrak menggunakan versi Solidity `^0.7.6` yang secara **default tidak memiliki pemeriksaan overflow**.
- Variabel `totalFees` bertipe `uint64` yang memiliki batas maksimum `18.446.744.073.709.551.615` (~18 ETH).
- Jika `fee` membuat `totalFees` melebihi batas tersebut, overflow akan terjadi dan nilainya menjadi sangat kecil (wrap around ke 0).

### Severity

| Faktor | Penilaian | Alasan |
|--------|-----------|--------|
| **Impact** | High | Fee bisa hilang atau stuck di kontrak, tidak dapat ditarik. |
| **Likelihood** | High | Semakin sukses protokol, semakin besar kemungkinan overflow. |

**Severity:** **High** (`[H-3]`)

### Dampak (Impact)

- `feeAddress` tidak dapat mengumpulkan fee yang benar.
- Fee yang seharusnya masuk menjadi **permanen stuck** di kontrak (tidak bisa ditarik).
- Fungsi `withdrawFees` memiliki pemeriksaan `require(address(this).balance == uint256(totalFees), ...)` yang akan gagal karena saldo kontrak tidak sama dengan `totalFees` yang sudah overflow.

### Proof of Concept (PoC)

Langkah-langkah dalam kode test Foundry:

1. Selesaikan undian dengan 4 pemain → kumpulkan fee awal.
2. 89 pemain baru masuk ke undian berikutnya.
3. Selesaikan undian kedua.
4. `totalFees` mengalami overflow dan nilainya **lebih kecil** dari sebelumnya.
5. Fungsi `withdrawFees` gagal karena pemeriksaan keseimbangan.

```solidity
function testTotalFeesOverflow() public playersEntered {
    // Raffle pertama: 4 pemain
    vm.warp(block.timestamp + duration + 1);
    vm.roll(block.number + 1);
    puppyRaffle.selectWinner();
    uint256 startingTotalFees = puppyRaffle.totalFees();
    // startingTotalFees = 800000000000000000 (0.8 ETH)

    // 89 pemain baru masuk
    uint256 playersNum = 89;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        players[i] = address(i);
    }
    puppyRaffle.enterRaffle{value: entranceFee * playersNum}(players);

    // Selesaikan undian kedua
    vm.warp(block.timestamp + duration + 1);
    vm.roll(block.number + 1);
    puppyRaffle.selectWinner();

    uint256 endingTotalFees = puppyRaffle.totalFees();
    assert(endingTotalFees < startingTotalFees); // Overflow terjadi

    // Withdraw gagal
    vm.prank(puppyRaffle.feeAddress());
    vm.expectRevert("PuppyRaffle: There are currently players active!");
    puppyRaffle.withdrawFees();
}
```

### Rekomendasi Mitigasi

| Opsi | Perubahan | Penjelasan |
|------|-----------|------------|
| **1. Upgrade Solidity** | `pragma solidity ^0.8.18;` | Versi 0.8.x ke atas secara otomatis memeriksa overflow. |
| **2. Gunakan SafeMath** | `import "@openzeppelin/contracts/utils/math/SafeMath.sol";` | Untuk versi lama, gunakan library SafeMath. |
| **3. Ubah tipe `totalFees` ke `uint256`** | `uint256 public totalFees = 0;` | Hapus `uint64` casting, gunakan `uint256` langsung. |
| **4. Hapus balance check** | Hapus baris `require(address(this).balance == uint256(totalFees), ...);` | Pemeriksaan ini tidak diperlukan dan mengganggu penarikan. |

**Rekomendasi terbaik:** Upgrade ke Solidity 0.8.18 **dan** ubah `totalFees` menjadi `uint256`.

```diff
- pragma solidity ^0.7.6;
+ pragma solidity ^0.8.18;

- uint64 public totalFees = 0;
+ uint256 public totalFees = 0;

- totalFees = totalFees + uint64(fee);
+ totalFees = totalFees + fee;
```

---

## Kerentanan 2: Unsafe Casting (uint256 → uint64)

### Kode Rentan

```solidity
totalFees = totalFees + uint64(fee);
```

### Root Cause

- `fee` dihitung dari `totalAmountCollected * 20 / 100` yang bertipe `uint256`.
- Nilai `fee` dapat melebihi `type(uint64).max` (~18 ETH).
- Ketika di-cast ke `uint64`, nilai tersebut akan **dipotong (truncated)** – hanya 64 bit terbawah yang dipertahankan.
- Akibatnya, `totalFees` bertambah dengan jumlah yang jauh lebih kecil dari seharusnya.

### Contoh di Chisel

```javascript
uint256 max = type(uint64).max;      // 18446744073709551615
uint256 fee = max + 1;               // 18446744073709551616
uint64(fee);                         // 0 (overflow/truncation)
```

### Severity

- **Impact:** Medium – fee bisa hilang, tetapi tidak langsung mematikan protokol.
- **Likelihood:** Medium – diperlukan total fee yang dikumpulkan > 18 ETH.
- **Severity:** **Medium** (`[M-3]`)

### Dampak (Impact)

Sama seperti integer overflow: `feeAddress` tidak menerima fee yang benar, dan sebagian fee mungkin stuck di kontrak.

### Rekomendasi Mitigasi

Hilangkan casting dengan mengubah `totalFees` menjadi `uint256`:

```diff
- uint64 public totalFees = 0;
+ uint256 public totalFees = 0;

- totalFees = totalFees + uint64(fee);
+ totalFees = totalFees + fee;
```

> Komentar dalam kode menyebutkan *"We do some storage packing to save gas"* – tetapi penghematan gas tidak sebanding dengan risiko kehilangan fee.

---

## Kesimpulan

- **Integer overflow** (High) dan **unsafe casting** (Medium) adalah kerentanan serius dalam pengelolaan fee di Puppy Raffle.
- Keduanya berasal dari penggunaan tipe `uint64` yang terlalu kecil dan versi Solidity lama tanpa pemeriksaan overflow.
- Mitigasi utama: upgrade ke Solidity 0.8.18 dan ubah `totalFees` menjadi `uint256`, hapus casting.
- Auditor harus selalu waspada terhadap *type casting* yang tidak aman, terutama dari `uint256` ke tipe yang lebih kecil.
