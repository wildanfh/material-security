# TSwap Reconnaissance Lanjutan

## Pemahaman DEX dan AMM

Dengan pemahaman yang lebih dalam tentang DEX dan AMM, kita melanjutkan pengintaian (recon) TSwap dengan membaca dokumentasi.

## Cara Kerja TSwap

Berdasarkan [README](https://github.com/Cyfrin/5-t-swap-audit/blob/main/README.md), protokol dimulai dengan **PoolFactory** yang membuat pool aset. Semua logika penanganan aset tersebut berada dalam kontrak `TSwapPool` yang dihasilkan.

> Setiap kontrak TSwapPool dapat dianggap sebagai pertukaran mandiri antara **tepat 2 aset**: ERC20 apa pun dan token WETH. Pool ini memungkinkan pengguna menukar secara tanpa izin antara ERC20 yang memiliki pool dengan WETH. Setelah cukup banyak pool dibuat, pengguna dapat dengan mudah "melompat" antar ERC20 yang didukung.

## Contoh Penggunaan TSwap

Dokumentasi memberikan contoh bagaimana TSwap seharusnya bekerja:

1. **User A memiliki 10 USDC**
2. Mereka ingin membeli DAI
3. Mereka menukar 10 USDC → WETH di pool **USDC/WETH**
4. Kemudian mereka menukar WETH → DAI di pool **DAI/WETH**

> Setiap pool adalah pasangan antara TOKEN X dan WETH.

## Diagram Protokol

Pada titik ini, sangat berguna untuk membuat **Diagram Protokol** yang menggambarkan bagaimana setiap bagian dari sistem bekerja sama. Diagram dapat membantu visualisasi alur data dan interaksi antar komponen.

Contoh diagram sederhana:

```
[User A] --10 USDC--> [USDC/WETH Pool] --WETH--> [DAI/WETH Pool] --DAI--> [User A]
```

Diagram ini sangat berharga untuk memahami keterkaitan antar bagian sistem yang terpisah-pisah.

## Liquidity Providers

Dokumentasi TSwap selanjutnya membahas tentang **penyedia likuiditas (liquidity providers)** – topik yang sudah kita bahas sebelumnya.

## Invariants (Prasyarat)

Bagian selanjutnya dari README protokol akan menjadi **integral** untuk bagaimana kita melanjutkan proses audit. Topik **invariants** akan dibahas di pelajaran berikutnya.

> Invariant adalah aturan atau properti sistem yang harus selalu benar, terlepas dari kondisi apa pun. Memahami invariant sangat penting untuk menemukan kerentanan logika bisnis.

## Ringkasan

- **PoolFactory** membuat pool pasangan ERC20 + WETH.
- **TSwapPool** menangani logika pertukaran untuk satu pasangan.
- **Contoh swap**: USDC → WETH → DAI melalui dua pool berbeda.
- **Diagram protokol** membantu visualisasi alur data.
- **Liquidity providers** menyediakan likuiditas.
- **Invariants** akan menjadi fokus utama berikutnya.

> Membuat diagram protokol adalah langkah yang sangat berharga dalam memahami bagaimana setiap bagian dari sistem bekerja sama.