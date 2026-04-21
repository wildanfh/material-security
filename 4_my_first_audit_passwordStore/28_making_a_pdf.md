# Membuat Laporan PDF Profesional dari Markdown

## Pendahuluan

Setelah menyusun temuan dalam format Markdown, langkah selanjutnya adalah mengonversinya menjadi **PDF profesional** yang dapat dibagikan ke klien dan ditambahkan ke portofolio. Materi ini memandu penggunaan **Pandoc** dan **LaTeX** dengan template `eisvogel` untuk menghasilkan laporan yang rapi dan menarik.

---

## Alat dan Sumber Daya yang Diperlukan

| Alat | Fungsi | Tautan |
|------|--------|--------|
| **GitHub Repo** | Template dan contoh laporan | [Cyfrin/audit-report-templating](https://github.com/Cyfrin/audit-report-templating) |
| **Pandoc** | Konverter dokumen universal (Markdown → PDF) | [pandoc.org/installing](https://pandoc.org/installing.html) |
| **LaTeX** | Sistem persiapan dokumen untuk typesetting | [latex-project.org/get](https://www.latex-project.org/get/) |
| **Markdown All in One** | Ekstensi VS Code untuk memaksimalkan format Markdown | [Marketplace](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) |
| **VSCode PDF** | Pratinjau PDF di dalam VS Code | [Marketplace](https://marketplace.visualstudio.com/items?itemName=tomoki1207.pdf) |

---

## Instalasi dan Persiapan

### 1. Instal Pandoc dan LaTeX
- Ikuti petunjuk instalasi di situs resmi masing-masing.
- **Untuk pengguna WSL/Linux:** Disarankan menginstal `texlive-full` (ukuran ~5.6GB) untuk menghindari error paket hilang:
  ```bash
  sudo apt install texlive-full
```

2. Menambahkan Template eisvogel.latex

· Template ini memberitahu Pandoc bagaimana memformat PDF.
· Salin file eisvogel.latex dari repo ke folder template Pandoc.
· Lokasi folder template:
  · Umum: ~/.pandoc/templates/
  · Alternatif (Linux/WSL): /usr/share/pandoc/data/templates
· Jika folder tidak ada, buat dengan:
  ```bash
  mkdir -p ~/.pandoc/templates
  ```

3. Menambahkan Logo (Opsional)

· Siapkan file logo dalam format PDF.
· Simpan dengan nama logo.pdf di folder audit-data (sama dengan laporan).

---

Membuat File report.md

1. Buat file report.md di dalam folder audit-data.
2. Salin konten dari report-example.md ke report.md.
3. Sesuaikan bagian-bagian berikut:

Bagian Yang Diisi
Title Judul laporan (misal: "PasswordStore Security Audit Report")
Author Nama Anda
Date Tanggal audit
Prepared by Nama Anda / tim
Auditors Nama auditor (bisa lebih dari satu)
Protocol summary Deskripsi singkat protokol dan cara kerjanya
Disclaimer Isi dengan nama Anda – pernyataan bahwa laporan bukan jaminan bebas bug
Risk classifications Kriteria severity (High, Medium, Low, Informational)
Audit details Commit hash yang menjadi acuan
Scope Daftar kontrak yang direview (gunakan #-- sebagai pengganti └── karena akan error di PDF)
Audit roles Peran-peran dalam protokol (owner, pengguna, dll)
Executive summary Ringkasan proses penilaian
Severity and number of issues Tabel ringkasan jumlah temuan per severity
Findings Tempelkan laporan temuan yang sudah dibuat ke dalam kategori severity masing-masing

Catatan: Karakter (tree line) akan menyebabkan error saat konversi PDF. Ganti dengan #-- atau hapus.

---

Menghasilkan PDF

Buka terminal, arahkan ke folder audit-data, lalu jalankan perintah:

```bash
pandoc report.md -o report.pdf --from markdown --template=eisvogel --listings
```

Jika berhasil, file report.pdf akan muncul di folder yang sama.

---

Pemecahan Masalah (Troubleshooting)

Error Kemungkinan Penyebab Solusi
Folder .pandoc tidak ditemukan File tersembunyi atau lokasi berbeda Gunakan ls -a untuk melihat file tersembunyi. Cek juga di /usr/share/pandoc/data/templates
Tidak bisa menulis file ke direktori Izin pengguna (permission denied) Gunakan sudo (contoh: sudo touch eisvogel.latex)
Tidak bisa menulis ke eisvogel.latex Izin menulis Gunakan sudo tee eisvogel.latex << 'EOF' lalu tempel isi template (1068 baris), akhiri dengan EOF
pandoc: report.md: openFile: does not exist LaTeX缺少 komponen Instal texlive-full (sudo apt install texlive-full)
"Missing number, treated as zero" Error sintaks LaTeX di report.md Ganti blok LaTeX di bagian atas report.md dengan kode yang telah diperbaiki (lihat materi)

Catatan: texlive-full berukuran sekitar 5.6GB. Pastikan koneksi stabil dan ruang penyimpanan cukup.

---

Hasil Akhir

Setelah berhasil, laporan PDF akan terlihat profesional dengan:

· Halaman judul berlogo
· Daftar isi otomatis (jika template mendukung)
· Tabel severity
· Temuan yang terstruktur rapi

Laporan ini dapat:

· Dibagikan ke klien
· Ditambahkan ke portofolio sebagai bukti kemampuan audit

---

Kesimpulan

· Pandoc + LaTeX + template eisvogel adalah solusi andal untuk menghasilkan PDF profesional dari Markdown.
· Persiapkan template dan konfigurasi dengan benar agar proses berjalan lancar.
· Gunakan texlive-full untuk menghindari error paket LaTeX yang hilang.
· Simpan laporan akhir sebagai portofolio untuk menunjukkan keterampilan security review.
