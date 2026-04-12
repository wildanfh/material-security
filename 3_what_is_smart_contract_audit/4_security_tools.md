# Tools Security

## Pendahuluan

Berbagai alat digunakan dalam security review smart contract. Mulai dari test suite bawaan framework hingga formal verification. Namun, penting untuk diketahui bahwa **sekitar 80% bug (termasuk dalam competitive audit) tidak dapat dideteksi secara otomatis oleh mesin**, termasuk AI saat ini.

---

## 1. Test Suites (Garis Pertahanan Pertama)

Test suite adalah fondasi awal keamanan proyek. Contoh framework:

- Foundry
- Hardhat
- Brownie
- Apeworx
- Remix (juga memiliki fitur test)

> *Rest in Peace Truffle* 😢

Konsep penting: **test coverage** – seberapa banyak kode yang tercakup oleh pengujian. Akan dibahas lebih lanjut.

---

## 2. Static Analysis (Analisis Statis)

Memeriksa masalah **tanpa menjalankan kode** (static). Alat-alat populer:

| Alat | Deskripsi |
|------|-----------|
| **Slither** | Alat analisis statis populer untuk Solidity. |
| **4nalyzer** | Alat analisis untuk codebase Ethereum. |
| **Mythril** | Alat keamanan EVM bytecode. |
| **Aderyn** | Alat analisis statis modern. |

Dalam kursus ini akan banyak menggunakan **Slither** dan **Aderyn**.

---

## 3. Fuzz Testing (Pengujian dengan Data Acak)

Dua jenis utama fuzz testing:

| Jenis | Deskripsi |
|-------|-----------|
| **Stateless fuzzing** | Data input acak, state direset setiap run. |
| **Stateful fuzzing (invariant test)** | Data acak + urutan fungsi acak, state berlanjut. |

### Jenis lain (tidak dibahas di kursus ini):
- **Differential test** – membandingkan hasil dari dua implementasi.
- **Chaos test** – menguji ketahanan sistem terhadap kondisi kacau.

> Disarankan untuk tetap mempelajarinya di luar kursus guna memperluas wawasan keamanan.

---

## 4. Formal Verification (Verifikasi Formal)

Verifikasi formal menggunakan **metode matematis** untuk membuktikan kebenaran kode. Pendekatan umum: **symbolic execution** – mengubah fungsi Solidity menjadi ekspresi matematika atau boolean.

### Alat formal verification:

| Alat | Deskripsi |
|------|-----------|
| **Manticore** | Symbolic execution tool untuk smart contract. |
| **Certora** | Platform verifikasi formal komersial. |
| **Z3** | Theorem prover dari Microsoft (digunakan sebagai backend). |

> Akan dibahas lebih mendalam di bagian selanjutnya kursus ini.

---

## 5. AI Tools (Belum Matang)

AI tools seperti ChatGPT **belum** memberikan nilai substansial untuk mengamankan codebase. Saat ini hanya berguna untuk:

- Sanity check (pemeriksaan cepat)
- Membantu mencari sesuatu dengan cepat

> **Peringatan:** Jika suatu proyek mengklaim telah diaudit oleh AI (misal ChatGPT), Anda harus **skeptis** dan mempertanyakan apakah proyek tersebut benar-benar serius dengan keamanan.

### Sumber Daya Tambahan
GitHub repo [Web3Bugs oleh ZhangZhuoSJTU](https://github.com/ZhangZhuoSJTU/Web3Bugs) menyediakan contoh bug yang dapat vs tidak dapat dideteksi oleh mesin.

---

## Fakta Kritis: 80% Bug Tidak Terdeteksi Mesin

> **Sekitar 80% bug nyata dan bug dalam competitive audit tidak dapat dideteksi secara otomatis oleh mesin, termasuk AI modern.**

### Implikasi:

| Fakta | Makna |
|-------|-------|
| Tools saat ini belum memadai | Kita membutuhkan alat yang lebih baik. |
| **Human auditor tetap sangat penting** | Sebagian besar bug berasal dari **business logic** dan implementasi yang salah, bukan dari odditas Solidity atau kriptografi. |

Perbedaan antara bug yang terdeteksi mesin vs manusia akan dipelajari lebih lanjut di kursus ini.

---

## Ringkasan Alat Berdasarkan Jenis

| Kategori | Contoh Alat | Keterangan |
|----------|-------------|-------------|
| Test Suites | Foundry, Hardhat, Brownie | Garis pertahanan pertama, menguji fungsionalitas. |
| Static Analysis | Slither, Aderyn, Mythril | Cari bug tanpa eksekusi kode. |
| Fuzzing | Foundry (fuzz & invariant) | Uji dengan input/aksi acak. |
| Formal Verification | Certora, Manticore, Z3 | Bukti matematis kebenaran. |
| AI Tools | ChatGPT | Belum substansial, gunakan dengan skeptis. |

---

## Kesimpulan

- Gunakan **semua lapisan alat** (test suite, static analysis, fuzzing, formal verification) untuk keamanan maksimal.
- **Jangan hanya mengandalkan alat otomatis** – 80% bug tidak terdeteksi mesin.
- **Human auditor** adalah komponen paling kritis karena memahami konteks bisnis dan logika kompleks.
- AI tools saat ini hanya pendukung, bukan pengganti.

> *"Human auditors and human security researchers remain paramount."*
