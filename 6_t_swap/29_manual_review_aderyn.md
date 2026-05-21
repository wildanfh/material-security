# Manual Review dengan Aderyn pada TSwap

## Persiapan

Pastikan Aderyn terbaru:

```bash
cargo install aderyn
```

Jalankan perintah:

```bash
aderyn .
```

Aderyn akan menghasilkan laporan dalam file `report.md`. Perlu diingat bahwa Aderyn terus berkembang, sehingga output yang Anda terima mungkin berbeda dari yang ditampilkan di materi.

---

## Ringkasan Laporan Aderyn

- **Jumlah file .sol:** 2
- **Total nSLOC:** 262
  - `PoolFactory.sol`: 35 baris
  - `TSwapPool.sol`: 227 baris
- **Tingkat keparahan:** Tidak ada High, 3 Low

---

## Low Issues yang Ditemukan

### L-1: Fungsi `public` yang tidak dipakai internal sebaiknya `external`

**Lokasi:** `TSwapPool.sol` baris 247 – fungsi `swapExactInput`

**Penjelasan:** Fungsi yang tidak dipanggil dari dalam kontrak dapat diubah dari `public` menjadi `external` untuk menghemat gas.

**Tindakan:** Tandai dengan komentar `// @Audit - functions not used internally can be marked external to save gas.`

---

### L-2: Gunakan konstanta, bukan literal angka

**Lokasi:** Beberapa baris di `TSwapPool.sol` (227, 244, 369, 375)

Contoh:

- `inputAmount * 997`
- Perkalian dengan `10000` dan `997`
- Literal `1e18`

**Penjelasan:** Angka literal ("magic numbers") mengurangi keterbacaan dan menyulitkan pemeliharaan. Sebaiknya didefinisikan sebagai konstanta dengan nama deskriptif.

**Tindakan:** Tandai dengan komentar `// @Audit - Use constants instead of magic numbers/literals`

---

### L-3: Event tidak menggunakan `indexed` fields

**Lokasi:** 
- `PoolFactory.sol` baris 35 – `event PoolCreated`
- `TSwapPool.sol` baris 43-45 – event `LiquidityAdded`, `LiquidityRemoved`, `Swap`

**Penjelasan:** Field event yang diberi `indexed` memudahkan pencarian oleh alat off-chain. Namun, setiap field `indexed` memakan gas ekstra. Pedoman umum: jika jumlah field ≤ 3, sebaiknya semua di-`indexed`. Jika lebih dari 3, pilih tiga yang paling penting.

**Catatan:** Ada perdebatan – beberapa auditor lebih suka lebih sedikit `indexed`. Terserah Anda apakah akan memasukkannya ke dalam laporan.

**Tindakan:** Tandai dengan komentar `// @Audit - Event should be indexed if there are more than 3 parameters`

---

## Kesimpulan

Aderyn berhasil menemukan beberapa temuan tingkat rendah/informational yang berguna untuk meningkatkan kualitas kode dan efisiensi gas. Alat seperti Aderyn dan Slither sangat membantu mengotomatisasi sebagian temuan, tetapi **tidak menggantikan manual review baris per baris**.

> *"Amazing! Tools like Aderyn and Slither really assist in helping us automate some of these findings. Now that we're done with them, we can move on to some actual, line-by-line, manual review."*