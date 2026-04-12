# Layers of Testing & Formal Verification

> Materi dari Josselin & Troy (Trail of Bits) — salah satu firma security smart contract paling terkemuka di dunia.

---

## Overview: 4 Layer Testing

```
Layer 1: Unit Tests          → Minimum absolute
Layer 2: Fuzz Tests          → Standar minimum yang direkomendasikan
Layer 3: Static Analysis     → Deteksi pattern vulnerability tanpa eksekusi
Layer 4: Formal Verification → Bukti matematis kebenaran sistem
```

> ⚠️ **Bahkan semua layer ini bukan jaminan kode bebas bug.**

---

## Layer 1: Unit Tests

**Bare minimum** dalam testing Web3 security.

Unit test mengajukan situasi spesifik ke fungsi dan memvalidasi bahwa situasi tersebut berjalan sesuai harapan.

### Contoh Bug:
```solidity
contract CaughtWithTest {
    uint256 public number;

    function setNumber(uint256 newNumber) public {
        number = newNumber + 1; // ← BUG: harusnya = newNumber
    }
}
```

### Unit Test yang Menangkap Bug:
```solidity
function testSetNumber() public {
    uint256 myNumber = 55;
    caughtWithTest.setNumber(myNumber);
    assertEq(myNumber, caughtWithTest.number()); // ← FAIL: 55 != 56
}
```

**Karakteristik:**
- Menguji satu situasi spesifik per test
- Cepat dan mudah ditulis
- Tidak bisa menemukan edge case yang tidak terpikirkan developer
- Semua framework populer (Foundry, Hardhat) sudah include ini

---

## Layer 2: Fuzz Tests

Memberikan **input random secara otomatis** untuk menemukan edge case yang melanggar invariant protokol.

### Apa itu Invariant?
> Invariant = properti protokol yang **harus selalu benar** dalam kondisi apapun.

Contoh invariant: *"shouldAlwaysBeZero tidak pernah boleh bernilai selain 0"*

### Contoh Bug yang Butuh Fuzz:
```solidity
contract MyContract {
    uint256 public shouldAlwaysBeZero = 0;

    function doStuff(uint256 data) public {
        if (data == 2) {
            shouldAlwaysBeZero = 1; // ← invariant rusak jika data = 2
        }
    }
}
```

Pada kontrak sederhana ini kita bisa lihat bugnya langsung. Tapi di kontrak kompleks dengan ratusan kondisi, mustahil ditemukan manual.

### Fuzz Test yang Menangkap Bug:
```solidity
function testIAlwaysGetZeroFuzz(uint256 data) public {
    myContract.doStuff(data);
    assert(myContract.shouldAlwaysBeZero() == 0);
}
```

`data` tidak di-hardcode — Foundry akan mengisi dengan nilai random sampai invariant rusak atau mencapai batas run yang dikonfigurasi.

**Karakteristik:**
- Otomatis menemukan counter-example tanpa developer harus memikirkan semua edge case
- Menemukan nilai `2` secara otomatis dari sekian miliar kemungkinan
- **Ini adalah standar minimum yang direkomendasikan untuk Web3 2024+**

### Stateless vs Stateful Fuzz (Recap)

| | Stateless | Stateful |
|---|---|---|
| State antar run | Reset setiap run | Dilanjutkan |
| Cocok untuk | Bug dari input tunggal | Bug dari urutan pemanggilan |
| Keyword Foundry | `testFuzz...` | `invariant_...` |

---

## Layer 3: Static Analysis

**Perbedaan fundamental:**

| Dynamic Testing | Static Analysis |
|---|---|
| Kode dieksekusi | Kode TIDAK dieksekusi |
| Unit test, Fuzz test | Slither, Aderyn |
| Temukan bug saat runtime | Temukan bug dari layout/syntax/pattern |

### Tools:
- **[Slither](https://github.com/crytic/slither)** — paling populer, dikembangkan Trail of Bits
- **[Aderyn](https://github.com/Cyfrin/aderyn)** — dikembangkan Cyfrin

### Contoh Bug yang Ditangkap Slither — Reentrancy:
```solidity
function withdraw() external {
    uint256 balance = balances[msg.sender];
    require(balance > 0);

    // ❌ Transfer ETH SEBELUM update balance = celah reentrancy!
    (bool success, ) = msg.sender.call{value: balance}("");
    require(success, "Failed to send Ether");

    balances[msg.sender] = 0; // ← harusnya SEBELUM transfer (CEI pattern)
}
```

**Fix — ikuti CEI Pattern (Checks-Effects-Interactions):**
```solidity
function withdraw() external {
    uint256 balance = balances[msg.sender];
    require(balance > 0);              // Check
    balances[msg.sender] = 0;          // Effect  ← pindah ke sini
    (bool success, ) = msg.sender.call{value: balance}(""); // Interaction
    require(success, "Failed to send Ether");
}
```

Slither mendeteksi pola ini karena melihat urutan operasi dalam kode — tanpa perlu menjalankannya.

```bash
slither .  # jalankan dari root project
```

---

## Layer 4: Formal Verification

### Definisi
> Formal Verification = membuktikan atau menyangkal secara **matematis** bahwa suatu properti sistem selalu terpenuhi.

Cara kerjanya: membuat **model matematis** dari sistem, lalu menggunakan solver untuk membuktikan apakah ada jalur eksekusi yang melanggar properti tersebut.

### Tiga Metode Formal Verification:
1. **Symbolic Execution** ← yang dibahas di materi ini
2. Abstract Interpretation
3. Model Checking

---

### Symbolic Execution: Cara Kerja

Alih-alih menguji nilai spesifik, Symbolic Execution **memodelkan semua kemungkinan jalur eksekusi secara matematis**.

#### Contoh:
```solidity
contract SmallSol {
    // Invariant: fungsi ini tidak boleh pernah revert
    function f(uint256 a) public returns (uint256) {
        a = a + 1;
        return a;
    }
}
```

Symbolic Execution mengidentifikasi **semua jalur yang mungkin**:

```
Path 1: assert(a != 2^256 - 1) → a := a + 1 → return a
        (semua nilai a kecuali uint256.max — AMAN)

Path 2: assert(a == 2^256 - 1) → overflow → revert
        (a = uint256.max — MELANGGAR INVARIANT!)
```

Solver akan mencoba "satisfy" Path 2. Jika bisa, artinya invariant bisa dilanggar — **bug ditemukan**.

> 💡 Tools formal verification menggunakan bahasa khusus bernama **SMT-LIB** untuk memproses model matematis ini.

### Tools Formal Verification:

| Tool | Keterangan |
|---|---|
| **Manticore** | Trail of Bits — symbolic execution |
| **Halmos** | a16z — symbolic testing untuk Foundry |
| **Certora** | Prover berbasis specification language |
| **Solidity SMTChecker** | Built-in di Solidity compiler |

### Cara Aktifkan SMTChecker di Foundry:
```toml
# foundry.toml
[profile.default.model_checker]
contracts = {'./src/MyContract.sol' = ['MyContract']}
engine = 'chc'
timeout = 1000
targets = ['assert']
```

Jalankan `forge build` → compiler langsung report jika ada assertion yang bisa dilanggar.

---

### Keterbatasan Formal Verification

**Path Explosion Problem:**

Ketika kode mengandung:
- Loop yang tidak deterministik
- Loop yang berpotensi infinite
- Terlalu banyak kondisi bercabang

→ Jumlah jalur yang harus diperiksa mendekati **tak terhingga** → solver tidak bisa menyelesaikan proof dalam waktu wajar.

Ini adalah alasan mengapa formal verification **tidak bisa menggantikan** layer testing lainnya, hanya melengkapi.

---

## Perbandingan Semua Layer

| Layer | Eksekusi Kode | Kecepatan | Yang Ditangkap |
|---|---|---|---|
| Unit Test | ✅ Ya | ⚡ Sangat cepat | Bug dari situasi spesifik yang diketahui |
| Fuzz Test | ✅ Ya | 🔄 Butuh banyak run | Edge case dari input random |
| Static Analysis | ❌ Tidak | ⚡ Cepat | Pattern vulnerability yang diketahui |
| Formal Verification | ❌ Tidak | 🐢 Lambat/kompleks | Bukti matematis untuk semua kemungkinan |

---

## Rekomendasi Praktis

```
Minimum untuk semua protokol:
✅ Unit Tests
✅ Fuzz Tests (stateless + stateful)
✅ Slither / Aderyn sebelum audit

Untuk protokol dengan TVL tinggi:
✅ Semua di atas +
✅ Formal Verification (Halmos/Certora)
✅ Private Audit + Competitive Audit
```

> **"A thorough fuzz testing suite should be the bare minimum standard in Web3 protocol security."** — Trail of Bits

---

## Resource Tambahan
- [secure-contracts.com](https://secure-contracts.com) — panduan lengkap dari Trail of Bits
- [Solidity SMTChecker Docs](https://docs.soliditylang.org/en/v0.8.26/smtchecker.html)
- [MIT Symbolic Execution Video](https://www.youtube.com/watch?v=yRVZPvHYHzw)
