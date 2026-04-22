# Solidity Visual Developer dan Tooling Tambahan

## Pendahuluan

Selain **Slither** dan **Aderyn**, terdapat beberapa alat tambahan yang sangat berguna dalam prosesaudit smart contract:

- **Solidity Metrics**
- **Solidity Visual Developer**
- **CLOC** (Count Lines of Code)

---

## Solidity Metrics

**Solidity Metrics** adalah ekstensi VS Code yang memberikan analisis komprehensif tentang codebase Solidity. Alat ini dapat diakses dengan klik kanan pada folder `src` lalu pilih `Solidity: Metrics`.

### Fitur Utama

#### 1. Inheritance Graph

Menampilkan hierarki pewarisan kontrak. Pada PuppyRaffle, terlihat bahwa kontrak `PuppyRaffle` mewarisi dari:
- `ERC721`
- `Ownable`

#### 2. Call Graph

Menampilkan hubungan pemanggilan antar fungsi. Berguna untuk memahami alur eksekusi dan dependensi antar fungsi dalam kontrak.

#### 3. Contract Summary

Menyediakan breakdown detail tentang:
- **Internal functions** – fungsi yang hanya dapat dipanggil dari dalam kontrak
- **External functions** – fungsi yang dapat dipanggil dari luar (atau `this`)
- **Payable functions** – fungsi yang menerima ether
- **State-modifying functions** – fungsi yang mengubah storage

Ringkasan ini sangat bernilai untuk memahami permukaan serangan (attack surface) sebuah kontrak.

---

## Solidity Visual Developer

**Solidity Visual Developer** adalah ekstensi VS Code yang dikembangkan oleh **tintinweb**. Alat ini sangat populer di kalangan developer dan auditor.

### Fitur Utama

1. **Inheritance Graph Interaktif** – dapat diklik untuk navigasi
2. **Syntax Highlighting** – berdasarkan tipe variabel (mapping, struct, dll)
3. **Pelaporan Serupa Solidity Metrics** – ringkasan statistik kontrak
4. **Security-focused features** – mendeteksi pola rentan tertentu

### Tampilan

Ekstensi ini memberikan visualisasi yang interaktif pada kode, memudahkan auditor memahami:
- Tipe data kompleks (mapping, nested types)
- Struktur inheritance
- Alur fungsi

### Instalasi

Pasang dari VS Code Marketplace:
```
Solidity Visual Developer
```

atau cari dengan nama `tintinweb.solidity-visual-auditor`.

---

## CLOC (Count Lines of Code)

Alat sederhana untuk menghitung jumlah baris kode. Berguna untuk:
- Mengetahui ukuran codebase
- Memperkirakan waktu audit
- Membuat Complexity Score

---

## Tooling Summary

| Tool | Fungsi | Bahasa/Platform |
|-----|-------|----------------|
| Slither | Static analysis - detector kerentanan | Python |
| Aderyn | Static analysis - laporan otomatis | Rust |
| Solidity Metrics | Analisis struktur & metrik | VS Code Extension |
| Solidity Visual Developer | Visualisasi & syntax highlighting | VS Code Extension |
| CLOC | Menghitung baris kode | CLI |

---

## Penggunaan dalam Audit

Alat-alat ini disarankan digunakan **sebelum** audit manual:

1. **SLOC** – estimasi effort
2. **Solidity Metrics** – struktur & call graph
3. **Aderyn** – laporan kerentanan umum
4. **Slither** – detector detail
5. **Manual Review** – analisis mendalam

---

## Kesimpulan

- **Solidity Metrics** memberikan insight invaluable tentang struktur kontrak (inheritance, call graph, contract summary).
- **Solidity Visual Developer** menawarkan visualisasi interaktif + syntax highlighting untuk keamanan.
- Kombinasikan semua tooling untuk audit yang efisien – jangan hanya mengandalkan satu alat.
-Setelah tooling, langkah berikutnya adalah **Recon** – mulai dari dokumentasi protokol.