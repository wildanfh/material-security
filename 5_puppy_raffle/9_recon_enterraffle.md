# Recon: Analisis Fungsi enterRaffle (Lanjutan)

## Pendahuluan

Melanjutkan analisis fungsi `enterRaffle` – fungsi utama entry point puppy raffle. Sekarang kita akan mengidentifikasi potential vulnerabilities dan pertanyaan audit.

---

## Kode Fungsi enterRaffle

```solidity
function enterRaffle(address[] memory newPlayers) public payable {
    require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
    for (uint256 i = 0; i < newPlayers.length; i++) {
        players.push(newPlayers[i]);
    }

    // Check for duplicates
    for (uint256 i = 0; i < players.length - 1; i++) {
        for (uint256 j = i + 1; j < players.length; j++) {
            require(players[i] != players[j], "PuppyRaffle: Duplicate player");
        }
    }
    emit RaffleEnter(newPlayers);
}
```

---

## Temuan dan Pertanyaan Audit

### 1. Custom Reverts (Solidity 0.7.6)

```solidity
require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
```

- **Pertanyaan:** Apakah custom reverts tersedia di Solidity 0.7.6?
- Perlu diverifikasi karena fitur ini baru tersedia di versi yang lebih baru.

### 2. Empty Array Check

```solidity
require(msg.value == entranceFee * newPlayers.length, ...)
```

- **Pertanyaan:** Apa yang terjadi jika `newPlayers.length == 0`?
- Jika array kosong passed:
  - `msg.value` harusnya 0
  - Tapi fungsi tetap jalan dan push array kosong ke `players[]`
  - Ini bisa menyebabkan masalah di downstream logic

### 3. Immutable Variable

```solidity
entranceFee
```

- `entranceFee` adalah **immutable variable**
- Diinisialisasi di constructor
- Tidak bisa diubah setelah deployment
- Perlu konfirmasi di constructor untuk validasi

### 4. Duplicate Check Logic

```solidity
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

Ini adalah bagian yang **mencurigakan** – blok kode yang "berbau" seperti ada bug:

- Check duplikat berjalan setelah semua player baru di-push ke array
- Ini berarti array yang dicek sudah termasuk player baru + player lama
- **Question:** Apakah ada case di mana loop ini bisa fail atau inefficient?
- Nested loop dengan O(n²) complexity – masalah gas.

---

## Catatan Audit Tambahan

```markdown
### @audit - enterRaffle

1. [ ] - Custom revert string digunakan - apakah support di 0.7.6?
2. [ ] - Empty array check - apa yang terjadi jika newPlayers.length == 0?
3. [ ] - Check duplikat O(n²) - consider optimization
4. [ ] - gas limit concern dengan banyak participants
```

---

## Skill "Mencium" Bug

Dengan pengalaman, auditor akan bisa "mencium" bug:

- Blok kode yang berantakan atau kompleks
- Loop bersarang yang tidak perlu
- Kondisi yang tidak jelas
- Logic yang redundan

Dalam kasus ini, nested loop untuk check duplikat adalah **red flag**:
- Complexity O(n²) – gas mahal
- Ada cara yang lebih efisien (mapping(address => bool))

---

## Kesimpulan

- Satu entry point (`enterRaffle`) sudah memberikan banyak insight
- Catat semua pertanyaan dan temuan di `notes.md`
- Perhatikan kode yang "berbau" seperti ada bug
- Jangan langsung assume aman – selalu verifikasi dengan testing
- Selanjutnya: Eksplorasi contoh exploit untuk menemukan bug yang serupa