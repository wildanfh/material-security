# Pengecekan Test Coverage (Cakupan Pengujian)

Setelah menyelesaikan analisis statis, langkah selanjutnya dalam audit adalah mengevaluasi kualitas kode melalui cakupan pengujian (*test coverage*).

### 1. Cara Mengecek Coverage
Dalam ekosistem Foundry, kita dapat dengan mudah mengecek berapa banyak bagian dari kode yang telah diuji oleh unit test menggunakan perintah:
```bash
forge coverage
```

### 2. Analisis Hasil pada Puppy Raffle
Pada protokol Puppy Raffle, hasil coverage menunjukkan angka yang cukup rendah (sekitar 60-70% untuk baris kode dan bahkan lebih rendah untuk fungsi).
*   **Masalah:** Banyak jalur logika (logic paths) dalam kontrak yang tidak tersentuh oleh pengujian sama sekali.
*   **Risiko:** Bagian kode yang tidak diuji adalah tempat paling umum di mana bug dan kerentanan bersembunyi.

### 3. Signifikansi dalam Audit
Kepentingan *test coverage* bergantung pada jenis audit yang dilakukan:
*   **Audit Kompetitif (Bug Bounty):** Cakupan pengujian yang rendah mungkin tidak menjadi fokus utama bagi auditor yang mencari bug, namun tetap menjadi indikator bahwa protokol tersebut memiliki banyak "blind spots" yang layak dijelajahi.
*   **Audit Privat (Hired Audit):** Auditor **wajib** melaporkan cakupan pengujian yang rendah sebagai temuan kategori **Informational**. Protokol yang baik harus memiliki coverage mendekati 100% untuk fungsi-fungsi kritikal.

### 4. Rekomendasi Auditor
Jika menemukan cakupan pengujian yang buruk, auditor harus merekomendasikan:
1.  Penambahan unit test untuk setiap fungsi publik dan eksternal.
2.  Pengujian skenario edge-case (seperti input nol, array kosong, dll).
3.  Implementasi *Integration Tests* untuk melihat interaksi antar fungsi.
4.  Pencapaian target minimal 90% coverage sebelum deployment ke mainnet.

### Kesimpulan
*Test coverage* adalah metrik kesehatan sebuah proyek. Meskipun angka yang tinggi tidak menjamin kode bebas bug, angka yang rendah hampir selalu menjamin adanya kerentanan yang belum ditemukan. Sebagai auditor, temuan ini membantu klien memahami risiko operasional proyek mereka.
