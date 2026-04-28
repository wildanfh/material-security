# Pitstop: Langkah Akhir Menuju Laporan Audit

Materi ini merupakan titik persiapan sebelum kita menyelesaikan seluruh rangkaian audit pada protokol Puppy Raffle. Ada beberapa langkah krusial yang akan dilakukan untuk memastikan tidak ada temuan yang terlewat.

### 1. Analisis Statis (Slither & Aderyn)
Kita akan menjalankan dan meninjau laporan dari dua alat analisis statis utama:
*   **Slither:** Digunakan untuk mendeteksi kerentanan umum, detektor optimasi gas, dan masalah kualitas kode.
*   **Aderyn:** Alat analisis statis yang lebih baru dan spesifik untuk ekosistem Solidity/Foundry yang memberikan laporan dalam format yang sangat mudah dibaca.
*   *Tujuan:* Membandingkan temuan manual kita dengan temuan otomatis untuk memastikan cakupan audit 100%.

### 2. Kualitas Kode & Pengujian (Code Quality/Tests)
Kita akan mengevaluasi infrastruktur pengujian yang sudah ada dalam repo:
*   Apakah test coverage sudah cukup?
*   Apakah ada test case yang gagal atau tidak mencakup skenario serangan yang kita temukan?
*   Menyiapkan **Proof of Concept (PoC)** untuk membuktikan kerentanan yang telah kita temukan sebelumnya.

### 3. Audit Kompetitif (CodeHawks)
Pengenalan singkat mengenai platform audit kompetitif seperti **CodeHawks**:
*   Cara berpartisipasi dalam kontes audit.
*   Prosedur pengiriman (submission) temuan yang profesional.
*   Bagaimana membangun reputasi sebagai auditor di platform publik.

### 4. Penulisan Laporan Akhir (Final Reporting)
Langkah paling penting dalam karier auditor adalah menuangkan temuan ke dalam dokumen laporan yang profesional.
*   Laporan akan menyertakan deskripsi masalah, dampak (impact), probabilitas (likelihood), rekomendasi mitigasi, dan kode PoC.
*   Contoh laporan final (Markdown & PDF) serta hasil output alat analisis statis tersedia di branch `audit-data` pada repositori kursus sebagai referensi.

### Apa Selanjutnya?
Setelah menyelesaikan langkah-langkah di atas, Anda akan memiliki satu laporan audit profesional yang siap ditambahkan ke dalam **portofolio security review** Anda. Ini adalah bukti nyata kompetensi Anda sebagai Smart Contract Security Researcher.
