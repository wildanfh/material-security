# Ringkasan Materi: Rekap Lengkap Audit PasswordStore

## Pendahuluan

Materi ini adalah **rekap akhir** dari seluruh proses security review yang telah dipelajari dan dipraktikkan pada protokol PasswordStore. Dari tahap onboarding, pengenalan tools, fase audit, metode Tincho, penulisan laporan, hingga pembuatan PDF profesional dan portofolio.

---

## 1. Onboarding Protocol

Sebelum audit dimulai, auditor harus melakukan **onboarding** untuk memastikan protokol siap diaudit. Jangan pernah menerima hanya tautan Etherscan tanpa konteks.

### Pertanyaan Minimal yang Harus Diajukan (Minimal Onboarding Questions)

| Kategori | Informasi yang Digali |
|----------|----------------------|
| **About the Project** | Ringkasan proyek, tujuan, invariant. |
| **Setup** | Tools yang dibutuhkan (Foundry, Hardhat, dll), cara build. |
| **Testing** | Cara menjalankan test, melihat coverage. |
| **Scope** | Commit hash spesifik, daftar kontrak yang diaudit. |
| **Compatibilities** | Rantai target, versi Solidity, token yang digunakan. |
| **Roles** | Aktor dalam sistem (owner, user, admin), hak akses masing-masing. |
| **Known Issues** | Masalah yang sudah diketahui tim protokol. |

> Auditor berperan sebagai **pendidik** – menjelaskan mengapa persiapan ini penting.

---

## 2. Tools untuk Menilai Ukuran dan Kompleksitas Codebase

| Alat | Fungsi |
|------|--------|
| **CLOC** | Menghitung jumlah baris kode (nSLOC) untuk estimasi durasi. |
| **Solidity Metrics** | Memberikan metrik kompleksitas, inheritance graph, call graph, dan breakdown kontrak. |

Nilai utama tools ini: **membantu memperkirakan beban kerja dan waktu yang dibutuhkan** untuk audit menyeluruh.

---

## 3. Tiga Fase Audit

| Fase | Sub-fase |
|------|----------|
| **Initial Review** | Scoping → Reconnaissance → Vulnerability Identification → Reporting |
| **Protocol Fixes** | Protocol memperbaiki isu, menambah test, retest. |
| **Mitigation Review** | Reconnaissance ulang → Identifikasi kerentanan sisa → Pelaporan akhir |

> Fokus kursus ini pada **Initial Review**.

---

## 4. Metode Tincho (The Red Guild)

Pendekatan dari legenda audit **Tincho Abbate**:

- **Baca dokumentasi** – pahami dulu sebelum lihat kode.
- **Catat sering** – langsung di dalam kode (komentar `// @Audit`) dan di file terpisah.
- **Mulai dari yang kecil** – audit kontrak paling sederhana dulu, baru yang kompleks.
- **Gunakan tools** – Solidity Metrics untuk memetakan hierarki kompleksitas.

---

## 5. Hasil Audit PasswordStore: Tiga Temuan

| ID | Judul | Severity |
|----|-------|----------|
| H-1 | `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password | High |
| H-2 | Storing the password on-chain makes it visible to anyone and no longer private | High |
| I-1 | The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect | Informational |

---

## 6. Menentukan Severity (Tingkat Keparahan)

Severity ditentukan oleh kombinasi **Impact** dan **Likelihood**.

### Impact (Dampak)

| Tingkat | Kriteria |
|---------|----------|
| **High** | Dana langsung atau hampir langsung berisiko; gangguan parah pada fungsionalitas. |
| **Medium** | Dana tidak langsung berisiko; ada gangguan fungsional tingkat sedang. |
| **Low** | Dana tidak berisiko; fungsi salah atau state ditangani tidak tepat. |

### Likelihood (Kemungkinan)

| Tingkat | Kriteria |
|---------|----------|
| **High** | Sangat mungkin terjadi (misal: peretas bisa memanggil fungsi langsung). |
| **Medium** | Mungkin terjadi dalam kondisi spesifik (misal: token ERC20 aneh digunakan). |
| **Low** | Tidak mungkin terjadi (misal: variabel sulit diubah disetel ke nilai unik di waktu tertentu). |

### Matriks Severity

| (Impact \ Likelihood) | High | Medium | Low |
|----------------------|------|--------|-----|
| **High** | H | H/M | M |
| **Medium** | H/M | M | M/L |
| **Low** | M | M/L | L |

---

## 7. Template Laporan Temuan

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:** Succinctly detail the vulnerability

**Impact:** The affects the vulnerability has

**Proof of Concept:** Programmatic proof of how the vulnerability is exploited

**Recommended Mitigation:** Recommendations on how to fix the vulnerability
```

PoC (Proof of Concept) sangat penting – bukti tak terbantahkan bahwa kerentanan ada.

---

8. Timeboxing (Manajemen Waktu)

· Tidak ada audit yang benar-benar "selesai" – selalu ada baris kode lain yang bisa diperiksa.
· Tetapkan batasan waktu (time-box) untuk setiap fase.
· Timeboxing mencegah perfeksionisme berlebihan dan menjaga efisiensi.

---

9. Membuat PDF Profesional

Tools yang Digunakan

Tool Fungsi
Pandoc Konverter Markdown → PDF
LaTeX Sistem typesetting untuk dokumen teknis
eisvogel.latex Template format PDF
Markdown All in One Ekstensi VS Code untuk markdown
VSCode PDF Pratinjau PDF di VS Code

Perintah Generate PDF

```bash
pandoc report.md -o report.pdf --from markdown --template=eisvogel --listings
```

Komponen Laporan PDF

· Halaman judul dengan logo
· Disclaimer (laporan bukan jaminan bebas bug)
· Klasifikasi risiko
· Ringkasan eksekutif
· Tabel jumlah temuan per severity
· Temuan lengkap (Description, Impact, PoC, Mitigation)

---

10. Portofolio di GitHub

· Buat repositori publik (misal: security-audit-portfolio)
· Unggah laporan PDF dengan nama berformat: YYYY-MM-DD Protocol Audit Report.pdf
· Tambahkan README.md yang menjelaskan isi repositori dan perjalanan keamanan Anda.
· Bagikan tautan ke klien atau rekruter.

---

Kesimpulan

· Onboarding yang matang adalah fondasi audit yang sukses.
· Gunakan tools (CLOC, Solidity Metrics) untuk estimasi dan navigasi codebase.
· Ikuti fase audit (scoping → reconnaissance → identifikasi → reporting).
· Terapkan metode Tincho (baca docs, catat, mulai dari kecil).
· Tentukan severity dengan matriks Impact × Likelihood.
· Tulis laporan dengan template terstruktur (Title, Description, Impact, PoC, Mitigation).
· Timeboxing – tahu kapan harus berhenti.
· Buat PDF profesional dengan Pandoc + LaTeX.
· Bangun portofolio di GitHub untuk menunjukkan kemampuan.