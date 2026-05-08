# Debugging Fuzz Sequences – Menemukan Bug dengan Handler‑based Fuzzing

## Menjalankan Test dan Mengatasi Error Awal

Setelah menulis handler dan invariant test, kita jalankan:

```bash
forge test --mt statefulFuzz_testInvariantBreaksHandler
```

Hasil awal menunjukkan `assertion violated` tetapi tanpa informasi detail.  
Kita gunakan **`-vvvv`** (verbosity tingkat 4) untuk memperoleh keluaran yang lebih lengkap.

```bash
forge test --mt statefulFuzz_testInvariantBreaksHandler -vvvv
```

Dari keluaran yang lebih panjang, terlihat error `ERC20InsufficientAllowance`.  
Penyebabnya: kita sebelumnya hanya menetapkan *function selectors* dengan `targetSelector`, tetapi **belum menetapkan `targetContract` ke handler**.  
Perbaikan di `setUp()`:

```diff
  targetSelector(FuzzSelector({addr: address(handler), selectors: selectors}));
+ targetContract(address(handler));
```

---

## Menemukan Error Kedua: `ERC20InsufficientBalance`

Setelah menambahkan `targetContract`, test berjalan dan muncul error baru:

```
ERC20InsufficientBalance: insufficient balance for transfer
```

Error terjadi saat memanggil `withdrawToken` pada token `YieldERC20`.  
Mari kita periksa `YieldERC20.sol`.

---

## Akar Masalah: Token dengan Biaya (Fee‑on‑Transfer)

Di `YieldERC20.sol`, fungsi `_update` (yang dipanggil oleh `transfer` dan `transferFrom`) mengandung logika khusus:

- Setiap 10 transaksi, dikenakan **biaya 10%** yang dikirim ke `owner`.
- Sisa 90% dikirim ke penerima.

Cuplikan kode:

```solidity
if (count >= 10) {
    uint256 userAmount = value * USER_AMOUNT / PRECISION;   // 90%
    uint256 ownerAmount = value * FEE / PRECISION;          // 10%
    count = 0;
    super._update(from, to, userAmount);
    super._update(from, owner, ownerAmount);
} else {
    count++;
    super._update(from, to, value);
}
```

Dengan adanya biaya ini, ketika `HandlerStatefulFuzzCatches` mencoba menarik seluruh saldo `YieldERC20` milik pengguna menggunakan `withdrawToken`, kontrak akan menerima token lebih sedikit dari yang diharapkan karena sebagian dipotong untuk `owner`.  
Akibatnya, kontrak kekurangan saldo untuk memenuhi transfer penuh – muncullah `ERC20InsufficientBalance`.

---

## Klasifikasi Kerentanan

Token seperti `YieldERC20` termasuk dalam kategori **"Weird ERC20"** (ERC20 aneh) dengan perilaku tidak standar, salah satunya *fee‑on‑transfer* (token yang memotong biaya setiap transaksi).  
Protokol yang tidak mengantisipasi token semacam ini dapat mengalami kegagalan saat menarik dana.

---

## Kekuatan Handler‑based Stateful Fuzzing

Pengujian ini berhasil menemukan kerentanan yang tidak terdeteksi oleh:

- **Stateless fuzzing** — karena diperlukan urutan deposit/withdraw dan perubahan state kontrak.
- **Open stateful fuzzing** — karena tanpa handler, fuzzer tidak akan pernah memanggil `deposit` dengan `YieldERC20` yang benar (kebanyakan revert karena token tidak didukung).

Handler‑based fuzzing memungkinkan kita untuk:

1. **Membatasi input** hanya pada token yang didukung dan jumlah yang masuk akal.
2. **Mempertahankan state** antar panggilan fungsi.
3. **Menemukan interaksi berbahaya** seperti biaya tersembunyi pada token.

---

## Kesimpulan

- **Debugging fuzzing** membutuhkan verbosity tinggi (`-vvvv`) untuk melacak penyebab kegagalan.
- **Handler** tidak hanya membatasi jalur, tetapi juga memungkinkan kita menguji skenario realistis seperti deposit/withdraw token dengan biaya transaksi.
- **Token fee‑on‑transfer** adalah salah satu contoh "Weird ERC20" yang dapat merusak invariant protokol jika tidak ditangani dengan benar.
- **Handler‑based stateful fuzzing** terbukti menjadi alat yang sangat ampuh dalam mengungkap kerentanan tersembunyi pada protokol DeFi.

> *"This should clearly demonstrate how powerful a tool this method of fuzz testing can be."*