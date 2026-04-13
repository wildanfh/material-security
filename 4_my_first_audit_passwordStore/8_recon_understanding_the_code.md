# Memahami Codebase – Pendekatan Tincho dalam Code Inspection

## Pendahuluan

Setelah fase reconnaissance dan pengorganisasian file, langkah selanjutnya adalah **memahami codebase secara mendalam** dengan membaca kode baris per baris. Pendekatan Tincho sangat pragmatis: ia tidak langsung mencari bug, tetapi **berusaha memahami fungsionalitas dan arsitektur kode** terlebih dahulu.

> *"Understanding the functionalities and architecture of the code forms the first and most important part of code inspection."*

---

## Langkah 1: Mulai dari Awal – Baca Kode Baris per Baris

Gunakan dokumentasi klien sebagai panduan tentang apa yang **seharusnya** dilakukan protokol. Untuk PasswordStore:

> *"A user should be able to store and retrieve their password, no one else should be able to see it."*

Saat membaca kode, cari apakah fungsionalitas tersebut benar-benar diimplementasikan.

---

## Langkah 2: Periksa Bagian Atas Kontrak (License, Pragma)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;
```

Elemen Yang Perlu Diperhatikan
License Biasanya fine, pastikan tidak ada license yang tidak tepat.
Compiler version Apakah versi terbaru? Jika tidak, catat sebagai pertanyaan.

Teknik Mencatat dengan Format Kustom

Gunakan komentar dengan prefix yang konsisten untuk memudahkan pencarian nanti:

```solidity
// Q: Is this the correct compiler version?
```

Contoh prefix yang bisa digunakan:

Prefix Arti
// Q: Pertanyaan untuk klien
// I: Informational finding (bukan bug, tapi saran perbaikan)
// H:, // M:, // L: Potensi vulnerability dengan severity tertentu

Dengan format ini, Anda bisa mencari semua // Q: di seluruh codebase untuk mengumpulkan pertanyaan.

---

Langkah 3: Baca NatSpec sebagai Dokumentasi Tambahan

```solidity
/*
 * @author not-so-secure-dev
 * @title PasswordStore
 * @notice This contract allows you to store a private password that others won't be able to see.
 * You can update your password at any time.
 */
```

NatSpec adalah extended documentation yang sangat berharga. Catat intent protokol di file catatan terpisah (misal .notes.md).

---

Langkah 4: Perhatikan Naming Conventions

Jika klien menggunakan konvensi penamaan yang jelas (seperti s_password untuk storage variable), itu bagus. Jika tidak, buat catatan informational:

```solidity
// I - naming convention could be more clear ie 'error PasswordStore__NotOwner();'
error NotOwner();
```

---

Langkah 5: Analisis Fungsi Satu per Satu

Lihat fungsi setPassword():

```solidity
/*
 * @notice This function allows only the owner to set a new password.
 * @param newPassword The new password to set.
 */
function setPassword(string memory newPassword) external {
    s_password = newPassword;
    emit SetNetPassword();
}
```

Pertanyaan Kritis yang Harus Diajukan

· Apakah benar hanya owner yang bisa memanggil? (Cek apakah ada modifier onlyOwner – dalam kode ini tidak ada! Ini temuan potensial.)
· Apakah event yang di-emit sudah tepat? (SetNetPassword – mungkin typo? Seharusnya SetNewPassword?)

Jika Dokumentasi Kurang Jelas

"Sometimes a protocol won't have clear documentation. This is where clear lines of communication with the client are fundamental."

Tinggalkan komentar // Q: What's this function do? lalu tanyakan langsung ke klien.

---

Alat Bantu yang Direkomendasikan

Alat Kegunaan
Headers (by transmissions11) Package untuk memberi label area-area dalam repo yang sedang direview, memudahkan navigasi.
File .notes.md Tempat dumping semua pemikiran, potensi attack vectors, dan pertanyaan.
Print source code + highlighter Beberapa researcher (seperti 0Kage dari Cyfrin) mencetak kode dan menggunakan highlighter warna berbeda untuk memvisualisasikan codebase.

---

Kesimpulan: Poin Penting dari Metode Tincho

1. Pahami dulu, cari bug belakangan. Jangan terburu-buru mencari vulnerability sebelum benar-benar memahami kode.
2. Baca baris per baris dari atas ke bawah.
3. Gunakan komentar terstruktur (// Q:, // I:, dll) untuk menandai area yang perlu ditinjau atau ditanyakan ke klien.
4. Buat file catatan terpisah untuk mengorganisasi temuan dan pertanyaan.
5. Komunikasikan ketidakjelasan ke klien – jangan berasumsi.
6. Gunakan alat bantu seperti Headers dan Solidity Metrics untuk efisiensi.

"Clarity in our understanding of the codebase and the intended functionalities are a necessary part of performing a security review."