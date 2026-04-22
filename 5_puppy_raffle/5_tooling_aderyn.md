# Aderyn: Static Analysis Tool Berbasis Rust

## Pendahuluan

**Aderyn** adalah alat static analysis berbasis Rust yang dikembangkan oleh **Alex Roan** dari tim Cyfrin. Alat ini berfungsi untuk mendeteksi kerentanan pada smart contract Solidity dengan cara menelusuri *Abstract Syntax Trees* (AST).

Aderyn menawarkan keunggulan berupa:
- Installasi dan running yang cepat
- Laporan yang terstruktur dan mudah dibaca
- Kompatibel dengan berbagai framework (Hardhat, Foundry, dll)

---

## Instalasi Aderyn

### Persyaratan: Rust

Sebelum menginstal Aderyn, pastikan **Rust** sudah terinstal di sistem Anda. Panduan instalasi tersedia di [rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).

### Instalasi dengan Cargo

Setelah Rust terinstal, gunakan perintah berikut untuk menginstal Aderyn:

```bash
cargo install aderyn
```

> **Catatan:** Jika Aderyn sudah terinstal, perintah ini akan memperbarui ke versi terbaru.

---

## Menjalankan Aderyn

Untuk menjalankan Aderyn pada proyek smart contract, gunakan perintah:

```bash
aderyn .
```

Perhatikan bahwa `.` menunjukkan root direktori proyek. Perintah ini akan:
1. Mengkompilasi kontrak secara otomatis
2. Menjalankan detector pada semua file `.sol`
3. Menampilkan laporan kerentanan di terminal
4. Menyimpan laporan ke file `report.md`

### Opsi Lain

Anda juga dapat menentukan path lain jika diperlukan:

```bash
aderyn [OPTIONS] <root>
```

---

## Contoh Laporan Aderyn

Setelah dijalankan, Aderyn menghasilkan laporan terstruktur yang mencakup:

### Ringkasan

| Key | Value |
|-----|-------|
| .sol Files | 1 |
| Total nSLOC | 143 |

### Kategori Issues

| Kategori | Jumlah |
|---------|--------|
| Critical | 0 |
| High | 0 |
| Medium | 1 |
| Low | 3 |
| NC | 4 |

### Jenis Issues yang Ditemukan

**Medium Issues:**
- M-1: Centralization Risk for trusted owners – owner memiliki hak istimewa yang perlu dipercaya untuk tidak melakukan update malicious atau drain funds.

**Low Issues:**
- L-1: `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
- L-2: Solidity pragma should be specific, not wide
- L-3: Conditional storage checks are not consistent

**NC (Non-Critical) Issues:**
- NC-1: Missing checks for `address(0)` when assigning values to address state variables
- NC-2: Functions not used internally could be marked external
- NC-3: Constants should be defined and used instead of literals
- NC-4: Event is missing `indexed` fields

---

## Perbandingan dengan Slither

| Aspek | Slither | Aderyn |
|-------|---------|--------|
| Bahasa | Python | Rust |
| Developer | Trail of Bits | Cyfrin (Alex Roan) |
| Kode Warna | Ya | Tidak |
| Format Laporan | Terminal | Markdown (report.md) |

---

## Kesimpulan

- **Aderyn** adalah alternatif powerful berbasis Rust untuk static analysis smart contract.
- Instalasi memerlukan Rust terlebih dahulu, lalu `cargo install aderyn`.
- Jalankan dengan `aderyn .` untuk analisis penuh.
- Laporan tersimpan di `report.md` – struktur bersih dan siap untuk audit report.
- Seperti Slither, Aderyn **tidak menggantikan review manual**, hanya mempercepat proses identifikasi pola kerentanan umum.
- Disarankan menjalankan Aderyn (dan Slither) **sebelum** audit manual.

> *"Fast, and efficient, Aderyn offers a swift vulnerability report of your smart contracts which is almost ready to be presented."*