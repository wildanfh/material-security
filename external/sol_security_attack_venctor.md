# Solidity Security: Daftar Lengkap Attack Vectors & Anti-Patterns

> Sumber: [blog.sigmaprime.io/solidity-security.html](https://blog.sigmaprime.io/solidity-security.html)  
> Referensi wajib bagi setiap smart contract security researcher.

---

## Daftar 16 Vulnerability

| # | Nama | Severity | Contoh Real |
|---|---|---|---|
| 1 | Re-Entrancy | Critical | The DAO ($60M) |
| 2 | Arithmetic Over/Under Flow | High | PoWHC, CVE-2018-10299 |
| 3 | Unexpected Ether | Medium | - |
| 4 | Delegatecall | Critical | Parity Multisig (2nd Hack) |
| 5 | Default Visibilities | Critical | Parity Multisig (1st Hack, $31M) |
| 6 | Entropy Illusion | High | PRNG Contracts |
| 7 | External Contract Referencing | Medium | Re-Entrancy Honey Pot |
| 8 | Short Address/Parameter Attack | Medium | - |
| 9 | Unchecked CALL Return Values | High | Etherpot, King of Ether |
| 10 | Race Conditions / Front Running | High | ERC20, Bancor |
| 11 | Denial of Service (DOS) | High | GovernMental |
| 12 | Block Timestamp Manipulation | Medium | GovernMental |
| 13 | Constructors with Care | High | Rubixi |
| 14 | Uninitialised Storage Pointers | High | OpenAddressLottery |
| 15 | Floating Points & Numerical Precision | Medium | Ethstick |
| 16 | tx.origin Authentication | High | - |

---

## 1. Re-Entrancy

### Cara Kerja
Ketika kontrak mengirim ETH ke alamat eksternal, kontrak attacker bisa langsung memanggil balik fungsi yang sama sebelum state diupdate.

### Contoh Vulnerable:
```solidity
function withdrawFunds(uint256 _amount) public {
    require(balances[msg.sender] >= _amount);
    // ❌ Kirim ETH dulu, update balance belakangan
    msg.sender.call.value(_amount)();
    balances[msg.sender] -= _amount; // terlambat!
}
```

### Fallback Attacker:
```solidity
function() payable {
    if (etherStore.balance > 1 ether) {
        etherStore.withdrawFunds(1 ether); // panggil lagi!
    }
}
```

### Fix — CEI Pattern + Mutex:
```solidity
bool reEntrancyMutex = false;

function withdrawFunds(uint256 _amount) public {
    require(!reEntrancyMutex);
    require(balances[msg.sender] >= _amount);
    // ✅ Update state DULU
    balances[msg.sender] -= _amount;
    lastWithdrawTime[msg.sender] = now;
    reEntrancyMutex = true;
    msg.sender.transfer(_amount); // transfer() hanya kirim 2300 gas
    reEntrancyMutex = false;
}
```

**Real World:** The DAO hack 2016 — $60M dicuri, menyebabkan Ethereum fork menjadi ETH dan ETC.

---

## 2. Arithmetic Over/Under Flow

### Cara Kerja
EVM menggunakan integer fixed-size. `uint8` hanya bisa simpan 0-255.
- `255 + 1 = 0` (overflow — wrap around)
- `0 - 1 = 255` (underflow — wrap around)

### Contoh Vulnerable:
```solidity
// ❌ Underflow: jika balances[sender] = 0 dan _value > 0
// require(0 - _value >= 0) → TRUE karena uint tidak negatif!
function transfer(address _to, uint _value) public {
    require(balances[msg.sender] - _value >= 0); // selalu true!
    balances[msg.sender] -= _value;
    balances[_to] += _value;
}
```

Attacker bisa transfer token gratis karena `0 - 1` pada uint menghasilkan `2^256 - 1`.

### Fix — SafeMath (Solidity < 0.8):
```solidity
// Solidity 0.8+ sudah built-in overflow check
// Untuk versi lama, gunakan OpenZeppelin SafeMath:
using SafeMath for uint256;

balances[msg.sender] = balances[msg.sender].sub(_value);
balances[_to] = balances[_to].add(_value);
```

**Real World:** PoWHC (Proof of Weak Hands Coin) — 866 ETH dicuri. CVE-2018-10299 di beberapa ERC20.

---

## 3. Unexpected Ether (Force-Sending ETH)

### Cara Kerja
Ada dua cara ETH bisa masuk ke kontrak tanpa memanggil fungsi apapun:
1. **`selfdestruct(targetAddress)`** — mengirim semua ETH ke target tanpa trigger apapun
2. **Pre-sent ETH** — alamat kontrak bisa diprediksi sebelum deploy, ETH bisa dikirim duluan

### Contoh Vulnerable:
```solidity
// ❌ Mengandalkan this.balance untuk logika kritis
function play() public payable {
    uint currentBalance = this.balance + msg.value;
    // Bisa dirusak dengan selfdestruct!
    require(currentBalance <= finalMileStone);
    if (currentBalance == payoutMileStone1) { ... }
}
```

Attacker kirim 0.1 ETH via `selfdestruct` → `this.balance` tidak pernah sama persis dengan milestone → game rusak permanen.

### Fix:
```solidity
// ✅ Tracking manual, jangan gunakan this.balance untuk logika
uint public depositedWei;

function play() public payable {
    uint currentBalance = depositedWei + msg.value;
    require(currentBalance <= finalMileStone);
    depositedWei += msg.value; // track manual
}
```

---

## 4. Delegatecall

### Cara Kerja
`delegatecall` menjalankan kode kontrak lain **dalam konteks storage kontrak pemanggil**. Storage slot diakses berdasarkan posisi (slot 0, slot 1, dst) — bukan nama variabel.

### Bahaya Storage Slot Mismatch:
```
Library Contract:          Calling Contract:
slot[0] = start            slot[0] = fibonacciLibrary  ← BERBEDA!
slot[1] = calculatedFib    slot[1] = calculatedFibNumber

Ketika library menulis ke slot[0] (start),
sebenarnya menimpa fibonacciLibrary address di calling contract!
```

Attacker bisa ganti `fibonacciLibrary` ke alamat kontrak jahat mereka sendiri, lalu drain semua ETH.

### Fix:
```solidity
// ✅ Gunakan keyword library (stateless, tidak bisa selfdestruct)
library SafeMathLib {
    function add(uint a, uint b) internal pure returns (uint) { ... }
}

// Hindari delegatecall ke kontrak stateful
// Selalu pastikan storage layout IDENTIK antara library dan caller
```

**Real World:** Parity Multisig Wallet hack ke-2 — $280M ETH terkunci permanen selamanya.

---

## 5. Default Visibilities

### Cara Kerja
Fungsi di Solidity default-nya `public` jika tidak dispesifikasi. Fungsi yang seharusnya private bisa dipanggil siapa saja.

### Contoh Vulnerable:
```solidity
contract HashForEther {
    function withdrawWinnings() {
        require(uint32(msg.sender) == 0);
        _sendWinnings(); // fungsi ini harusnya private!
    }

    // ❌ Tidak ada visibility specifier = public!
    function _sendWinnings() {
        msg.sender.transfer(this.balance); // siapa saja bisa panggil ini
    }
}
```

### Fix:
```solidity
// ✅ Selalu tentukan visibility secara eksplisit
function _sendWinnings() private {
    msg.sender.transfer(this.balance);
}
```

**Real World:** Parity Multisig Wallet hack ke-1 — $31M dicuri karena `initWallet()` tidak diberi visibility dan bisa dipanggil siapa saja untuk reset ownership.

---

## 6. Entropy Illusion (Pseudo-Randomness)

### Cara Kerja
Blockchain adalah sistem deterministik — tidak ada randomness sejati. Developer sering salah menggunakan:
- `block.timestamp`
- `block.number`
- `blockhash`

...sebagai sumber "random", padahal semua bisa diprediksi atau dimanipulasi miner.

### Contoh Vulnerable:
```solidity
// ❌ Miner bisa manipulasi block.timestamp dan blockhash
function random() private view returns (uint) {
    return uint(keccak256(block.timestamp, block.difficulty));
}
```

### Fix:
Gunakan **Chainlink VRF** (Verifiable Random Function) untuk randomness yang benar-benar tidak bisa dimanipulasi.

```solidity
// ✅ Gunakan Chainlink VRF
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";
```

---

## 7. External Contract Referencing

### Cara Kerja
Ketika kontrak menyimpan alamat kontrak eksternal, ada risiko alamat tersebut menunjuk ke kontrak jahat — terutama jika alamat bisa diubah atau tidak diverifikasi.

### Contoh Honey Pot:
```solidity
contract Rot13Encryption {
    // Kelihatannya legitimate tapi bisa diganti dengan kontrak jahat
    function rot13Encrypt(string text) public pure returns (string) { ... }
}

// Attacker deploy kontrak jahat di alamat yang sama
// Lalu setup honey pot untuk mencuri ETH dari yang mencoba exploit
```

### Fix:
```solidity
// ✅ Hardcode alamat sebagai constant jika memungkinkan
address constant TRUSTED_CONTRACT = 0x1234...;

// ✅ Atau gunakan interface yang terverifikasi
// ✅ Selalu verifikasi kontrak yang di-reference
```

---

## 8. Short Address/Parameter Attack

### Cara Kerja
Serangan ini menarget ABI encoding. Jika user mengirim alamat yang lebih pendek dari 20 bytes, EVM akan padding dengan 0 di akhir — menyebabkan parameter berikutnya (biasanya amount) terbaca lebih besar dari yang dimaksud.

### Fix:
```solidity
// ✅ Validasi panjang input
modifier validAddress(address _addr) {
    require(_addr != address(0));
    _;
}

// Validasi di sisi backend/frontend sebelum send transaksi
```

---

## 9. Unchecked CALL Return Values

### Cara Kerja
`address.call()` tidak throw exception saat gagal — hanya return `false`. Jika return value tidak dicek, eksekusi berlanjut seolah transfer berhasil.

### Contoh Vulnerable:
```solidity
// ❌ Return value tidak dicek
function withdraw(uint amount) public {
    msg.sender.call.value(amount)(); // bisa gagal tanpa revert!
    balances[msg.sender] -= amount;  // balance tetap dikurangi
}
```

### Fix:
```solidity
// ✅ Selalu cek return value, atau gunakan transfer()/send()
(bool success, ) = msg.sender.call{value: amount}("");
require(success, "Transfer failed");

// Atau lebih aman:
msg.sender.transfer(amount); // auto-revert jika gagal
```

**Real World:** Etherpot dan King of the Ether — dana hilang karena call failure tidak ditangani.

---

## 10. Race Conditions / Front Running

### Cara Kerja
Transaksi Ethereum bersifat publik sebelum di-mine. Attacker (atau miner) bisa melihat transaksi pending dan submit transaksi mereka sendiri dengan gas price lebih tinggi untuk dieksekusi duluan.

### Skenario:
```
1. Alice submit tx: approve(Bob, 100 tokens) → gas 10 gwei
2. Alice kemudian submit tx: approve(Bob, 50 tokens) → gas 10 gwei
3. Bob melihat tx pertama di mempool
4. Bob front-run tx kedua Alice dengan gas 50 gwei
5. Bob transferFrom 100 tokens (dari approval pertama)
6. Tx kedua Alice dieksekusi → Bob dapat 50 token lagi
7. Total Bob dapat 150, Alice cuma mau kasih 50!
```

### Fix:
```solidity
// ✅ Gunakan increaseAllowance/decreaseAllowance bukan set langsung
function increaseAllowance(address spender, uint256 addedValue) public {
    _approve(msg.sender, spender, allowance(msg.sender, spender) + addedValue);
}

// ✅ Untuk DEX: gunakan commit-reveal scheme
// ✅ Gunakan minimum/maximum threshold parameter
```

---

## 11. Denial of Service (DOS)

### Cara Kerja
Beberapa pola bisa menyebabkan kontrak tidak bisa digunakan:

**Tipe 1 — Loop tak terbatas:**
```solidity
// ❌ Loop yang terlalu panjang bisa habiskan gas limit
function refundAll() public {
    for (uint i; i < refunds.length; i++) {
        refunds[i].transfer(amounts[i]); // gagal = seluruh tx revert
    }
}
```

**Tipe 2 — External call yang bisa diblok:**
```solidity
// ❌ Jika satu recipient adalah kontrak yang reject ETH,
// seluruh loop akan revert selamanya
```

### Fix:
```solidity
// ✅ Pull payment pattern — user yang claim sendiri
mapping(address => uint) public pendingWithdrawals;

function withdraw() public {
    uint amount = pendingWithdrawals[msg.sender];
    pendingWithdrawals[msg.sender] = 0;
    payable(msg.sender).transfer(amount);
}
```

**Real World:** GovernMental — jutaan dolar ETH terkunci karena gas limit.

---

## 12. Block Timestamp Manipulation

### Cara Kerja
Miner bisa memanipulasi `block.timestamp` hingga ~900 detik (15 menit) dari waktu sebenarnya. Kontrak yang bergantung pada timestamp untuk keputusan kritis bisa dieksploitasi.

### Contoh Vulnerable:
```solidity
// ❌ Miner bisa adjust timestamp untuk menang
function isSummerTime() public view returns (bool) {
    return (block.timestamp % (365 days) >= 90 days);
}

function bet() public payable {
    if (block.timestamp % 7 == 0) { // bisa dimanipulasi!
        msg.sender.transfer(prize);
    }
}
```

### Fix:
```solidity
// ✅ Gunakan block.number untuk time-sensitive logic
// ✅ Jika butuh timestamp, pastikan toleransi > 15 menit
// ✅ Jangan gunakan timestamp untuk randomness
```

---

## 13. Constructors with Care

### Cara Kerja
Di Solidity versi lama (< 0.4.22), constructor harus bernama sama dengan nama kontrak. Typo pada nama constructor menyebabkan fungsi tersebut menjadi fungsi biasa yang bisa dipanggil siapa saja.

### Contoh Vulnerable (Solidity Lama):
```solidity
contract Rubixi {
    address private owner;

    // ❌ Nama berbeda dari nama kontrak! Ini fungsi publik biasa
    function DynamicPyramid() public {
        owner = msg.sender; // siapa saja bisa jadi owner!
    }
}
```

### Fix:
```solidity
// ✅ Solidity 0.4.22+ gunakan keyword constructor
contract MyContract {
    address private owner;

    constructor() public {
        owner = msg.sender;
    }
}
```

**Real World:** Rubixi — kontrak rename dari "DynamicPyramid" tapi lupa ubah nama constructor, siapa saja bisa claim ownership.

---

## 14. Uninitialised Storage Pointers

### Cara Kerja
Variabel lokal bertipe `struct` atau `array` yang tidak di-inisialisasi akan default menunjuk ke `storage slot 0`. Ini bisa menyebabkan penulisan tak sengaja ke state variable penting.

### Contoh Vulnerable:
```solidity
// ❌ nameRecord tidak di-inisialisasi = menunjuk ke slot 0!
function register(uint256 _id, address _owner) public {
    NameRecord nameRecord; // storage pointer ke slot 0!
    nameRecord.name = _id;   // menimpa variable pertama di storage
    nameRecord.mappedAddress = _owner;
}
```

### Fix:
```solidity
// ✅ Selalu inisialisasi storage variable
// ✅ Gunakan memory untuk variable temporary
function register(uint256 _id, address _owner) public {
    NameRecord memory nameRecord; // 'memory' bukan storage
    nameRecord.name = _id;
    // ...
}
```

---

## 15. Floating Points & Numerical Precision

### Cara Kerja
Solidity tidak mendukung floating point. Semua kalkulasi menggunakan integer. Pembagian integer selalu dibulatkan ke bawah (floor), yang bisa menyebabkan loss of precision atau logical errors.

### Contoh Vulnerable:
```solidity
// ❌ Urutan operasi salah — hasil precision loss
uint numerator = 10;
uint denominator = 100;
uint result = numerator / denominator * target; // = 0 * target = 0!
```

### Fix:
```solidity
// ✅ Kalikan DULU sebelum bagi (multiply before divide)
uint result = numerator * target / denominator; // = 10 * target / 100

// ✅ Gunakan faktor pengali besar (1e18) untuk simulasi decimal
uint PRECISION = 1e18;
uint price = (amount * PRECISION) / totalSupply;
```

---

## 16. tx.origin Authentication

### Cara Kerja
`tx.origin` adalah EOA yang memulai seluruh call chain. `msg.sender` adalah kontrak/address yang langsung memanggil fungsi ini. Jika kontrak menggunakan `tx.origin` untuk autentikasi, bisa diexploit via phishing.

### Contoh Vulnerable:
```solidity
// ❌ tx.origin bisa dimanipulasi via kontrak perantara
function transferTo(address _to, uint _amount) public {
    require(tx.origin == owner); // SALAH!
    _to.transfer(_amount);
}
```

**Serangan:**
```
1. Owner tertipu klik link → memanggil kontrak attacker
2. Kontrak attacker panggil transferTo(attackerAddr, balance)
3. tx.origin = owner (masih owner!) → require LULUS
4. ETH ditransfer ke attacker
```

### Fix:
```solidity
// ✅ Selalu gunakan msg.sender untuk autentikasi
require(msg.sender == owner); // BENAR

// tx.origin hanya boleh digunakan untuk:
// - Memastikan EOA yang memanggil (bukan kontrak)
// - Anti-contract-call protection
```

---

## Rangkuman: Checklist Sebelum Deploy

```
Security Checklist:
□ Semua fungsi punya visibility specifier (public/private/internal/external)
□ Ikuti CEI pattern (Checks → Effects → Interactions)
□ Gunakan nonReentrant modifier untuk fungsi yang kirim ETH
□ Jangan gunakan this.balance untuk logika kritis
□ Gunakan SafeMath atau Solidity 0.8+ untuk aritmatika
□ Cek return value dari semua external call
□ Gunakan msg.sender bukan tx.origin untuk autentikasi
□ Jangan gunakan block.timestamp/blockhash untuk randomness
□ Gunakan keyword constructor (bukan nama fungsi = nama kontrak)
□ Inisialisasi semua storage pointer dengan benar
□ Kalikan sebelum membagi untuk presisi numerik
□ Gunakan pull payment pattern untuk menghindari DOS
□ Verifikasi semua alamat kontrak eksternal yang di-reference
□ Hati-hati dengan storage slot saat menggunakan delegatecall
```

---

## Resource Tambahan

- [Ethereum Smart Contract Best Practices — Consensys](https://consensys.github.io/smart-contract-best-practices)
- [DASP Top 10 of 2018](http://www.dasp.co/)
- [Solidity Docs — Security Considerations](https://docs.soliditylang.org/en/latest/security-considerations.html)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/) — praktik langsung semua vulnerability ini
- [Ethernaut](https://ethernaut.openzeppelin.com/) — gamified challenges
