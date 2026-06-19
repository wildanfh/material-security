# Ringkasan Materi: Reconnaissance Lanjutan (ThunderLoan)

## Pendahuluan

Setelah memahami konsep arbitrase dan flash loan, kita melanjutkan reconnaissance dengan membaca dokumentasi ThunderLoan lebih dalam. Beberapa poin penting muncul: penggunaan oracle TSwap, komposabilitas DeFi, dan rencana upgrade kontrak.

---

## Komposabilitas dan Oracle

### TSwap sebagai Oracle

Dokumentasi ThunderLoan menyebutkan:

> *"To calculate the fee, we're using the famous on-chain TSwap price oracle."*

Ini menarik karena TSwap adalah protokol yang baru saja kita audit. Penggunaan TSwap sebagai oracle menunjukkan salah satu fitur terpenting DeFi: **komposabilitas** (composability).

**Komposabilitas** berarti protokol dapat dibangun di atas protokol lain atau digabungkan dengan protokol lain. ThunderLoan memanfaatkan TSwap untuk mendapatkan data harga on-chain.

> **Penting bagi auditor:** Kerentanan dalam protokol yang digunakan (TSwap) dapat berdampak pada protokol yang menggunakannya (ThunderLoan). Ini harus selalu diingat saat melakukan security review.

### Apa Itu Oracle?

Oracle adalah sistem atau perangkat yang membawa data dunia nyata ke dalam blockchain. Contoh:
- **Chainlink Price Feeds** – oracle terdesentralisasi yang menyediakan data harga.
- **AMM DEX (seperti TSwap/Uniswap)** – dapat berfungsi sebagai oracle harga dengan membaca reserve pool.

ThunderLoan menggunakan TSwap sebagai oracle untuk menghitung biaya (fee) flash loan berdasarkan harga aset saat itu.

---

## Upgradeability (Kemampuan Upgrade)

Dokumentasi juga menyebutkan:

> *"We are planning to upgrade from the current ThunderLoan contract to the ThunderLoanUpgraded contract. Please include this upgrade in scope of a security review."*

ThunderLoan menggunakan pola upgradeable yang ditandai dengan inheritance dari:
- `Initializable`
- `OwnableUpgradeable`
- `UUPSUpgradeable`
- `OracleUpgradeable`

### Mengapa Ini Penting?

- Upgradeable contract membawa **vektor serangan baru** yang belum kita temui sebelumnya.
- Perubahan implementasi dapat memperkenalkan bug baru atau merusak state yang sudah ada.
- Proses upgrade itu sendiri harus diaudit – tidak hanya kode saat ini, tetapi juga mekanisme transisi.

> **Catatan:** Detail teknis tentang proxy dan upgradeability tidak dibahas secara mendalam di kursus ini (tersedia di Advanced Foundry). Namun, sebagai auditor, kita harus menyadari risiko yang terkait dengan kontrak upgradeable.

---

## Langkah Selanjutnya

Dengan konteks yang sudah cukup – pemahaman tentang flash loan, arbitrase, oracle, komposabilitas, dan upgradeability – kita siap untuk mulai **memeriksa kode secara langsung**.

Pada pelajaran berikutnya, kita akan mulai menggunakan alat-alat yang sudah dipelajari (Slither, Aderyn, fuzzing) untuk menemukan kerentanan awal ("low hanging fruit") di ThunderLoan.