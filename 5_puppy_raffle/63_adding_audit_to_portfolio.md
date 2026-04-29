# Menyelesaikan Laporan PDF Puppy Raffle dan Menambahkannya ke Portofolio

## Persiapan Folder `audit-data`

1. Buat folder `audit-data` di dalam repositori Puppy Raffle.
2. Salin logo (PDF) dari proyek PasswordStore sebelumnya atau siapkan logo sendiri.
3. Salin template laporan dari repo [audit-report-templating](https://github.com/Cyfrin/audit-report-templating) ke file baru bernama `report-formatted.md` di dalam `audit-data`.

## Mengisi Template Laporan

Buka `report-formatted.md` dan isi bagian-bagian berikut:

| Bagian | Isi |
|--------|-----|
| `title` | Puppy Raffle Audit Report |
| `author` | Nama Anda |
| `date` | Tanggal audit |
| `Prepared by` | Nama Anda |
| `Lead Auditors` | Nama Anda (atau tim) |
| `Protocol Summary` | Ringkasan singkat protokol (bisa ambil dari README) |
| `Disclaimer` | Isi dengan nama firma/Anda sendiri |
| `Audit Details` | Commit hash yang diaudit |
| `Scope` | `./src/` dan `#-- PuppyRaffle.sol` (ganti karakter └── dengan `#--`) |
| `Roles` | Owner dan Player (dari README) |
| `Executive Summary` | Komentar pribadi tentang pengalaman audit |
| `Issues found` | Tabel jumlah temuan per severity (High, Medium, Low, Info, Gas) |
| `Findings` | Tempelkan semua temuan dari `findings.md` ke dalam kategori masing-masing |

> **Catatan:** Perhatikan komentar `//` dalam template yang menjelaskan apa yang harus diisi di setiap bagian.

## Contoh Tabel Issues found

| Severity | Number of issues found |
| -------- | ---------------------- |
| High     | 3                      |
| Medium   | 3                      |
| Low      | 1                      |
| Info     | 7                      |
| Gas      | 2                      |
| Total    | 16                     |

## Generate PDF

Buka terminal, arahkan ke folder `audit-data`, lalu jalankan perintah:

```bash
pandoc report-formatted.md -o report.pdf --from markdown --template=eisvogel --listings
```

Jika berhasil, file `report.pdf` akan muncul di folder yang sama.

## Menambahkan ke Portofolio GitHub

1. Buka repositori portofolio yang sudah dibuat sebelumnya (misal `updraft-security-portfolio`).
2. Unggah file `report.pdf` dengan nama yang jelas dan berisi tanggal, contoh: `2024-01-12 Puppy Raffle Audit Report.pdf`.
3. Commit perubahan.
4. (Opsional) Perbarui `README.md` repositori portofolio dengan daftar audit yang telah dilakukan.

## Pemecahan Masalah Umum

| Error | Solusi |
|-------|--------|
| `.pandoc` folder tidak ditemukan | Gunakan `ls -a` untuk melihat file tersembunyi. Cek juga `/usr/share/pandoc/data/templates` |
| Tidak bisa menulis file ke direktori | Gunakan `sudo touch eisvogel.latex` |
| Tidak bisa menulis ke `eisvogel.latex` | Gunakan `sudo tee eisvogel.latex << 'EOF'` lalu tempel isi template (1068 baris) |
| `File Not Found` saat pandoc | Install `texlive-full` (`sudo apt install texlive-full`) – ukuran ~5.6GB |
| `Missing number, treated as zero` | Ganti blok LaTeX di bagian atas `report.md` dengan kode yang sudah diperbaiki (lihat materi) |

## Kesimpulan

Setelah PDF berhasil dibuat, Anda memiliki laporan audit profesional yang siap ditambahkan ke portofolio. Langkah selanjutnya adalah terus berlatih dengan codebase lain dan mengikuti kompetisi audit seperti CodeHawks First Flight.