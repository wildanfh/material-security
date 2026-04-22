# Static Analysis dengan Slither dan Aderyn

## Pendahuluan

Audit smart contract adalah tugas yang berat namun esensial. Untuk mempermudah proses ini, tersedia alat bantu yang hebat untuk menangkap bug secara efisien. Dua alat static analysis populer adalah **Slither** dan **Aderyn**. Pengetahuan tentang alat ini tidak hanya bermanfaat bagi auditor, tetapi juga bagi developer top yang ingin meningkatkan keamanan protokol.

---

## Apa Itu Static Analysis?

**Static analysis** adalah metode pemeriksaan kode untuk mencari potensi masalah **tanpa menjalankan kode** (debugging dengan mencari pola atau keyword tertentu).

- Keuntungan: Cepat, otomatis, dapat menemukan pola kerentanan umum.
- Keterbatasan: Tidak dapat menemukan kerentanan logika bisnis yang kompleks.

Dua alat static analysis utama:
- **[Slither](https://github.com/crytic/slither)** – dikembangkan oleh Trail of Bits, menggunakan Python.
- **[Aderyn](https://github.com/Cyfrin/aderyn)** – dikembangkan oleh Cyfrin (Alex Roan), menggunakan Rust.

> **Catatan:** Alat-alat ini sebaiknya dijalankan **sebelum** melakukan audit manual.

---

## Slither: Static Analysis Berbasis Python

Slither adalah alat static analysis paling populer dan kuat, kompatibel dengan Solidity dan Vyper. Proyek open-source, memungkinkan pengembang menambahkan plugin melalui PR.

### Dokumentasi Detectors

Bagian penting dari dokumentasi Slither adalah **[Detectors](https://github.com/crytic/slither/wiki/Detector-Documentation)** – daftar lengkap kerentanan yang diperiksa oleh Slither beserta rekomendasi perbaikannya.

Contoh detector yang akan membantu mengidentifikasi masalah seperti yang ditemukan di PasswordStore (misal: akses kontrol, variabel private yang tidak benar-benar private).

### Instalasi Slither

Instruksi instalasi tersedia di repositori Slither. Pilih metode yang sesuai dengan sistem Anda. Jika mengalami masalah, gunakan AI seperti Phind atau ChatGPT untuk debugging instalasi.

**Pastikan Python sudah terinstal.** Kemudian upgrade Slither:

```bash
pip3 install --upgrade slither-analyzer
```

### Menjalankan Slither

Slither secara otomatis mendeteksi framework proyek (Hardhat, Foundry, Dapp, Brownie) dan melakukan kompilasi.

Untuk menjalankan Slither pada repositori saat ini:

```bash
slither .
```

Proses ini mungkin memakan waktu tergantung ukuran codebase. Pada Puppy Raffle, outputnya akan sangat besar.

### Output Slither: Kode Warna

Slither menggunakan kode warna untuk menunjukkan tingkat keparahan potensi masalah:

| Warna | Arti |
|-------|------|
| **Hijau** | Area yang mungkin aman, bisa jadi temuan *informational*, perlu dilihat sekilas. |
| **Kuning** | Potensi masalah terdeteksi, sebaiknya diperiksa lebih dekat. |
| **Merah** | Masalah signifikan yang harus segera diperbaiki. |

Contoh output dapat dilihat di materi video.

---

## Aderyn: Static Analysis Berbasis Rust

Aderyn adalah alat static analysis yang dibangun dengan Rust oleh Alex Roan dari Cyfrin. Fungsinya serupa dengan Slither – menelusuri *Abstract Syntax Trees* (AST) untuk menyoroti kerentanan yang dicurigai.

> Detail penggunaan Aderyn akan dibahas lebih lanjut di bagian lain kursus.

---

## Kesimpulan

- **Static analysis** mempercepat proses audit dengan mendeteksi pola kerentanan umum secara otomatis.
- **Slither** adalah alat paling populer (Python) dengan banyak detector siap pakai.
- **Aderyn** adalah alternatif berbasis Rust yang juga powerful.
- Jangan hanya mengandalkan alat – **alat static analysis tidak menggantikan review manual**.
- Jalankan alat sebelum audit manual untuk menghemat waktu dan fokus pada bug yang lebih kompleks.

> *"Always remember, static analysis tools enhance our security review, they don't replace our manual efforts!"*