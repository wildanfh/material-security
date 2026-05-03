# Latihan Stateless & Stateful Fuzzing – Pengantar

## Pendekatan dalam Security Review

Dalam security review langsung, repositori yang diaudit bisa sangat besar (misalnya `TSwapPool.sol`). Meskipun manual review sangat penting, mengotomatiskan proses sebanyak mungkin dan memanfaatkan alat bantu dapat meningkatkan efisiensi secara signifikan.

Kita akan menggunakan **Foundry Fuzzer**, tetapi saya juga mendorong Anda untuk melihat [**Echidna**](https://github.com/crytic/echidna) yang mengintegrasikan `slither` untuk fuzzing yang lebih "cerdas".

## Setup Awal

Clone repositori [**sc-exploits-minimized**](https://github.com/Cyfrin/sc-exploits-minimized):

```bash
git clone https://github.com/Cyfrin/sc-exploits-minimized
cd sc-exploits-minimized
code .
```

> **Catatan:** Gunakan `git pull` untuk memastikan repositori lokal Anda terupdate.

## Tiga Jenis Pengujian untuk Invariant Break

Kita akan fokus pada tiga jenis pengujian:

1. **Stateless Fuzzing** (paling mudah)
2. **Open Stateful Fuzzing** (sedang)
3. **Closed Stateful Fuzzing with Handler** (paling sulit)

## Langkah Sebelum Memulai

1. **Baca dan pahami** informasi dalam [**invariant-break README**](https://github.com/Cyfrin/sc-exploits-minimized/blob/main/src/invariant-break/README.md). Memahami konsep di dalamnya akan memberi Anda keuntungan besar pada sesi berikutnya.

2. **Hapus folder `test/invariant-break`** dari repositori `sc-exploits-minimized` yang baru saja Anda clone. Menulis ulang kode ini dan memahami cara kerja tes adalah kunci untuk menguasai keterampilan ini.

## Setelah Latihan

Setelah kita selesai mempelajari konsep dan mempraktikkan metode pengujian ini, kita akan kembali ke **TSwap** dan menerapkan pembelajaran kita ke dalam basis kode tersebut.