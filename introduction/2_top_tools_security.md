# Top Tools untuk Smart Contract Security

## Prinsip Utama

> **Keamanan bukan sesuatu yang ditempelkan di akhir siklus development — ini adalah pertimbangan fundamental sejak awal.**

---

## Proses Audit Secara Umum

```
1. Manual Review (paling penting)
   → Baca kode & dokumentasi
   → Pahami apa yang seharusnya dilakukan protokol

2. Gunakan Tools
   → Static analysis, fuzz testing, formal verification, dll
```

### Kenapa Manual Review Paling Penting?

**80% bug adalah business logic bug** — bukan error kode, tapi implementasi yang tidak sesuai dengan apa yang seharusnya terjadi.

Contoh:

```solidity
// Dokumentasi bilang: "setNumber harus assign newNumber ke number"
// Tapi kodenya melakukan ini:
function setNumber(uint256 newNumber) public {
    number = newNumber + 1; // ← BUG! harusnya = newNumber
}
```

Kode di atas tidak ada error syntax, tidak akan gagal kompilasi. Hanya dengan membaca dokumentasi kamu bisa tahu ini salah.

---

## 6 Kategori Tools Security

### 1. Test Suites
**Lini pertahanan pertama.** Semua framework populer (Foundry, Hardhat) punya test suite. Gunakan secara rutin selama development.

---

### 2. Static Analysis
Memeriksa kode untuk masalah **tanpa mengeksekusinya**.

| Tool | Link |
|---|---|
| **Aderyn** | [github.com/Cyfrin/aderyn](https://github.com/Cyfrin/aderyn) |
| **Slither** | [github.com/crytic/slither](https://github.com/crytic/slither) |
| **Mythril** | [github.com/Consensys/mythril](https://github.com/Consensys/mythril) |

**Contoh penggunaan Slither:**
```bash
slither .
```
Slither langsung output semua issue yang terdeteksi ke terminal. Tidak menangkap segalanya, tapi wajib dijalankan sebelum audit.

**Bug yang bisa ditangkap Slither — Reentrancy:**
```solidity
function withdraw() external {
    uint256 balance = balances[msg.sender];
    require(balance > 0);
    // ❌ Transfer ETH SEBELUM update balance = celah reentrancy!
    (bool success, ) = msg.sender.call{value: balance}("");
    require(success, "Failed to send Ether");
    balances[msg.sender] = 0; // ← harusnya ini duluan
}
```
Reentrancy adalah salah satu vulnerability paling berbahaya di Web3 — penyerang bisa memanggil `withdraw()` berulang kali sebelum balance di-reset ke 0.

---

### 3. Fuzz Testing
Memberikan **input random** secara otomatis untuk menemukan edge case yang tidak terpikirkan.

#### Stateless Fuzz Testing
Setiap run dimulai dari state baru — tidak ingat state dari run sebelumnya.

```solidity
// Invariant: "doMoreMath tidak boleh pernah return 0"
function testFuzz(uint256 randomNumber) public {
    uint256 returnedNumber = caughtWithFuzz.doMoreMath(randomNumber);
    assert(returnedNumber != 0);
}
```
Fuzz test akan mencoba ribuan angka random dan menemukan bahwa input `1265` membuat fungsi return 0 — sesuatu yang hampir mustahil ditemukan manual.

#### Stateful Fuzz Testing
Mengingat state akhir dari satu run dan menggunakannya sebagai state awal run berikutnya. Diperlukan ketika bug hanya muncul dari **urutan pemanggilan fungsi tertentu**.

```solidity
// Bug hanya muncul jika:
// 1. changeValue(0) dipanggil dulu
// 2. Lalu doMoreMathAgain(0) dipanggil
// → Stateless fuzz tidak akan menangkap ini!

contract CaughtWithStatefulFuzzTest is StdInvariant, Test {
    function setUp() public {
        caughtWithStatefulFuzz = new CaughtWithStatefulFuzz();
        targetContract(address(caughtWithStatefulFuzz)); // ← kunci stateful fuzz
    }

    function invariant_testMathDoesntReturnZero() public {
        assert(caughtWithStatefulFuzz.storedValue() != 0);
    }
}
```

**Perbedaan Stateless vs Stateful:**

| | Stateless | Stateful |
|---|---|---|
| State antar run | Reset setiap run | Dilanjutkan dari run sebelumnya |
| Cocok untuk | Bug dari input tunggal | Bug dari urutan pemanggilan |
| Foundry keyword | `testFuzz...` | `invariant_...` + `targetContract()` |

---

### 4. Differential Testing
Menulis kode dengan **beberapa cara berbeda** lalu membandingkan hasilnya untuk memastikan konsistensi. Tidak dibahas mendalam di materi ini.

---

### 5. Formal Verification
Menggunakan **bukti matematis** untuk memverifikasi kebenaran sistem — membuktikan secara formal apakah bug *pasti ada* atau *pasti tidak ada*.

Salah satu pendekatannya adalah **Symbolic Execution** — alih-alih menguji nilai spesifik, sistem menganalisis semua kemungkinan jalur eksekusi secara matematis.

**Contoh dengan Solidity Compiler (SMTChecker):**
```solidity
function functionOneSymbolic(int128 x) public pure {
    if (x / 4 == -2022) {
        assert(false); // ← beritahu compiler: kondisi ini TIDAK BOLEH terjadi
        revert();
    }
    assert(true);
}
```

Konfigurasi di `foundry.toml`:
```toml
[profile.default.model_checker]
contracts = {'./src/CaughtWithSymbolic.sol' = ['CaughtWithSymbolic']}
engine = 'chc'
timeout = 1000
targets = ['assert']
```

Jalankan `forge build` → compiler langsung memberikan counter-example yang melanggar assertion.

**Tools Formal Verification lainnya:**
| Tool | Link |
|---|---|
| **Manticore** | [github.com/trailofbits/manticore](https://github.com/trailofbits/manticore) |
| **Halmos** | [github.com/a16z/halmos](https://github.com/a16z/halmos) |
| **Certora** | [certora.com/prover](https://www.certora.com/prover) |

---

### 6. AI Tools
Masih berkembang, bisa hit or miss. Berguna untuk:
- Memahami konteks codebase dengan cepat
- Merangkum / mengklarifikasi dokumentasi
- Brainstorming attack vector

> ⚠️ Jangan bergantung penuh pada AI tools untuk audit — gunakan sebagai pelengkap, bukan pengganti.

---

## Rangkuman: Tool yang Tepat untuk Bug yang Tepat

| Tipe Bug | Tool Terbaik |
|---|---|
| Kesalahan logic yang obvious | Manual Review |
| Off-by-one, assignment salah | Unit Test |
| Reentrancy, access control | Static Analysis (Slither/Aderyn) |
| Edge case dari input random | Stateless Fuzz Test |
| Bug dari urutan pemanggilan fungsi | Stateful Fuzz Test |
| Bukti matematis kebenaran sistem | Formal Verification |

---

## Resource Tambahan

- [solcurity](https://github.com/transmissions11/solcurity) — checklist keamanan Solidity
- [simple-security-toolkit](https://github.com/nascentxyz/simple-security-toolkit) — toolkit + audit readiness checklist
- [Reentrancy explained](https://solidity-by-example.org/hacks/re-entrancy/) — wajib dipahami
- [EVM Symbolic Execution comparison](https://hackmd.io/@SaferMaker/EVM-Sym-Exec) — perbandingan tools formal verification
