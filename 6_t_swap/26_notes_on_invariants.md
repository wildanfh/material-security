# Invariants & Additional Fuzzing Tools

## 1. Pentingnya Invariants

### Apa yang sudah dipelajari
Fuzzing terbukti sangat powerful untuk **membreak protocol invariants** — kondisi yang *harus selalu benar* dalam sebuah protokol, tidak peduli state apapun yang terjadi.

### FREI-PI
**FREI-PI** (Function Requirements — Effects — Interactions + Protocol Invariants) adalah konsep *baking invariants langsung ke dalam desain protokol* sejak awal.

> Kalau invariants tidak dipertimbangkan sejak desain, protokol bisa sangat rentan — seperti yang terjadi pada **Euler Finance**.

### Case Study: Euler Finance (by Tincho)
- Video oleh **Tincho (Martin Abbatemarco)** membahas exploit nyata pada Euler Finance
- Menunjukkan secara langsung apa yang terjadi ketika invariants tidak dijaga dengan benar
- **Pelajaran utama:** Invariants yang tidak di-enforce on-chain bisa dieksploitasi secara sistematis oleh attacker

---

## 2. Additional Fuzzing Tools

Selain **Foundry** (yang dipakai eksklusif selama kursus), ada beberapa tool lain yang perlu diketahui:

### 🦔 Echidna
| Atribut | Detail |
|---|---|
| **Bahasa** | Haskell |
| **Repo** | [github.com/crytic/echidna](https://github.com/crytic/echidna) |
| **Pendekatan** | Grammar-based fuzzing campaign |
| **Cara kerja** | Berdasarkan contract ABI untuk falsifikasi user-defined predicates atau Solidity assertions |
| **Dibuat oleh** | Trail of Bits (Crytic) |

Echidna cocok untuk **property-based testing** — kamu mendefinisikan properti/invariant, Echidna akan mencoba semua kombinasi input untuk membuktikan properti itu bisa dilanggar.

---

### ☁️ Consensys Diligence Fuzzing (FaaS)
- **Fuzzing as a Service** — infrastruktur fuzzing yang dikelola Consensys Diligence
- **Berbayar** — tidak dicover dalam kursus ini
- Cocok untuk tim/protokol yang ingin fuzzing skala besar tanpa setup sendiri

---

### 🧬 Mutation Testing
- **Konsep:** Mengubah bagian kecil dari source code (mutasi), lalu cek apakah test suite yang ada bisa mendeteksi perubahan tersebut
- Kalau test **tidak** gagal setelah mutasi → berarti test suite kurang kuat / ada blind spot
- Referensi: [notes.md di repo t-swap audit](https://github.com/Cyfrin/5-t-swap-audit/blob/audit-data/test/mutation/notes.md)
- **Tidak dicover** dalam kursus ini, tapi worth dipelajari mandiri

---

### 🔁 Differential Testing
- **Konsep:** Membandingkan dua versi berbeda dari implementasi yang sama (misal: versi lama vs versi baru dari sebuah protokol)
- Jika output berbeda untuk input yang sama → ada bug di salah satu versi
- Sangat berguna untuk audit upgrade/migration contract
- Akan dibahas lebih detail di course selanjutnya

---

## 3. Resource Tambahan

### Solodit — Research Weird ERC20s
- **URL:** [solodit.xyz](https://solodit.xyz/)
- Database audit report dari berbagai protokol
- Bisa dipakai untuk research **Weird ERC20s** — token dengan behaviour non-standard yang sering jadi attack vector:
  - Fee-on-transfer tokens
  - Rebasing tokens
  - Tokens dengan `transfer()` yang tidak return bool
  - Tokens yang bisa blacklist address
  - dll.
- Cek bagaimana auditor lain mengidentifikasi Weird ERC20 issues di audit-audit sebelumnya

---

## 4. Langkah Selanjutnya

Setelah memahami fuzzing dan tooling:

```
Fuzzing (automated) ──► Manual Review
```

**Manual review** adalah tahap berikutnya — membaca kode secara langsung, berpikir secara adversarial, dan menemukan bug yang tidak bisa dideteksi oleh fuzzer manapun.

---

## Ringkasan Cepat

| Tool | Tipe | Status dalam Kursus |
|---|---|---|
| Foundry | Fuzzing framework | ✅ Sudah dicover |
| Echidna | Property-based fuzzing | 📖 Perlu eksplorasi mandiri |
| Consensys FaaS | Fuzzing as a Service | 💰 Berbayar, tidak dicover |
| Mutation Testing | Test quality check | 📖 Referensi tersedia |
| Differential Testing | Comparative testing | 🔜 Dicover di kursus lanjutan |
| Solodit | Audit research DB | 🔍 Resource mandiri |