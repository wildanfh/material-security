# Menulis Temuan Gas: Constant & Immutable

Temuan ini berfokus pada efisiensi biaya gas dengan mengidentifikasi variabel state yang nilainya tidak pernah berubah setelah diinisialisasi. Ini dikategorikan sebagai temuan **Gas (G)**.

### 1. Dasar Teori: Storage vs. Bytecode
Dalam Ethereum Virtual Machine (EVM), membaca data dari **storage** (SLOAD) adalah salah satu operasi yang paling mahal (ribuan unit gas). 
*   **`constant`:** Digunakan untuk nilai yang sudah diketahui saat kompilasi. Nilai ini langsung disuntikkan ke dalam bytecode.
*   **`immutable`:** Digunakan untuk nilai yang disetel satu kali di dalam constructor. Nilai ini juga disimpan di dalam bytecode, bukan storage.

Menggunakan kedua keyword ini akan menghemat banyak gas setiap kali variabel tersebut diakses.

### 2. Identifikasi Temuan di Puppy Raffle
Auditor menemukan beberapa variabel yang seharusnya tidak disimpan di storage:
*   `raffleDuration`: Disetel di constructor dan tidak pernah diubah (seharusnya `immutable`).
*   `commonImageUri`, `rareImageUri`, `legendaryImageUri`: Berisi string statis (seharusnya `constant`).

### 3. Struktur Penulisan Laporan (G-1)

**[G-1] Unchanged state variables should be declared constant or immutable**

**Description:**
Membaca dari storage jauh lebih mahal daripada membaca variabel constant atau immutable. Variabel-variabel berikut diinisialisasi satu kali dan tidak pernah dimodifikasi, sehingga tidak perlu menempati slot storage.

**Instances (Kasus):**
- `PuppyRaffle::raffleDuration` harus dideklarasikan sebagai `immutable`.
- `PuppyRaffle::commonImageUri` harus dideklarasikan sebagai `constant`.
- `PuppyRaffle::rareImageUri` harus dideklarasikan sebagai `constant`.
- `PuppyRaffle::legendaryImageUri` harus dideklarasikan sebagai `constant`.

### 4. Workflow Auditor
Setelah memindahkan temuan ini ke laporan, auditor memperbarui catatan di dalam kode sumber:
```javascript
// report-written: G-1 - unchanged state variables should be constant/immutable
```

### Kesimpulan
Optimasi gas melalui variabel `constant` dan `immutable` adalah "low-hanging fruit" dalam audit. Meskipun tidak memengaruhi keamanan fungsional, rekomendasi ini memberikan nilai tambah yang besar bagi klien dalam hal efisiensi operasional kontrak di mainnet yang berbiaya gas tinggi.
