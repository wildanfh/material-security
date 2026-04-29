# Temuan MEV pada Fungsi Refund (Ditunda)

Dalam proses audit PuppyRaffle, terdapat satu catatan temuan terkait **MEV (Miner Extractable Value)** pada fungsi `refund`.

## Fungsi yang Terdampak

```javascript
function refund(uint256 playerIndex) public {
    // @Audit: MEV
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
    require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");
    
    // slither-disable-next-line reentrancy-no-eth,reentrancy-events
    payable(msg.sender).sendValue(entranceFee);

    players[playerIndex] = address(0);
    emit RaffleRefunded(playerAddress);
}
```

## Status Temuan: Ditunda (Skipped)

Meskipun terdapat indikasi kerentanan MEV pada logika fungsi ini, instruktur memutuskan untuk **melewatkan (skip)** pembahasan detailnya untuk saat ini.

### Alasan Penundaan:
1.  **Kompleksitas**: MEV adalah topik tingkat lanjut yang memerlukan pemahaman mendalam tentang cara kerja *mempool* dan bagaimana validator/miner mengurutkan transaksi.
2.  **Kurikulum**: Topik MEV akan dibahas secara khusus dan lebih mendalam pada bagian selanjutnya dalam kursus ini.

## Catatan untuk Auditor
Saat melakukan audit manual, jika Anda menemukan pola yang berpotensi terkena serangan MEV (seperti manipulasi urutan transaksi untuk keuntungan finansial), penting untuk menandainya (tagging) seperti `// @Audit: MEV` agar bisa diinvestigasi lebih lanjut di fase audit yang lebih dalam.
