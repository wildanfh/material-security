# Fuzzing Recap – Rekap Lengkap Stateless, Stateful, dan Handler‑based Fuzzing

## Pendahuluan

Setelah mempelajari berbagai jenis fuzzing, kita akan merekap semua konsep penting yang telah dibahas. Fuzzing adalah teknik pengujian otomatis dengan memberikan data acak ke dalam sistem untuk menemukan kerentanan. Terdapat tiga pendekatan utama:

1. **Stateless Fuzzing** – data acak, state direset setiap run.
2. **Open Stateful Fuzzing** – state berlanjut antar run, fuzzer memanggil fungsi secara acak.
3. **Handler‑based Stateful Fuzzing** – menggunakan handler untuk membatasi input dan urutan fungsi.

---

## 1. Stateless Fuzzing

**Ciri utama:** Setiap kali fuzzer menjalankan satu putaran (*run*), *state* kontrak direset ke kondisi awal. Fuzzer hanya menguji fungsi individual dengan input acak.

**Konfigurasi di `foundry.toml`:**

```toml
[fuzz]
runs = 256      # Berapa kali run dijalankan
seed = '0x2'    # Nilai awal generator angka acak (bisa diulang)
```

**Kelebihan:**
- Mudah diimplementasikan.
- Sangat baik untuk menemukan edge case pada fungsi murni (pure) atau view.
- Memperluas cakupan tes di luar unit test biasa.

**Kekurangan:**
- **Tidak dapat menangkap kerentanan yang muncul karena perubahan state kontrak** (misal: urutan panggilan `changeValue` kemudian `doMoreMathAgain`).

---

## 2. Open Stateful Fuzzing

**Ciri utama:** State kontrak **dipertahankan** dari satu run ke run berikutnya. Fuzzer memanggil fungsi secara acak tanpa batasan, dan state terus berubah.

**Penerapan di Foundry:**
- Impor `StdInvariant` dan wariskan ke kontrak test.
- Tentukan `targetContract(address)`
- Fungsi test harus diawali `invariant_` atau `statefulFuzz_`

**Kelebihan:**
- Dapat menemukan bug yang bergantung pada urutan panggilan dan perubahan state.
- Lebih realistis karena meniru interaksi pengguna nyata.

**Kekurangan:**
- **Path explosion** – terlalu banyak jalur tidak valid (misal: memanggil fungsi dengan token tidak didukung → revert).
- Fuzzer menjadi tidak efisien karena banyak membuang waktu pada panggilan yang revert.
- Sulit menangani prasyarat seperti *approve* token.

---

## 3. Handler‑based Stateful Fuzzing (Closed Stateful Fuzzing)

**Ciri utama:** Menggunakan **handler contract** sebagai proxy (pembungkus) antara fuzzer dan kontrak target. Handler membatasi input dan urutan fungsi yang masuk akal.

**Komponen utama:**
- **Handler**: kontrak yang menyediakan fungsi‑fungsi spesifik (misal `depositYieldERC20`, `withdrawMockUSDC`).
- Di dalam handler, kita melakukan:
  - *Bound* nilai input agar tidak melebihi saldo.
  - *Approve* token sebelum transfer.
  - Memastikan hanya token yang didukung yang digunakan.
- Di invariant test, kita mengarahkan fuzzer hanya ke fungsi‑fungsi handler tersebut menggunakan `targetSelector`.

**Keuntungan:**
- Menghilangkan *revert* yang tidak relevan.
- Fuzzing menjadi lebih efisien dan terarah.
- Mampu menemukan kerentanan kompleks seperti **token fee‑on‑transfer** (Weird ERC20).
- Ideal untuk protokol DeFi dengan banyak ketergantungan.

---

## 4. Perbandingan Ketiga Metode

| Metode | State Reset? | Batasan Input | Efisiensi | Kemampuan temukan bug state‑dependent |
|--------|--------------|---------------|-----------|----------------------------------------|
| Stateless | Ya (setiap run) | Tidak ada | Cepat | ❌ Tidak |
| Open Stateful | Tidak | Tidak ada | Lambat (banyak revert) | ✅ Ya |
| Handler‑based Stateful | Tidak | Ya, melalui handler | Sangat efisien | ✅ Ya (bahkan untuk token aneh) |

---

## 5. Weird ERC20 dan Contoh Penemuan

**Weird ERC20** adalah token yang memiliki perilaku tidak standar, misalnya:
- **Fee‑on‑transfer** (memotong biaya setiap transfer).
- *Rebasing* (saldo berubah secara otomatis).
- *Deflationary/inflationary* token.

Dalam contoh `YieldERC20`, setiap 10 transaksi dikenakan biaya 10% yang dikirim ke `owner`. Kerentanan ini hanya dapat ditemukan dengan **handler‑based stateful fuzzing** karena:
- Diperlukan urutan deposit dan withdraw.
- Diperlukan token yang benar‑benar didukung.
- Tanpa handler, fuzzer akan kebanyakan revert karena token tidak dikenal atau tidak ada *approve*.

---

## 6. Penerapan Selanjutnya pada TSWAP

Setelah menguasai ketiga metode fuzzing, kita akan:
1. Menerapkannya pada protokol **TSwap** (fork Uniswap).
2. Menjalankan *stateful fuzzing* dengan handler untuk menguji invariant.
3. Melakukan *manual review* untuk melengkapi temuan otomatis.

> **Catatan:** Kombinasi fuzzing (otomatis) + manual review adalah praktik terbaik dalam security audit.

---

## 7. Istirahat dan Marathon Belajar

Materi ini padat dan menuntut pemahaman bertahap. Disarankan untuk:
- Beristirahat sejenak setelah mempelajari setiap metode.
- Mencoba menulis sendiri handler untuk kontrak sederhana.
- Menggunakan `-vvvv` untuk debugging fuzzing.

> *"Learning is a marathon, not a sprint; don't forget to hydrate, take breaks, and recharge yourself."*

---

## Kesimpulan

- **Stateless fuzzing** cocok untuk fungsi independen.
- **Open stateful fuzzing** diperlukan untuk state, tetapi rentan *path explosion*.
- **Handler‑based stateful fuzzing** adalah solusi paling ampuh untuk protokol kompleks.
- Dengan handler, kita dapat menemukan kerentanan tersembunyi seperti token fee‑on‑transfer.
- Selanjutnya, kita akan menerapkan semua ini ke TSwap dan melanjutkan dengan manual review.