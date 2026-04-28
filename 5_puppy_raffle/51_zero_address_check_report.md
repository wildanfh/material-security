# Menulis Temuan: Missing Zero Address Check

Temuan ini berkaitan dengan praktik terbaik dalam validasi input (input validation). Meskipun sering kali dikategorikan sebagai **Informational (I)**, temuan ini sangat penting untuk menjaga integritas operasional kontrak.

### 1. Mengapa Zero Address Check Penting?
Dalam Solidity, `address(0)` adalah alamat default untuk tipe data alamat yang tidak diinisialisasi.
*   Jika `feeAddress` secara tidak sengaja disetel ke `address(0)`, dana yang dikirim ke sana (fees) bisa hilang selamanya atau menyebabkan fungsi `withdrawFees` gagal.
*   Melakukan pemeriksaan `address(0)` mencegah kesalahan manusia (human error) saat deployment atau saat pemanggilan fungsi administratif.

### 2. Memanfaatkan Hasil Aderyn
Lagi-lagi, alat analisis statis **Aderyn** telah melakukan pekerjaan berat untuk kita. Aderyn mengidentifikasi semua lokasi di mana variabel state bertipe alamat diberikan nilai tanpa adanya pengecekan nol. Auditor cukup memindahkan data ini ke laporan akhir.

### 3. Struktur Penulisan Laporan (I-3)

**[I-3] Missing checks for `address(0)` when assigning values to address state variables**

**Description:**
Penetapan nilai ke variabel state bertipe alamat dilakukan tanpa melakukan pengecekan terhadap `address(0)`. Hal ini dapat menyebabkan variabel krusial seperti alamat penerima biaya menjadi tidak valid.

**Locations (Lokasi Temuan):**
- **Constructor:** Saat inisialisasi `feeAddress`.
- **Fungsi `selectWinner`:** Saat menetapkan `previousWinner`.
- **Fungsi `changeFeeAddress`:** Saat memperbarui `feeAddress`.

**Code Snippet (Contoh):**
```solidity
function changeFeeAddress(address newFeeAddress) external onlyOwner {
    // Seharusnya ada: require(newFeeAddress != address(0), "Invalid address");
    feeAddress = newFeeAddress;
    emit FeeAddressChanged(newFeeAddress);
}
```

### 4. Workflow Auditor
Setelah temuan ini didokumentasikan, auditor memperbarui tag di dalam kode untuk menandai bahwa tugas ini telah selesai (*actioned*).
*   **Sebelum:** `// @Audit: check for zero address!`
*   **Sesudah:** `// @Written: check for zero address!`

### Kesimpulan
Meskipun terlihat sepele, melaporkan kurangnya pengecekan alamat nol menunjukkan ketelitian seorang auditor. Penggunaan alat bantu seperti Aderyn mempercepat proses penulisan laporan sehingga auditor bisa fokus pada logika kerentanan yang lebih kompleks di tahap berikutnya.
