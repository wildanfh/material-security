# Intro to TSwap – Invariants, Fuzzing, dan DeFi

## Peringatan Penting

Sebelum memulai, **JANGAN LIHAT KONTRAK** di repositori TSwap. [Repo TSwap](https://github.com/Cyfrin/5-t-swap-audit)

## Apa Itu TSwap?

TSwap adalah versi modifikasi dari **Uniswap** — sebuah bursa terdesentralisasi (DEX) yang menggunakan model **Automated Market Maker (AMM)**.

## Tantangan Unik di Section Ini

Kita akan mencoba menemukan bug **tanpa membaca kode** sama sekali.

> Ini bukan pendekatan normal, tujuannya untuk menunjukkan seberapa kuat tools dan metode yang akan dipelajari.

## Tools & Konsep Lanjutan yang Akan Dipelajari

| Tools / Konsep | Kegunaan |
|----------------|----------|
| **Fuzzing** | Menguji dengan input acak untuk menemukan edge case |
| **Invariant Testing** | Memastikan aturan bisnis protokol selalu terpenuhi |
| **Echidna** | Alat fuzzing untuk invariant testing |
| **Mutation Testing** | Menguji kualitas test suite dengan mengubah kode |

## Konsep DeFi yang Akan Didalami

- **AMM (Automated Market Maker)** – Mekanisme penentuan harga otomatis di DEX
- **Uniswap & DEX** – Protokol pertukaran terdesentralisasi
- **Curve.fi** – DEX khusus stablecoin dengan slippage rendah
- **Constant Product Formula** – Rumus `x * y = k` di balik AMM
- **Rebase Attacks** – Eksploitasi token dengan mekanisme rebasing
- **Reentrancy Attacks** – Serangan ulang masuk
- **Core Invariant Breaking** – Melanggar aturan fundamental protokol

## Pendekatan Manual Review

Waktu untuk manual review akan **sangat terbatas** karena fokus utama adalah belajar tools dan metode pengujian otomatis. Namun, konsep yang dipelajari sangat penting untuk audit DeFi sesungguhnya.