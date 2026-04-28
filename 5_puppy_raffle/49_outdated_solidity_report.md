# Menulis Temuan: Versi Solidity Outdated

Temuan berikutnya yang dimasukkan ke dalam laporan adalah penggunaan versi compiler Solidity yang sudah usang (*outdated*). Ini dikategorikan sebagai temuan **Informational (I)**.

### 1. Deskripsi Masalah
Puppy Raffle menggunakan Solidity versi `0.7.6`. Compiler Solidity sering merilis versi baru yang mencakup perbaikan bug, optimasi gas, dan fitur keamanan tambahan. Menggunakan versi lama mencegah kontrak memanfaatkan pemeriksaan keamanan terbaru yang dilakukan oleh compiler.

### 2. Mengacu pada Dokumentasi Slither
Alat analisis statis seperti **Slither** sangat berguna untuk memberikan basis argumen pada temuan semacam ini. Auditor dapat merujuk pada dokumentasi Slither untuk memberikan penjelasan yang lebih otoritatif mengenai risiko penggunaan versi lama.

### 3. Struktur Penulisan Laporan (I-2)

**[I-2] Using an Outdated Version of Solidity is Not Recommended**

**Description:**
Solc (Solidity Compiler) sering merilis versi baru. Menggunakan versi lama mencegah akses ke pemeriksaan keamanan Solidity yang baru. Kami juga menyarankan untuk menghindari pernyataan pragma yang kompleks.

**Risiko yang Dipertimbangkan:**
*   Risiko terkait bug pada rilis lama.
*   Risiko dari perubahan pembuatan kode (*code generation*) yang kompleks.
*   Kehilangan fitur bahasa baru yang meningkatkan keamanan (seperti *checked arithmetic* di 0.8.0).

### 4. Rekomendasi Auditor
*   **Gunakan Versi Terbaru:** Deploy kontrak menggunakan salah satu versi Solidity terbaru (misal: `0.8.18` atau yang lebih baru saat ini).
*   **Sederhanakan Pragma:** Gunakan pernyataan pragma yang sederhana dan terkunci (locked), bukan rentang versi yang lebar.
*   **Pengujian:** Pertimbangkan untuk menggunakan versi Solidity terbaru selama proses pengujian untuk memastikan kompatibilitas dan keamanan.

### 5. Tip Penulisan
Untuk temuan kategori **Informational**, auditor sering kali tidak perlu terlalu bertele-tele. Penjelasan yang singkat, padat, dan langsung ke poin rekomendasi sudah dianggap cukup dan profesional.

### Kesimpulan
Memperbarui versi Solidity adalah salah satu langkah termudah untuk meningkatkan profil keamanan sebuah kontrak. Sebagai auditor, mendorong klien untuk beralih ke versi `0.8.x` membantu mereka menghindari bug aritmatika (overflow/underflow) yang sering terjadi di versi `0.7.x` ke bawah.
