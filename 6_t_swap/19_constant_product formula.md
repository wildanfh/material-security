# Constant Product Formula – Penjelasan dan Turunan Matematis

## Pendahuluan

TSwap (fork dari Uniswap) menggunakan **constant product formula** sebagai invariant inti protokol. Invariant ini didefinisikan dalam dokumentasi sebagai:

> *"Perbandingan antara Token A dan WETH akan selalu tetap sama. Namun karena ada biaya (fee), invariant secara teknis meningkat."*

Rumus dasar yang harus selalu dipenuhi adalah:

**`x * y = k`**

- `x` = saldo token X dalam pool
- `y` = saldo token Y dalam pool
- `k` = konstanta (hasil kali x dan y)

## Invariant dengan Perubahan Saldo

Ketika terjadi pertukaran (swap), saldo token berubah:
- Token X bertambah sebesar `∆x` (tambah karena trader membeli Y)
- Token Y berkurang sebesar `∆y` (berkurang karena trader menjual Y)

Rumus invariant yang harus tetap berlaku adalah:

```
x * y = (x + ∆x) * (y − ∆y)
```

Dengan kata lain: **produk antara kedua saldo harus tetap konstan** (dalam kondisi tanpa fee).

## Turunan Matematis Menuju Bentuk yang Dapat Diuji

Dari persamaan di atas, kita dapat menyusun ulang untuk mendapatkan hubungan antara `∆x` dan `∆y` yang bergantung pada saldo saat ini.

Langkah-langkah aljabar:

1. **Ekspansi dengan FOIL** (First, Outer, Inner, Last):

```
x * y = (x + ∆x) * (y − ∆y)
x * y = x*y − x*∆y + ∆x*y − ∆x*∆y
```

2. **Sederhanakan** – kurangi kedua sisi dengan `x*y`:

```
0 = − x*∆y + ∆x*y − ∆x*∆y
```

3. **Kelompokkan suku yang mengandung `∆x`**:

```
x*∆y = ∆x*y − ∆x*∆y
x*∆y = ∆x * (y − ∆y)
```

4. **Tentukan `β` (beta)** sebagai rasio perubahan Y terhadap saldo Y awal:

```
β = ∆y / y   →   ∆y = β * y
```

Substitusi ke persamaan:

```
x * (β * y) = ∆x * (y − β * y)
x * y * β = ∆x * y * (1 − β)
```

5. **Sederhanakan** dengan membagi kedua sisi dengan `y` (asumsi y ≠ 0):

```
x * β = ∆x * (1 − β)
```

6. **Selesaikan untuk `∆x`**:

```
∆x = (β * x) / (1 − β)
```

Hasil ini sesuai dengan dokumentasi TSwap:

> **∆x = (β/(1-β)) * x**

Dengan cara yang sama (menggunakan `α = ∆x / x`), kita peroleh:

> **∆y = (α/(1+α)) * y**

## Invariant dengan Fee (Biaya)

Ketika ada biaya transaksi, `k` tidak lagi konstan – ia **meningkat** karena fee ditambahkan ke pool. Parameter fee dilambangkan dengan `ρ` (rho), dan `γ = 1 − ρ` (gamma). Maka persamaan invariant dengan fee menjadi:

```
∆x = (β/(1-β)) * (1/γ) * x
∆y = (αγ/(1+αγ)) * y
```

Dengan kata lain, fee menyebabkan `k` bertambah sedikit setiap kali transaksi, sehingga keuntungan terakumulasi untuk liquidity provider.

## Penerapan dalam Fuzzing TSwap

Untuk menguji invariant ini secara otomatis, kita akan menulis **stateful fuzz test suite** yang:

1. Mengamati perubahan saldo `∆x` dan `∆y` pada setiap swap.
2. Memverifikasi bahwa `∆x` dan `∆y` memenuhi hubungan matematis di atas (dengan toleransi kecil karena rounding).
3. Menggunakan handler untuk membatasi input hanya pada token yang didukung (WETH, USDC, dll).

Dengan demikian, kita dapat memastikan bahwa protokol selalu mengikuti constant product formula, baik dengan maupun tanpa fee.

## Kesimpulan

- **Invariant inti TSwap** adalah `x * y = k` (tanpa fee) atau peningkatan `k` yang dapat diprediksi (dengan fee).
- Turunan matematis menghasilkan rumus `∆x = (β/(1-β)) * x` yang mudah diuji secara numerik.
- Fuzzing stateful dengan handler adalah metode ideal untuk memverifikasi invariant ini dalam berbagai skenario swap.