# Menggunakan Compiler sebagai Alat Static Analysis

## Pendahuluan

Selain alat khusus seperti Slither dan Aderyn, **compiler Solidity itu sendiri** dapat berfungsi sebagai static analysis tool. Perintah `forge build` akan menampilkan peringatan (warning) tentang variabel yang tidak digunakan, parameter yang tidak terpakai, kode mati, dll. Peringatan ini sering diabaikan, tetapi dapat mengarah pada temuan serius.

---

## Kasus 1: Parameter `deadline` Tidak Digunakan di `deposit`

### Kode

```solidity
function deposit(
    uint256 wethToDeposit,
    uint256 minimumLiquidityTokensToMint,
    uint256 maximumPoolTokensToDeposit,
    uint64 deadline   // ← tidak pernah digunakan
) external ...
```

### Peringatan Compiler

Compiler memberi tahu bahwa parameter `deadline` tidak digunakan dalam fungsi.

### Analisis Severity

- **Impact:** High – pengguna mungkin mengandalkan deadline untuk membatalkan transaksi jika kondisi tidak terpenuhi. Namun karena deadline tidak diimplementasikan, deposit yang seharusnya gagal (karena melewati batas waktu) tetap akan berhasil. Ini adalah **pelanggaran terhadap ekspektasi pengguna** dan dapat menyebabkan kerugian (misal: deposit terjadi pada harga yang tidak menguntungkan).
- **Likelihood:** High – selalu terjadi karena deadline tidak pernah diperiksa.

**Severity: HIGH**

### Tindakan

Tandai dengan komentar audit:

```solidity
// @Audit-High - unused parameter may lead to unsupported trust in protocol functionality. 
// Deposits expected to fail may succeed.
uint64 deadline
```

---

## Kasus 2: Variabel `poolTokenReserves` Tidak Digunakan

### Kode

Di dalam fungsi `deposit` (atau fungsi lain), terdapat deklarasi:

```solidity
uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
```

Namun variabel ini tidak pernah digunakan lagi setelahnya.

### Analisis

- Ini adalah **dead code** – sisa dari implementasi lama yang sudah diganti dengan fungsi matematika terpisah.
- Tidak mempengaruhi keamanan, tetapi membuang gas (biaya eksekusi tidak perlu).

### Severity: Informational (atau Gas)

### Tindakan

```solidity
// @Audit-Informational - Line unused, can be removed to save gas
uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
```

---

## Kasus 3: Return Value `output` Tidak Digunakan di `swapExactInput`

### Kode

```solidity
function swapExactInput(...) public returns (uint256 output) {
    // ... logika swap
    // tidak pernah mengassign nilai ke `output`
}
```

### Analisis

- Fungsi dideklarasikan mengembalikan `uint256 output`, tetapi tidak pernah mengisi nilai tersebut.
- Dalam kode kontrak, nilai return ini tidak digunakan. Namun, jika protokol lain atau front-end mengharapkan nilai kembalian yang benar, mereka akan mendapatkan nilai default (0) yang menyesatkan.
- Dampak tergantung pada apakah ada pihak eksternal yang mengandalkan return value.

### Severity: Low (karena tidak digunakan secara internal, potensi masalah hanya jika diekspor ke eksternal)

### Tindakan

```solidity
// @Audit-Low - Return value not updated/used
returns (uint256 output)
```

---

## Ringkasan Temuan dari Compiler

| Temuan | Lokasi | Severity | Keterangan |
|--------|--------|----------|------------|
| Parameter `deadline` tidak digunakan | `deposit` | **High** | Pengguna mengira ada proteksi waktu, tetapi tidak ada |
| Variabel `poolTokenReserves` tidak digunakan | Fungsi terkait | Informational / Gas | Hanya membuang gas, hapus saja |
| Return value `output` tidak di-assign | `swapExactInput` | Low | Tidak digunakan internal, potensi masalah jika dipanggil eksternal |

---

## Kesimpulan

- **Compiler adalah static analysis tool yang paling mendasar dan sering diabaikan.** Jalankan `forge build` dan perhatikan setiap warning.
- Tidak semua warning tidak berbahaya – parameter `deadline` yang tidak digunakan bisa menjadi **High severity** karena merusak ekspektasi pengguna.
- Gunakan komentar `// @Audit-...` untuk mencatat temuan ini langsung dalam kode, dengan severity yang sesuai.

> *"Often an unused variable, or dead code, can be pretty innocuous. This situation is a little sneaky, however."*