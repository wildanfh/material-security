# Menulis Temuan Pertama: Floating Pragma

Setelah menyelesaikan tahap investigasi, saatnya memindahkan catatan dari basis kode ke dalam laporan resmi. Temuan pertama yang kita buat adalah mengenai penggunaan **Floating Pragma**.

### 1. Memanfaatkan Tag `@Audit`
Langkah awal dalam menulis laporan adalah menelusuri kembali catatan yang telah kita buat di dalam kode menggunakan pencarian (search) untuk tag `@Audit`. Ini membantu kita memastikan tidak ada temuan yang terlewat.

### 2. Menggunakan Output Alat (Aderyn)
Alat seperti **Aderyn** sangat membantu karena memberikan kerangka laporan yang sudah terformat dalam Markdown. Auditor dapat mengambil konten dari laporan Aderyn dan menyesuaikannya.

**Contoh Output Aderyn:**
> ## L-2: Solidity pragma should be specific, not wide
> Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

### 3. Penyesuaian Tingkat Keparahan (Severity)
Meskipun Aderyn mengategorikan temuan ini sebagai *Low* (L), auditor memiliki kebebasan untuk mengubahnya berdasarkan kebijakan atau konteks proyek. Dalam materi ini, auditor memutuskan untuk mengategorikannya sebagai **Informational (I)**.

### 4. Struktur Penulisan Laporan (I-1)
Berikut adalah format penulisan temuan yang rapi:

**[I-1] Solidity pragma should be specific, not wide**

**Description:**
Mempertimbangkan untuk menggunakan versi Solidity yang spesifik di dalam kontrak daripada versi yang lebar (wide). Penggunaan simbol `^` memungkinkan kompilasi dengan versi compiler yang berbeda-beda.

**Location:**
- Found in `src/PuppyRaffle.sol` [Line: 3]

**Code Snippet:**
```solidity
pragma solidity ^0.7.6;
```

### 5. Workflow Auditor: Menandai Progress
Sangat penting bagi auditor untuk menandai catatan di dalam kode setelah temuan tersebut selesai ditulis ke dalam laporan. Hal ini mencegah duplikasi atau kebingungan.
*   **Sebelum:** `// @Audit: Floating Pragma is bad`
*   **Sesudah:** `// report-written: use of floating pragma is bad!`

### Kesimpulan
Menulis laporan bukan berarti harus mulai dari nol. Memanfaatkan alat analisis statis dan template yang ada sangat disarankan untuk menjaga efisiensi, selama auditor tetap melakukan penyesuaian yang diperlukan agar laporan tersebut akurat dan profesional.
