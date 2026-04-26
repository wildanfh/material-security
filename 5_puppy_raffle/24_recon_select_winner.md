# Manual Review: Analisis Fungsi selectWinner

## Pendahuluan

Kita melanjutkan penelusuran manual pada protokol `PuppyRaffle`. Sejauh ini kita telah menemukan:
- `enterRaffle`: Kerentanan **DoS** (pemborosan gas).
- `refund`: Kerentanan **Re-entrancy**.
- `getActivePlayerIndex`: Edge case pada index 0.

Sekarang, kita fokus pada fungsi utama protokol: `selectWinner`.

---

## 1. Bedah Fungsi `selectWinner`

Fungsi ini bertanggung jawab untuk menentukan pemenang, mendistribusikan hadiah, dan mencetak NFT puppy.

```javascript
function selectWinner() external {
    // 1. CHECKS
    require(block.timestamp >= raffleStartTime + raffleDuration, "PuppyRaffle: Raffle not over");
    require(players.length >= 4, "PuppyRaffle: Need at least 4 players");

    // 2. RANDOMNESS (Pseudo-Random)
    uint256 winnerIndex =
        uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;
    address winner = players[winnerIndex];

    // 3. ACCOUNTING & FEES
    uint256 totalAmountCollected = players.length * entranceFee;
    uint256 prizePool = (totalAmountCollected * 80) / 100;
    uint256 fee = (totalAmountCollected * 20) / 100;
    totalFees = totalFees + uint64(fee); // Perhatikan casting uint64 ini!

    // 4. RARITY & MINTING
    uint256 tokenId = totalSupply();
    uint256 rarity = uint256(keccak256(abi.encodePacked(msg.sender, block.difficulty))) % 100;
    // ... logic penentuan rarity ...

    // 5. RESET STATE
    delete players;
    raffleStartTime = block.timestamp;
    previousWinner = winner;

    // 6. INTERACTIONS (External Call)
    (bool success,) = winner.call{value: prizePool}("");
    require(success, "PuppyRaffle: Failed to send prize pool to winner");
    _safeMint(winner, tokenId);
}
```

---

## 2. Catatan Audit (@Audit)

Selama mereview fungsi ini, ada beberapa "bendera merah" (red flags) yang muncul di pikiran auditor:

| Area Fokus | Pertanyaan / Observasi |
| :--- | :--- |
| **Pola CEI** | Apakah fungsi ini mengikuti pola CEI? Ada `_safeMint` yang dipanggil *setelah* `call`. Apa dampaknya? |
| **Randomness** | Pemilihan pemenang menggunakan `block.timestamp` dan `block.difficulty`. Apakah ini benar-benar acak atau bisa dimanipulasi (Predictable)? |
| **Integer Casting** | `totalFees` diupdate dengan `uint64(fee)`. Apakah ada risiko *overflow* jika fee sangat besar? |
| **Akses Kontrol** | Fungsi ini `external` dan tidak punya modifier akses (seperti `onlyOwner`). Siapa saja bisa memanggilnya asalkan waktu durasi telah habis. |
| **Re-entrancy** | Apakah `_safeMint` dapat memicu re-entrancy kembali ke dalam fungsi ini atau fungsi lain seperti `refund`? |

---

## 3. Masalah Utama: Pseudo-Randomness

Salah satu poin paling kritis di sini adalah penggunaan data *on-chain* untuk menghasilkan angka acak.
- `block.timestamp`: Bisa sedikit dimanipulasi oleh miner/validator.
- `block.difficulty`: Tidak lagi bisa diandalkan (terutama setelah The Merge di Ethereum).
- `msg.sender`: Penyerang bisa mencoba berbagai alamat untuk mendapatkan hasil hash yang diinginkan.

---

## 4. Tantangan (Challenge)

Ada bug **masif** yang melibatkan interaksi antara fungsi `refund` dan `selectWinner`. 

> **Tantangan:** Bisakah Anda menemukan skenario di mana seorang pemain bisa melakukan refund *sekaligus* mempengaruhi hasil pemilihan pemenang, atau melakukan eksploitasi saat transisi pemilihan pemenang terjadi?

---

## Kesimpulan

Fungsi `selectWinner` adalah jantung dari protokol ini, namun ia penuh dengan potensi masalah keamanan. Fokus kita berikutnya adalah membuktikan apakah mekanisme "keberuntungan" (randomness) di sini benar-benar adil atau bisa dicurangi.
