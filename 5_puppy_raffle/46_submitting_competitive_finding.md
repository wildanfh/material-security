# Prosedur Pengiriman (Submission) Temuan Audit Kompetitif

Menemukan bug hanyalah setengah dari pekerjaan; setengah sisanya adalah meyakinkan juri dan protokol melalui laporan yang berkualitas. Pada audit kompetitif di CodeHawks, kualitas laporan sangat menentukan besarnya hadiah yang diterima.

### 1. Komponen Utama Submission
Saat Anda mengklik **"Submit a Finding"** pada kontes yang aktif, Anda perlu mengisi beberapa kolom penting:

*   **Title (Judul):** Judul yang baik harus mencakup **Root Cause + Impact**. 
    *   *Contoh Buruk:* "Loop problem in enterRaffle"
    *   *Contoh Baik:* "Classic DOS via unbounded loop in enterRaffle causes gas exhaustion for new players"
    *   Catatan: Anda tidak perlu menambahkan ID seperti [H-1], juri akan menambahkannya nanti jika temuan valid.
*   **Severity (Tingkat Keparahan):** Pilih antara High, Medium, atau Low/Informational sesuai dengan dampak teknisnya.
*   **Relevant GitHub Links:** Jangan hanya menautkan alamat file. Gunakan **GitHub Permalink** dengan cara klik kanan pada nomor baris di GitHub dan pilih "Copy Permalink". Ini membantu juri langsung melihat titik masalah.

### 2. Struktur Isi Laporan (The Finding)
CodeHawks menyediakan template standar yang mencakup:
1.  **Summary:** Ringkasan singkat masalah.
2.  **Vulnerability Details:** Penjelasan teknis mendalam tentang bagaimana bug bekerja.
3.  **Impact:** Dampak nyata bagi protokol (kehilangan dana, DoS, dll).
4.  **Tools Used:** Alat yang digunakan (manual review, Foundry, Slither, dll).
5.  **Recommendations:** Saran perbaikan yang konkret.

### 3. Proof of Concept (PoC) — Wajib!
Dalam audit kompetitif, **PoC (Proof of Concept)** dalam bentuk kode (Proof of Code) hampir bersifat wajib untuk temuan High dan Medium. Tanpa bukti kode yang menunjukkan bug tersebut benar-benar bisa dieksploitasi, kemungkinan besar temuan Anda akan ditolak atau diturunkan tingkat keparahannya.

### 4. Selected Report & Bonus Payout
Jika banyak auditor menemukan bug yang sama (duplikat), juri akan memilih **satu laporan terbaik** sebagai "Selected Report".
*   **Kriteria:** Penjelasan paling jelas, rekomendasi paling tepat, dan PoC yang paling bersih.
*   **Keuntungan:** Penulis *Selected Report* akan mendapatkan **bonus pembayaran** di luar bagi hasil standar.

### 5. Fleksibilitas & Portofolio
*   Selama kontes masih dibuka, Anda bisa mengubah (edit) atau menghapus temuan yang sudah dikirim.
*   Setelah kontes berakhir, Anda bisa mengunduh temuan Anda dalam format Markdown melalui menu **"My Findings"**. Tambahkan ini ke portofolio GitHub Anda untuk menunjukkan pengalaman nyata.

### Kesimpulan
Kualitas penulisan laporan adalah pembeda antara auditor amatir dan profesional. Dengan menulis laporan yang jelas dan menyertakan bukti kuat, Anda tidak hanya meningkatkan peluang menang tetapi juga membangun reputasi yang solid di industri keamanan Web3.
