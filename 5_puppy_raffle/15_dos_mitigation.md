# Recommended Mitigation untuk Denial of Service (DoS) di PuppyRaffle

## Pendahuluan

Setelah mengidentifikasi kerentanan **Denial of Service (DoS)** pada fungsi `PuppyRaffle::enterRaffle` (akibat *nested loop* O(n²) untuk pengecekan duplikat), langkah selanjutnya adalah memberikan **rekomendasi mitigasi** yang praktis dan tidak mengubah fungsionalitas utama protokol secara drastis.

> Prinsip: Pertahankan fungsionalitas asli sebisa mungkin. Jika perubahan fungsi diperlukan, jelaskan alasannya dengan jelas.

---

## Tiga Rekomendasi Mitigasi

### 1. Mengizinkan Duplikat (Allow Duplicates)

**Pertimbangan:**  
- Pemeriksaan duplikat hanya mencegah alamat dompet yang sama masuk beberapa kali, tetapi tidak mencegah orang yang sama menggunakan dompet berbeda.  
- Karena pengguna dapat membuat banyak alamat dompet, pemeriksaan duplikat tidak memberikan keamanan berarti.

**Rekomendasi:**  
Hapus seluruh pengecekan duplikat. Ini akan menghilangkan *nested loop* sepenuhnya dan membuat biaya gas konstan (tidak meningkat seiring jumlah pemain).

```diff
-        // Check for duplicates
-        for (uint256 i = 0; i < players.length; i++) {
-            for (uint256 j = i + 1; j < players.length; j++) {
-                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
-            }
-        }
```

---

2. Menggunakan Mapping untuk Pengecekan Duplikat (Constant Time)

Pendekatan:
Gunakan mapping(address => uint256) untuk mencatat raffleId setiap pemain. Dengan ini, pengecekan duplikat menjadi O(1) (konstan) tidak bergantung pada jumlah pemain.

Perubahan yang Diperlukan:

· Tambahkan variabel state:
  ```solidity
  mapping(address => uint256) public addressToRaffleId;
  uint256 public raffleId = 0;
  ```
· Di enterRaffle, setelah menambahkan pemain baru, catat raffleId mereka:
  ```solidity
  for (uint256 i = 0; i < newPlayers.length; i++) {
      players.push(newPlayers[i]);
      addressToRaffleId[newPlayers[i]] = raffleId;
  }
  ```
· Ganti pengecekan duplikat (hanya untuk pemain baru, bukan seluruh array):
  ```solidity
  for (uint256 i = 0; i < newPlayers.length; i++) {
      require(addressToRaffleId[newPlayers[i]] != raffleId, "PuppyRaffle: Duplicate player");
  }
  ```
· Di selectWinner, tingkatkan raffleId setelah undian selesai (sebelum undian berikutnya):
  ```solidity
  raffleId = raffleId + 1;
  ```

Keuntungan:

· Biaya gas tetap rendah meskipun ribuan pemain sudah masuk.
· Fungsionalitas asli (mencegah duplikat dalam satu undian) tetap terjaga.

---

3. Menggunakan OpenZeppelin EnumerableSet Library

Pendekatan:
Manfaatkan library EnumerableSet dari OpenZeppelin. Library ini menyediakan struktur data set yang dapat diiterasi dan memiliki operasi pengecekan keanggotaan dalam waktu konstan (O(1)).

Contoh penggunaan:

```solidity
import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

contract PuppyRaffle {
    using EnumerableSet for EnumerableSet.AddressSet;
    EnumerableSet.AddressSet private playersSet;

    function enterRaffle(address[] memory newPlayers) public payable {
        // ... validasi value
        for (uint256 i = 0; i < newPlayers.length; i++) {
            require(!playersSet.contains(newPlayers[i]), "Duplicate player");
            playersSet.add(newPlayers[i]);
            players.push(newPlayers[i]);
        }
        // ...
    }
}
```

Keuntungan:

· Menggunakan library yang sudah teruji dan aman.
· Operasi pengecekan dan penambahan efisien (O(1) average).

---

Laporan Akhir (Lengkap dengan Mitigasi)

<details>
<summary>DoS Writeup – Full Report</summary>

```markdown
### [M-#] Looping through players array to check for duplicates in `PuppyRaffle::enterRaffle` is a potential denial of service (DoS) attack, incrementing gas costs for future entrants

**Description:** The `PuppyRaffle::enterRaffle` function loops through the `players` array to check for duplicates. However, the longer the `players` array is, the more checks a new player will have to make. This means the gas costs for players who enter right when the raffle starts will be dramatically lower than those who enter later. Every additional address in the `players` array is an additional check the loop will have to make.

```solidity
// @audit DoS Attack
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

Impact: The gas costs for raffle entrants will greatly increase as more players enter the raffle, discouraging later users from entering and causing a rush at the start of a raffle to be one of the first entrants in queue. An attacker might make the players array so big that no one else enters, guaranteeing themselves the win.

Proof of Concept:
If we have 2 sets of 100 players enter, the gas costs will be as such:

· 1st 100 players: ~6,252,048 gas
· 2nd 100 players: ~18,068,138 gas

This is more than 3x more expensive for the second 100 players.

<details>
<summary>Proof of Code</summary>

```solidity
function testDenialOfService() public {
    vm.txGasPrice(1);
    uint256 playersNum = 100;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < players.length; i++) {
        players[i] = address(i);
    }
    uint256 gasStart = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * players.length}(players);
    uint256 gasEnd = gasleft();
    uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;

    address[] memory playersTwo = new address[](playersNum);
    for (uint256 i = 0; i < playersTwo.length; i++) {
        playersTwo[i] = address(i + playersNum);
    }
    uint256 gasStartTwo = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * playersTwo.length}(playersTwo);
    uint256 gasEndTwo = gasleft();
    uint256 gasUsedSecond = (gasStartTwo - gasEndTwo) * tx.gasprice;

    assert(gasUsedSecond > gasUsedFirst);
}
```

</details>

Recommended Mitigations:
There are several options:

1. Allow duplicates – Remove the duplicate check entirely, as users can create multiple wallet addresses anyway. This eliminates the loop.
2. Use a mapping for O(1) duplicate check – Track each player’s raffle ID using a mapping and increment the raffle ID after each draw. Only check new players, not the entire array.
3. Use OpenZeppelin’s EnumerableSet – Leverage the battle-tested library for constant-time set operations.

```

</details>

---

## Kesimpulan

- **Mitigasi harus mempertahankan fungsi asli** sebisa mungkin, tetapi jika perubahan diperlukan, jelaskan alasannya.
- **Pendekatan mapping** (opsi #2) adalah keseimbangan terbaik antara efisiensi dan pelestarian fungsionalitas (tetap mencegah duplikat dalam satu undian).
- **Mengizinkan duplikat** (opsi #1) adalah solusi paling sederhana dan paling murah gas, tetapi mengubah perilaku protokol (duplikat diizinkan).
- **EnumerableSet** (opsi #3) adalah solusi teraman karena menggunakan library standar OpenZeppelin.