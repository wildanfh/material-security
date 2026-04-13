# Reconnaissance – Memahami Codebase dengan Metode Tincho

## Pendahuluan

Setelah fase scoping selesai, kita memasuki fase **reconnaissance** (pengintaian). Tujuan utamanya adalah memahami codebase secara menyeluruh sebelum mulai mencari kerentanan. Materi ini mempraktikkan metode **The Tincho** pada proyek PasswordStore.

---

## Langkah 1: Clone Repositori dan Buka di VS Code

```bash
git clone https://github.com/Cyfrin/3-passwordstore-audit.git
cd 3-passwordstore-audit
code .
```

---

Langkah 2: Baca Dokumentasi (README)

· Buka file README.md di VS Code.
· Gunakan CTRL + SHIFT + V untuk membuka preview mode (tampilan markdown yang sudah diformat).
· Alternatif: buka command palette (CTRL + SHIFT + P), ketik markdown open preview.

Pastikan ekstensi Markdown Preview sudah terinstal di VS Code jika tidak berfungsi.

Apa yang Harus Dipahami dari Dokumentasi?

· Tujuan protokol.
· Fitur dan batasan.
· Peran aktor (owner, pengguna lain).
· Invariant yang dijanjikan.

Mulai Berpikir Seperti Penyerang

Saat membaca dokumentasi, mulailah mengajukan pertanyaan adversarial:

"Is there any way for an unauthorized user to access a stored password?"

Catat semua potensi celah yang terlintas di pikiran.

---

Langkah 3: Organisasi File dan Penilaian Kompleksitas

Mengikuti saran Tincho, langkah berikutnya adalah mengorganisasi file-file yang in scope dan menilai kompleksitas masing-masing.

Install Ekstensi Solidity Metrics

· Nama ekstensi: Solidity Metrics
· Pengembang: tintinweb
· Fungsi: Memberikan metrik tentang kode Solidity (jumlah baris, kompleksitas siklomatis, inheritance graph, call graph, dll)

Cara Menggunakan

1. Klik kanan pada folder yang berisi kontrak (misal src/).
2. Pilih Solidity: Metrics dari menu konteks.
3. Ekstensi akan memproses file dan menampilkan laporan.

Pro-tip: Jika repo memiliki lebih dari satu folder yang relevan, gunakan CTRL + Click untuk memilih beberapa folder sekaligus.

Ekspor Laporan

· Setelah laporan muncul, buka command palette (CTRL + SHIFT + P).
· Ketik export this metrics report.
· Pilih format ekspor (biasanya HTML).
· Simpan untuk referensi di kemudian hari.

---

Langkah 4: Memanfaatkan Data dari Solidity Metrics

Dalam laporan, perhatikan beberapa bagian penting:

Bagian Kegunaan
File list & line counts Mengetahui ukuran setiap file. Gunakan untuk memprioritaskan review (mulai dari yang kecil).
Inheritance Graph Memahami hierarki pewarisan kontrak. Kerentanan sering muncul di rantai pewarisan yang kompleks.
Call Graph Melihat fungsi mana memanggil fungsi lain. Berguna untuk melacak alur eksekusi dan potensi re-entrancy.
Contracts Summary Ringkasan setiap kontrak: jumlah fungsi, modifier, event, dll.

Jika Banyak File

· Salin data jumlah baris per file ke Google Sheets atau Notion.
· Urutkan dari yang terkecil ke terbesar.
· Mulai review dari file yang lebih kecil dan mudah dipahami terlebih dahulu.

Untuk PasswordStore, hanya ada satu file → langkah ini tidak terlalu diperlukan, tetapi tetap biasakan untuk proyek yang lebih besar.

---

Kesimpulan dan Langkah Selanjutnya

· Langkah awal reconnaissance: baca dokumentasi → pahami konteks → gunakan tools (Solidity Metrics) → organisasi file.
· Dengan memahami codebase dan fungsinya, Anda siap untuk beralih ke pemeriksaan kode secara mendalam.
· Tools seperti CLOC dan Solidity Metrics membantu Anda mengukur kompleksitas dan memprioritaskan area yang paling kritis.

"Understanding your codebase and its functionalities is the first step towards securing it."
