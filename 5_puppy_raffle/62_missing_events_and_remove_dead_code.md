# Missing Events di PuppyRaffle

## Deskripsi

Beberapa state changes dalam kontrak tidak memancarkan event, sehingga menyulitkan sistem eksternal (front-end, subgraph, dll) untuk melacak perubahan secara akurat.

## Lokasi yang Bermasalah

- `PuppyRaffle::totalFees` di dalam fungsi `selectWinner`
- `PuppyRaffle::raffleStartTime` di dalam fungsi `selectWinner`
- `PuppyRaffle::totalFees` di dalam fungsi `withdrawFees`

## Rekomendasi Mitigasi

Tambahkan event setiap kali terjadi perubahan state penting. Contoh:

```solidity
event FeesUpdated(uint256 newTotalFees);
event RaffleReset(uint256 newStartTime);
```

---

# Dead Code: `_isActivePlayer` Tidak Pernah Digunakan

## Deskripsi

Fungsi `_isActivePlayer` dideklarasikan tetapi tidak pernah dipanggil di mana pun dalam kontrak. Ini termasuk dead code yang dapat dihapus.

## Kode

```solidity
function _isActivePlayer() internal view returns (bool) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == msg.sender) {
            return true;
        }
    }
    return false;
}
```

## Rekomendasi Mitigasi

Hapus fungsi tersebut untuk menghemat bytecode dan mengurangi kompleksitas kode.

```diff
-    function _isActivePlayer() internal view returns (bool) {
-        // ...
-    }
```