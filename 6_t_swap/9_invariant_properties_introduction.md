# Invariant / Properties – Pengantar

## Definisi Invariant

> **Invariant** adalah properti atau kondisi dari suatu sistem yang **harus selalu benar** (always hold true), terlepas dari keadaan atau tindakan apa pun yang terjadi di dalam sistem.

## Konteks dalam TSwap

Setelah memahami banyak konteks tentang apa yang seharusnya dilakukan TSwap dan bagaimana cara kerjanya, bagian penting dari README TSwap adalah **core invariant** (invariant inti) protokol.

Kebanyakan protokol memiliki beberapa jenis invariant – bahkan sesuatu sesederhana ERC20 pun memiliki kondisi yang harus selalu benar.

> Sumber daya bermanfaat: [Repositori properti oleh Trail of Bits](https://github.com/crytic/properties) – berisi katalog contoh invariant/properti untuk berbagai token dan operasi Ethereum.

## Core Invariant TSwap

Dokumentasi TSwap menyatakan:

> *"Sistem kami bekerja karena rasio antara Token A dan WETH akan selalu tetap sama. Ya, untuk sebagian besar. Karena kami menambahkan biaya, invariant kami secara teknis meningkat."*

### Rumus Dasar

**`x * y = k`**

- `x` = Saldo Token X
- `y` = Saldo Token Y
- `k` = Rasio konstan antara X dan Y

### Turunan Rumus

Dari rumus dasar, kita dapatkan hubungan perubahan:

```
x * y = (x + ∆x) * (y − ∆y)
```

- `∆x` = Perubahan saldo token X
- `∆y` = Perubahan saldo token Y

Dengan definisi:
- `β = ∆y / y` (proporsi perubahan Y)
- `α = ∆x / x` (proporsi perubahan X)

### Invariant Tanpa Biaya (No Fees)

```
∆x = (β/(1-β)) * x
∆y = (α/(1+α)) * y
```

### Invariant Dengan Biaya (With Fees)

- `ρ` = biaya (antara 0 dan 1, persentase)
- `γ = (1 - ρ)` (dibaca gamma)

```
∆x = (β/(1-β)) * (1/γ) * x
∆y = (αγ/(1+αγ)) * y
```

> **Pernyataan protokol:** "Protokol kami harus selalu mengikuti invariant ini agar proses swap berjalan dengan benar!"

## Catatan Penting

**Ini bukanlah norma!** Biasanya, kita hanya diberikan dokumentasi, dan tugas kitalah yang menentukan, melalui proses *reconnaissance*, properti mana dari suatu protokol yang memenuhi syarat sebagai invariant.

## Selanjutnya

Kita telah menyentuh invariant di masa lalu. Pada pelajaran berikutnya, kita akan menonton **refresher** yang membahas tentang **fuzz tests** dan bagaimana hubungannya dengan invariant.

> Memahami invariant adalah kunci untuk menguji kebenaran sistem secara otomatis dan menemukan kerentanan logika bisnis yang tidak terdeteksi oleh static analysis biasa.