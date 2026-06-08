```markdown
# Rekap Section 5 – Audit TSwap: Fuzzing, Invariant, dan Manual Review

## Pendahuluan

Section ini merupakan perjalanan panjang dan padat, mencakup berbagai konsep lanjutan dalam security review smart contract. Berikut adalah rekap dari semua materi yang telah dipelajari.

---

## 1. Konteks & Pemahaman Protokol

Tanpa melakukan manual review secara mendalam, kita mampu membangun **fuzzing test suites** (stateless dan stateful) untuk mengidentifikasi pelanggaran invariant. Pengalaman ini menunjukkan betapa efisiennya penggunaan alat (tooling) dan pemahaman protokol yang baik dalam membantu proses audit.

---

## 2. Apa Itu AMM dan DEX

Sebagai bagian dari pengumpulan konteks TSwap, kita mempelajari:

- **Automated Market Maker (AMM)** – mekanisme penentuan harga otomatis menggunakan liquidity pool.
- **Decentralized Exchange (DEX)** – bursa terdesentralisasi yang memungkinkan swap token tanpa perantara.
- Perbedaan AMM dengan **order book** konvensional, dan mengapa AMM lebih cocok untuk lingkungan blockchain (biaya gas lebih rendah, selalu tersedia likuiditas).

![recap1](/security-section-5/48-recap/recap1.png)

![recap2](/security-section-5/48-recap/recap2.png)

---

## 3. Liquidity Providers (Penyedia Likuiditas)

- **Liquidity Providers** menyetor dana ke liquidity pool sehingga AMM dapat memfasilitasi perdagangan.
- Biaya (fee) yang dikenakan saat transaksi digunakan sebagai **insentif** bagi liquidity providers, dibagikan secara proporsional berdasarkan kontribusi mereka ke pool.

---

## 4. Core Invariants & Constant Product Formula

- **Core invariant** adalah properti fundamental protokol yang harus selalu benar.
- Contoh: rumus **constant product** `x * y = k` yang digunakan di TSwap dan Uniswap – rasio antara dua token dalam pool harus tetap sama.
- Sumber daya penting: [Repositori Properties oleh Trail of Bits](https://github.com/crytic/properties) – berisi banyak contoh invariant untuk token umum dan lainnya.

---

## 5. Onboarding Klien yang Mendetail

Kita menekankan pentingnya **extensive onboarding** untuk menggali informasi sebanyak mungkin dari klien, seperti:

- **Test coverage** – seberapa banyak kode diuji.
- **Token/chain compatibilities** – token yang didukung, rantai target, versi Solidity.
- **nSLOC** – jumlah baris kode untuk estimasi waktu audit.
- **Scope** – kontrak mana yang masuk audit, commit hash spesifik.
- **Roles** – peran aktor dalam protokol.

Dengan onboarding yang baik, auditor mendapatkan konteks yang cukup sebelum menyentuh kode.

---

## 6. Fuzzing (Stateless & Stateful)

Kita mempelajari cara menulis:

- **Stateless fuzzing** – setiap run dimulai dengan state baru, cocok untuk menguji fungsi individual dengan input acak.
- **Stateful fuzzing** – state dipertahankan antar run, diperlukan untuk menemukan kerentanan yang bergantung pada urutan panggilan fungsi.
- **Handler‑based fuzzing** – menggunakan handler untuk membatasi input dan urutan panggilan, meningkatkan efisiensi dan mengurangi path explosion.

Fuzzing terbukti sangat kuat dalam menangkap bug yang tidak terdeteksi oleh unit test biasa atau static analysis.

---

## 7. FREI‑PI / CEI (Checks‑Effects‑Interactions)

Kita menyentuh metodologi **FREI‑PI** (dari Nascent) dan gagasan **hardcoding invariant checks** langsung ke dalam kode smart contract. Prinsip CEI (Checks‑Effects‑Interactions) adalah praktik standar untuk mencegah reentrancy.

---

## 8. Tooling dalam Audit TSwap

Berbagai alat yang digunakan:

| Alat | Fungsi |
|------|--------|
| **Foundry** (fuzzing) | Stateful & stateless fuzzing, invariant test |
| **Slither** | Static analysis (Python) |
| **Aderyn** | Static analysis (Rust) |
| **Solodit** | Platform agregasi laporan audit untuk belajar dari temuan orang lain |
| **Compiler Solidity** (`forge build`) | Memberi peringatan tentang variabel tidak terpakai, parameter tidak digunakan, dll. |
| **Echidna** (disinggung) | Alternatif fuzzing yang terintegrasi dengan Slither |
| **Certora** (disinggung) | Verifikasi formal (akan dipelajari lebih lanjut) |

---

## 9. Weird ERC20s

- **Weird ERC20** adalah token yang berperilaku tidak standar, misal: fee‑on‑transfer, rebasing, reentrancy, upgradeable, blocklist.
- Kerentanan ini sangat umum di DeFi. Contoh: token `YieldERC20` dalam latihan kita memotong 10% fee setiap 10 transfer.
- Sumber daya penting:
  - [Weird ERC20 GitHub repo](https://github.com/d-xo/weird-erc20)
  - [Token Integration Checklist oleh Trail of Bits](https://secure-contracts.com/development-guidelines/token_integration.html)

Auditor harus selalu waspada terhadap token aneh saat mengintegrasikan protokol.

---

## 10. Manual Review – Temuan di TSwap

Manual review yang kita lakukan menghasilkan beberapa temuan kritis, antara lain:

- **Slippage protection** – `swapExactOutput` tidak memiliki parameter `maxInputAmount` (High).
- **Incorrect fee calculation** – penggunaan `10000` alih‑alih `1000` menyebabkan fee >90% (High).
- **Missing deadline check** – parameter `deadline` di `deposit` tidak digunakan (Medium).
- **Fee‑on‑transfer** – transfer token ekstra setiap 10 swap melanggar invariant `x * y = k` (High).
- **Parameter terbalik** pada event `LiquidityAdded` (Low).
- **`sellPoolTokens` memanggil fungsi swap yang salah** (High).
- **Return value tidak di‑assign** di `swapExactInput` (Low).
- Berbagai temuan informational (zero address check, magic numbers, event indexing, dll).

---

## Kesimpulan dan Pencapaian

Setelah menyelesaikan section ini, Anda telah:

- Memahami konsep AMM, DEX, liquidity providers, dan constant product formula.
- Mampu melakukan onboarding klien secara ekstensif.
- Menguasai stateful & stateless fuzzing dengan handler.
- Menggunakan berbagai static analysis tools (Slither, Aderyn, compiler).
- Menyadari risiko Weird ERC20s.
- Melakukan manual review mendalam dan menemukan berbagai kerentanan High, Medium, Low, dan Informational.
- Menyusun laporan audit PDF profesional.

> *"You deserve a congratulations. If you've made it this far in the course, you're doing incredibly well and are already very prepared to begin challenging live competitive audits."*