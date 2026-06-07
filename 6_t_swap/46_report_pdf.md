# Ringkasan Materi: Membuat PDF Laporan Audit TSwap

## Pendahuluan

Setelah menyelesaikan semua temuan (findings) dalam audit TSwap, langkah terakhir adalah **membuat laporan PDF profesional** dan menambahkannya ke portofolio GitHub. Proses ini sudah dilakukan beberapa kali sebelumnya (PasswordStore, Puppy Raffle), sehingga diharapkan menjadi lebih mudah.

---

## Langkah-langkah Pembuatan PDF

### 1. Persiapan Folder `audit-data`

Pastikan folder `audit-data` berisi:
- `findings.md` – berisi semua temuan yang sudah ditulis (High, Medium, Low, Informational).
- `logo.pdf` – logo perusahaan atau personal untuk halaman judul.

### 2. Membuat File Template Laporan

Salin file template `report-example.md` dari [repo audit-report-templating](https://github.com/Cyfrin/audit-report-templating/blob/main/report-example.md) ke dalam `audit-data` dengan nama yang mencerminkan tanggal dan protokol, misalnya:

```
2024-04-21-tswap-audit.md
```

### 3. Mengisi Template

Edit file template tersebut secara menyeluruh:

| Bagian | Yang Diisi |
|--------|------------|
| **Judul, penulis, tanggal** | Nama protokol, tim auditor, tanggal audit. |
| **Prepared by / Lead Auditors** | Nama Anda atau tim. |
| **Protocol Summary** | Ringkasan singkat tentang TSwap (apa yang dilakukan protokol). |
| **Disclaimer** | Pernyataan bahwa laporan bukan jaminan bebas bug. |
| **Risk Classification** | Matriks severity (Impact vs Likelihood) – sudah tersedia. |
| **Audit Details & Scope** | Commit hash, kontrak yang diaudit (`PoolFactory.sol`, `TSwapPool.sol`). |
| **Roles** | Peran dalam protokol (owner, liquidity provider, swapper). |
| **Executive Summary** | Kesimpulan umum dari proses audit. |
| **Issues found** | Tabel ringkasan jumlah temuan per severity (High, Medium, Low, Info). |
| **Findings** | **Salin semua temuan dari `findings.md`** ke bagian yang sesuai (High, Medium, Low, Informational). |

> **Kreativitas:** Anda dapat menyesuaikan format dan detail laporan sesuai preferensi. Yang penting semua temuan tercantum dengan jelas.

### 4. Menjalankan Perintah Pandoc

Buka terminal, arahkan ke folder `audit-data`, lalu jalankan:

```bash
pandoc 2024-04-21-tswap-audit.md -o report.pdf --from markdown --template=eisvogel --listings
```

**Parameter:**
- `2024-04-21-tswap-audit.md` – file template yang sudah diisi.
- `-o report.pdf` – output file PDF.
- `--from markdown` – format input markdown.
- `--template=eisvogel` – menggunakan template LaTeX yang sudah disiapkan.
- `--listings` – untuk format kode yang lebih baik.

Jika berhasil, file `report.pdf` akan muncul di folder yang sama.

---

## Contoh Hasil Akhir

Laporan PDF yang dihasilkan akan terlihat profesional, dengan struktur:
- Halaman judul berlogo
- Daftar isi
- Ringkasan protokol
- Klasifikasi risiko
- Tabel jumlah temuan
- Rincian temuan per severity (lengkap dengan deskripsi, impact, PoC, rekomendasi mitigasi)

Contoh laporan akhir dapat dilihat di:
[`5-t-swap-audit/audit-data/report.pdf`](https://github.com/Cyfrin/5-t-swap-audit/blob/audit-data/audit-data/report.pdf)

---

## Menambahkan ke Portofolio GitHub

1. Buka repositori portofolio Anda (misal `updraft-security-portfolio`).
2. Unggah file `report.pdf` dengan nama deskriptif, misal: `2024-04-21_TSwap_Audit_Report.pdf`.
3. Commit dan push.
4. (Opsional) Perbarui `README.md` dengan tautan ke laporan terbaru.

---

## Kesimpulan

- **Template laporan** memudahkan konsistensi format antar audit.
- **Pandoc + eisvogel** menghasilkan PDF berkualitas tinggi dari markdown.
- Setiap selesai audit, segera tambahkan PDF ke portofolio untuk menunjukkan perkembangan kemampuan.
- Proses ini semakin cepat setelah dilakukan beberapa kali.