# Liquidity Providers – Mengapa AMM Memiliki Biaya?

## Dari Mana Asal Liquidity Pool?

Pertanyaan mendasar: **"Dari mana asal kumpulan token (liquidity pool) ini?"**

Jawabannya: **Liquidity Providers (LP)** – para penyedia likuiditas.

LP menyetorkan token mereka ke dalam liquidity pool untuk membiayai perdagangan pengguna. Sebagai imbalan, mereka menerima **LP Token** (token penyedia likuiditas) yang proporsional dengan kontribusi mereka terhadap total pool.

![Liquidity Providers Diagram](https://updraft.cyfrin.io/security-section-5/6-liquidity-providers/liquidity-providers1.png)

## Mengapa Orang Mau Menyediakan Likuiditas? Apa Itu LP Token?

**Jawabannya: BIAYA (FEES)**

Setiap transaksi di DEX seperti TSwap atau Uniswap dikenakan biaya (contoh: 0,3%). Biaya ini biasanya **ditambahkan ke liquidity pool** yang bersangkutan.

![Liquidity Providers with Fees Diagram](https://updraft.cyfrin.io/security-section-5/6-liquidity-providers/liquidity-providers2.png)

## Bagaimana LP Mendapatkan Keuntungan?

LP Token yang dipegang oleh penyedia likuiditas memberi mereka hak atas **sejumlah proporsi token di dalam pool**.

- Ketika biaya transaksi terkumpul, total aset dalam pool bertambah.
- Karena LP memiliki proporsi tetap (melalui LP Token), **nilai klaim total mereka ikut naik**.
- Inilah sumber keuntungan bagi Liquidity Providers.

## Ringkasan Insentif

| Peran | Tindakan | Imbalan |
|-------|----------|---------|
| Trader | Membayar fee per transaksi | Mendapatkan token yang ditukar |
| Liquidity Provider | Menyetor token ke pool | Mendapat LP Token → klaim atas pool (termasuk fee yang terkumpul) |
| Protokol | Mengelola pool | Ekosistem likuid berkelanjutan |

## Kesimpulan

- LP menyediakan modal awal untuk liquidity pool.
- Fee transaksi ditambahkan ke pool, meningkatkan nilainya.
- LP Token mewakili kepemilikan proporsional, sehingga nilai LP ikut naik seiring akumulasi fee.
- Ini menciptakan insentif ekonomi yang membuat AMM dapat berjalan tanpa memerlukan order book.