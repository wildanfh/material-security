# TSwap Primer

## Langkah Setelah Onboarding

Setelah protokol selesai di-onboard, langkah selanjutnya adalah:

1. **Baca dokumentasi** – untuk memahami protokol dan cara kerjanya.
2. **Dapatkan konteks** – karena TSwap adalah fork dari Uniswap, kita akan belajar banyak tentang DeFi, AMM (Automated Market Maker), dan DEX (Decentralized Exchange).

## Tujuan Unik di Section Ini

Kita akan berusaha **memahami invariant** protokol dengan sangat baik sehingga mampu **membuat test suite sendiri tanpa pernah melihat kode kontrak**.

> Ini pendekatan yang tidak biasa, tujuannya untuk menunjukkan kekuatan tools dan metode fuzzing/invariant testing.

## Yang Tidak Boleh Dilakukan

**Jangan lihat kontrak dulu** – fokus pada dokumentasi dan pemahaman invariant.

## Yang Akan Dipelajari

- Prinsip kerja AMM dan DEX (terinspirasi dari Uniswap)
- Invariant utama dalam protokol seperti `x * y = k`
- Cara membuat test yang memverifikasi invariant tanpa mengetahui implementasi kode

> *"Context is king."*