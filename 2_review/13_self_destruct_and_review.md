## Apa Itu `selfdestruct`?

`selfdestruct` adalah keyword di Solidity yang berfungsi untuk **menghancurkan** (delete) sebuah smart contract dari blockchain. Setelah contract dihancurkan, seluruh **ETH yang tersimpan di dalamnya akan dikirim secara paksa** ke alamat yang ditentukan.

---

## Perilaku Unik `selfdestruct`

- Dalam kondisi normal, sebuah kontrak **tanpa** `receive()` atau `fallback()` yang `payable` **tidak dapat menerima ETH**.
- Namun, melalui `selfdestruct`, Anda **bisa memaksa mengirim ETH** ke kontrak mana pun, termasuk kontrak yang tidak memiliki fungsi penerima ETH.
- Ini karena `selfdestruct` adalah mekanisme tingkat sistem (opcode EVM) yang tidak mengikuti aturan penerimaan normal.

> **Inti:** `selfdestruct` adalah salah satu dari sedikit cara untuk "memaksa" ETH masuk ke kontrak yang seharusnya menolak ETH.

---

## Contoh Serangan dengan `selfdestruct`

Berikut adalah ilustrasi kontrak game yang rentan terhadap serangan menggunakan `selfdestruct`.

### Kontrak Game: `EtherGame`

```solidity
contract EtherGame {
    uint public targetAmount = 7 ether;
    address public winner;

    function deposit() public payable {
        require(msg.value == 1 ether, "You can only send 1 Ether");

        uint balance = address(this).balance;
        require(balance <= targetAmount, "Game is over");

        if (balance == targetAmount) {
            winner = msg.sender;
        }
    }

    function claimReward() public {
        require(msg.sender == winner, "Not winner");
        (bool sent, ) = msg.sender.call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }
}
```

**Aturan Game:**
- Setiap pemain hanya bisa deposit **1 ETH** sekali.
- Pemain ke-7 yang deposit akan menjadi pemenang (balance tepat 7 ETH).
- Pemenang bisa menarik semua ETH.

### Kontrak Serangan: `Attack`

```solidity
contract Attack {
    EtherGame etherGame;

    constructor(EtherGame _etherGame) {
        etherGame = EtherGame(_etherGame);
    }

    function attack() public payable {
        address payable addr = payable(address(etherGame));
        selfdestruct(addr);
    }
}
```

### Langkah Serangan

1. `EtherGame` di-deploy. Alice dan Bob masing-masing deposit 1 ETH (balance = 2 ETH).
2. Attacker deploy `Attack` dengan alamat `EtherGame`.
3. Attacker memanggil `attack()` **sambil mengirim 5 ETH**.
4. `selfdestruct(addr)` menghancurkan `Attack` dan memaksa 5 ETH masuk ke `EtherGame`.
5. Sekarang balance `EtherGame` menjadi **7 ETH langsung tanpa melalui fungsi `deposit()`**.
6. Tidak ada pemain yang mencapai balance tepat 7 ETH melalui deposit normal, sehingga `winner` tidak pernah ter-set.
7. Game rusak (stuck). Tidak ada yang bisa menang, dan tidak ada yang bisa deposit lagi karena `balance <= targetAmount` akan gagal.

### Mengapa Ini Terjadi?

- `EtherGame` mengasumsikan bahwa **satu-satunya cara balance bertambah** adalah melalui fungsi `deposit()` yang membatasi 1 ETH per transaksi.
- `selfdestruct` melanggar asumsi ini dengan **memaksa masuknya ETH dalam jumlah berapa pun** tanpa melalui `deposit()`.
- Akibatnya, invariant `balance == targetAmount` tidak pernah tercapai, atau tercapai secara tiba-tiba tanpa winner.

---

## Implikasi Keamanan

| Risiko | Penjelasan |
|--------|------------|
| **Forced ETH** | Kontrak yang tidak siap bisa menerima ETH paksa, merusak logika yang bergantung pada `address(this).balance`. |
| **Broken Invariant** | Banyak kontrak mengasumsikan bahwa balance hanya berubah melalui fungsi internal mereka. `selfdestruct` mematahkan asumsi ini. |
| **Game/Protocol Stuck** | Seperti contoh di atas, protokol bisa menjadi tidak berfungsi (stuck). |

### Pencegahan

- Jangan mengandalkan `address(this).balance` sebagai satu-satunya penentu logika kritis jika kontrak Anda bisa menerima ETH paksa.
- Gunakan **variabel internal** untuk melacak deposit, bukan hanya `balance`. Contoh: `uint256 totalDeposited` yang hanya bertambah melalui `deposit()`.
- Waspadai bahwa `selfdestruct` masih mungkin digunakan meskipun kontrak Anda tidak memiliki fungsi `receive`/`fallback` payable.

> **Catatan:** `selfdestruct` telah didepresi dalam Solidity versi terbaru (v0.8.18+ disarankan untuk tidak digunakan), tetapi opcode-nya masih ada di EVM dan akan tetap ada untuk kompatibilitas ke belakang. Di masa depan (setelah hard fork Shanghai), perilaku `selfdestruct` mungkin berubah.

---

## Kesimpulan

- `selfdestruct` adalah mekanisme untuk menghancurkan kontrak dan memaksa ETH ke alamat target.
- Ini adalah alat yang kuat bagi penyerang untuk **memaksa ETH masuk ke kontrak yang tidak menginginkannya**.
- Auditor dan developer harus memahami `selfdestruct` karena dapat merusak invariant yang mengandalkan `address(this).balance`.
- Contoh serangan pada `EtherGame` menunjukkan bagaimana game sederhana bisa dirusak hanya dengan satu panggilan `selfdestruct`.

> *"If you're hunting for an exploit where you need to force ETH into a contract, `selfdestruct` will be your instrument of choice."*
