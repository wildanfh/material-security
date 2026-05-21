# Manual Review dengan Slither pada TSwap

## Pendahuluan

Langkah pertama dalam manual review adalah menjalankan **static analyzers**. TSwap telah menyediakan `slither.config.json` dan perintah `make slither` untuk memudahkan. Hasil analisis Slither memberikan beberapa temuan, mulai dari yang berwarna merah (potensi masalah serius) hingga hijau (informational atau potensi masalah kecil).

---

## Red Detectors (Temuan Merah)

Slither melaporkan dua variabel yang tidak diinisialisasi:

```
PoolFactory.s_pools (src/PoolFactory.sol#27) is never initialized.
PoolFactory.s_tokens (src/PoolFactory.sol#28) is never initialized.
```

**Penjelasan:** Di `PoolFactory.sol`, terdapat dua mapping:

```solidity
mapping(address token => address pool) private s_pools;
mapping(address pool => address token) private s_tokens;
```

Mapping di Solidity **tidak perlu diinisialisasi** secara eksplisit; semua key secara default memetakan ke nilai default (address(0)). Namun, Slither tetap memberi peringatan karena tidak ada nilai awal yang diberikan. Dalam konteks ini, mapping hanya digunakan untuk pemeriksaan, sehingga **tidak berbahaya**. Tetap perlu dicatat sebagai peringatan informational.

---

## Green Detectors (Temuan Hijau)

### 1. Zero‑address check pada constructor PoolFactory

```bash
PoolFactory.constructor(address).wethToken lacks a zero-check on i_wethToken
```

Kode:

```solidity
constructor(address wethToken) {
    i_wethToken = wethToken;
}
```

**Masalah:** Tidak ada pemeriksaan apakah `wethToken != address(0)`. Jika `wethToken = address(0)`, protokol akan menggunakan alamat nol sebagai token WETH, yang dapat menyebabkan kegagalan fungsi.

**Temuan:** `Informational` – perlu ditambahkan `require(wethToken != address(0), "Invalid address")`.

**Tempat mencatat:** Tambahkan komentar `// @Audit: No Zero Address Check on wethToken`.

---

### 2. Potensi Reentrancy di `TSwapPool._swap`

```bash
Reentrancy in TSwapPool._swap(...):
External calls:
- outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000)
Event emitted after the call(s):
- Swap(...)
```

**Konteks:** Fungsi `_swap` mentransfer token ekstra sebagai insentif setiap 10 swap, kemudian memancarkan event `Swap`. Transfer dilakukan **sebelum** event dipancarkan, tetapi urutan ini tidak menimbulkan reentrancy karena:

- `safeTransfer` adalah panggilan eksternal ke token ERC20.
- Namun, tidak ada perubahan state penting setelah transfer (hanya event). Reentrancy biasanya berbahaya jika ada pembaruan state setelah panggilan eksternal. Di sini, state yang berubah (`swap_count`) sudah dimodifikasi sebelum transfer.
- Meskipun demikian, reentrancy tetap mungkin jika token yang digunakan memiliki callback yang memanggil kembali `_swap`. Perlu analisis lebih lanjut.

**Kesimpulan sementara:** Kemungkinan bukan masalah kritis, tetapi perlu dicatat untuk diteliti lebih dalam.

---

### 3. Penggunaan berbagai versi Solidity

Slither mendeteksi bahwa proyek menggunakan beragam versi Solidity (misal `^0.8.0`, `^0.7.6`, dll). Kontrak utama (`TSwapPool` dan `PoolFactory`) menggunakan versi `^0.8.20`, yang sudah modern. Variasi versi pada kontrak dependensi atau mock tidak berbahaya untuk audit ini.

**Kesimpulan:** Abaikan.

---

## Ringkasan Temuan dari Slither

| Temuan | Lokasi | Severity | Status |
|--------|--------|----------|--------|
| Mapping tidak diinisialisasi | `PoolFactory.s_pools`, `s_tokens` | Informational | Dapat diabaikan (perilaku default Solidity) |
| Zero‑address check | `PoolFactory.constructor` | Low / Informational | Perlu ditambahkan |
| Potensi reentrancy | `TSwapPool._swap` | Medium? | Perlu investigasi manual lebih lanjut |
| Versi Solidity beragam | Seluruh proyek | Informational | Diabaikan |

---

## Kesimpulan

Slither tidak menemukan banyak temuan juicy (kerentanan tinggi) pada TSwap, tetapi menjalankan static analyzer adalah **kebiasaan baik** yang harus dilakukan sebelum manual review mendalam. Temuan seperti zero‑address check dan potensi reentrancy perlu diverifikasi secara manual.

**Langkah selanjutnya:** Menjalankan **Aderyn** (static analyzer berbasis Rust) untuk mendapatkan perspektif tambahan.

> *"This was a quick review of what Slither brought to our process in TSwap. Not a lot of juicy findings from Slither, but this is a good habit to get into."*