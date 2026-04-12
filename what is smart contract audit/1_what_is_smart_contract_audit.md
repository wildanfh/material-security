# Security Review (Smart Contract Audit)

## Pendahuluan

Istilah "smart contract audit" sebenarnya kurang tepat karena mengandung implikasi jaminan atau legal. Istilah yang lebih akurat adalah **"security review"** (tinjauan keamanan). Namun kedua istilah ini sering digunakan secara bergantian di industri.

> Sebagai security researcher, Anda harus memahami bahwa audit bukanlah jaminan bebas bug, melainkan proses mencari sebanyak mungkin kerentanan dalam waktu terbatas.

---

## Tiga Fase Security Review

Berdasarkan GitHub repository, berikut tiga fase utama:

### 1. Initial Review
- **Scoping** – Menentukan batasan kontrak yang akan direview.
- **Reconnaissance** – Pengintaian, memahami kode dan arsitektur.
- **Vulnerability identification** – Identifikasi kerentanan.
- **Reporting** – Penyusunan laporan awal.

### 2. Protocol Fixes
- Tim protokol memperbaiki isu yang ditemukan.
- Melakukan retest dan menambahkan tes.

### 3. Mitigation Review
- Reconnaissance ulang terhadap perbaikan.
- Identifikasi kerentanan yang mungkin tersisa.
- Pelaporan akhir.

> Tidak ada pendekatan "one-size-fits-all". Dua metode terkenal: "the Tincho" dan "the Hans" (akan dipelajari lebih lanjut).

---

## Mengapa Security Review Penting?

- **Immutability** – Kode di blockchain tidak bisa diubah setelah deploy.
- **Lingkungan adversarial** – Blockchain bersifat permissionless, siapapun bisa menyerang.
- **Kerugian finansial** – Hampir $4 miliar dicuri tahun lalu akibat kerentanan smart contract.

### Manfaat Tambahan
- Meningkatkan pemahaman developer terhadap kode sendiri.
- Mempercepat implementasi fitur.
- Membiasakan tim dengan tren dan tooling terbaru.

### Beyond One Audit
Protocol biasanya memulai **security journey** yang mencakup:
- Formal Verification
- Competitive Audits
- Mitigation Reviews
- Bug Bounty Programs

---

## Proses Audit Tipikal

### 1. Reach Out (Penjajakan)
Protocol menghubungi tim audit. Biaya ditentukan berdasarkan:
- **Code complexity / nSLOC** (jumlah baris kode non-komentar)
- **Scope** – luasnya cakupan
- **Duration** – durasi yang diinginkan
- **Timeline** – tenggat waktu

#### Perkiraan Durasi berdasarkan nSLOC (perkiraan kasar)

| nSLOC | Estimasi Durasi |
|-------|----------------|
| 100 | 2.5 hari |
| 500 | 1 minggu |
| 1000 | 1-2 minggu |
| 2500 | 2-3 minggu |
| 5000 | 3-5 minggu |
| 5000+ | 5+ minggu |

> Ini hanya patokan kasar, sangat bervariasi tergantung kompleksitas.

Setelah kesepakatan, protocol mengirimkan **commit hash** (ID unik kode yang akan diaudit) dan **down payment**. Kemudian tanggal mulai ditetapkan.

### 2. Audit Berjalan
Auditor menggunakan segala alat yang dimiliki untuk menemukan kerentanan sebanyak mungkin.

### 3. Initial Report
Laporan awal berisi temuan yang dikategorikan berdasarkan **severity**:

| Tingkat | Deskripsi |
|---------|-----------|
| **High** | Kerentanan kritis yang dapat menyebabkan kehilangan dana atau takeover total. |
| **Medium** | Kerentanan signifikan dengan dampak terbatas atau memerlukan kondisi khusus. |
| **Low** | Kerentanan kecil, dampak minimal. |
| **Info / Non-critical** | Bukan vulnerabilitas, tetapi saran perbaikan kode dan best practices. |
| **Gas efficiencies** | Optimasi biaya gas. |

Severity untuk High/Medium/Low ditentukan oleh **impact** (dampak) dan **likelihood** (kemungkinan exploit).

### 4. Mitigation Phase
Tim protocol memiliki waktu terbatas untuk memperbaiki temuan. Biasanya mereka mengimplementasikan rekomendasi dari auditor.

### 5. Final Report
Laporan akhir berfokus pada **perbaikan yang telah dilakukan** dan memverifikasi bahwa isu sudah tertangani. Ini membangun hubungan baik untuk kolaborasi ke depan.

---

## Faktor Keberhasilan Audit

Agar audit berhasil, pastikan:

- **Dokumentasi yang baik** – jelaskan arsitektur, asumsi, dan invariant.
- **Test suite yang solid** – coverage tinggi, termasuk edge cases.
- **Kode yang jelas dan readable** – ikuti gaya konsisten.
- **Mengikuti best practices modern** – gunakan library teraudit, hindari pola usang.
- **Saluran komunikasi terbuka** – auditor adalah perpanjangan tim.
- **Video walkthrough awal** – jelaskan kode secara visual.

> Dengan memberikan konteks yang cukup, auditor dapat bekerja lebih akurat dan efisien.

---

## Setelah Audit (Post Audit)

- Audit adalah **bagian dari security journey**, bukan tujuan akhir.
- **Setiap perubahan kode** setelah audit harus direview ulang, sekecil apa pun.
- Satu audit mungkin tidak cukup – semakin banyak pihak yang mereview, semakin besar kemungkinan menemukan celah sebelum terlambat.

---

## Apa yang BUKAN Audit?

> **Audit bukan jaminan bahwa kode bebas bug.**  
> Keamanan adalah proses berkelanjutan yang terus berkembang.

---

## Kesimpulan

- Security review adalah proses **timeboxed** untuk menemukan sebanyak mungkin kerentanan.
- Terdiri dari tiga fase: initial review, protocol fixes, mitigation review.
- Severity temuan: High, Medium, Low, Info, Gas.
- Faktor keberhasilan: dokumentasi, test suite, komunikasi.
- Audit hanyalah satu langkah dalam perjalanan keamanan yang lebih besar.

> *"There is no silver bullet in smart contract auditing. But understanding the process, methods, and importance of regular security reviews can significantly enhance your protocol's robustness."*
