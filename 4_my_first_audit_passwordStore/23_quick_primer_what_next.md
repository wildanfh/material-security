# Langkah Selanjutnya Setelah Menemukan Temuan

## Pendahuluan

Setelah berhasil mengidentifikasi tiga temuan dalam audit PasswordStore, masih ada dua langkah penting yang harus diselesaikan sebelum laporan dapat diserahkan ke klien atau dipublikasikan:

1. **Menentukan severity (tingkat keparahan)** untuk setiap temuan.
2. **Mengonversi laporan markdown (`findings.md`) ke PDF profesional** yang dapat dibagikan.

---

## Langkah 1: Menentukan Severity Finding

Severity (tingkat keparahan) adalah penilaian seberapa **serius** suatu kerentanan berdasarkan dampak dan kemungkinan eksploitasi. Setiap temuan yang telah ditulis harus dilengkapi dengan label severity yang sesuai.

### Kategori Severity Umum

| Severity | Deskripsi |
|----------|-----------|
| **Critical / High** | Kerentanan yang dapat menyebabkan kehilangan dana, takeover total protokol, atau pelanggaran invariant utama. Contoh: Access Control pada `setPassword()`, penyimpanan password di on-chain. |
| **Medium** | Kerentanan dengan dampak signifikan tetapi terbatas, atau memerlukan kondisi khusus untuk dieksploitasi. |
| **Low** | Kerentanan dengan dampak minimal, tidak langsung mengancam dana atau fungsi kritis. |
| **Informational** | Bukan kerentanan keamanan, tetapi masalah kualitas kode atau dokumentasi. Contoh: NatSpec tidak akurat pada `getPassword()`. |
| **Gas** | Masalah efisiensi biaya gas – tidak mempengaruhi keamanan, tetapi boros. |

> Materi selanjutnya akan membahas secara mendalam cara menentukan severity dengan tepat.

---

## Langkah 2: Mengonversi ke PDF Profesional

Laporan dalam bentuk `findings.md` (markdown) perlu diubah menjadi **PDF** agar:
- Mudah dibagikan ke klien.
- Terlihat profesional.
- Dapat disimpan sebagai portofolio.

### Contoh Hasil Akhir

Laporan PDF yang sudah jadi dapat dilihat di repositori GitHub kursus:
[`3-passwordstore-audit/blob/audit-data/audit-data/report.pdf`](https://github.com/Cyfrin/3-passwordstore-audit/blob/audit-data/audit-data/report.pdf)

> PDF tersebut mencakup semua temuan dengan severity, deskripsi, PoC, dan rekomendasi mitigasi dalam format yang rapi.

---

## Kesimpulan

- Dua tugas tersisa setelah menemukan temuan: **menentukan severity** dan **membuat PDF**.
- Severity membantu klien memprioritaskan perbaikan (Critical/High harus segera ditangani).
- PDF profesional meningkatkan kredibilitas auditor dan memudahkan distribusi laporan.
