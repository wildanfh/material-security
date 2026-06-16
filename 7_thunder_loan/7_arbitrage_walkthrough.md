# Ringkasan: Langkah-Langkah Arbitrase dengan Flash Loan

## Skenario Awal

Seorang pengguna biasa hanya memiliki **1 ETH** yang saat itu bernilai **$5**. Dengan modal sekecil itu, peluangnya untuk melakukan arbitrase antar bursa terdesentralisasi (DEX) sangat terbatas.

## Peran Flash Loan

Flash loan memungkinkan pengguna tersebut memanfaatkan dana besar sementara tanpa perlu memiliki modal sendiri yang besar.

## Proses Arbitrase Langkah demi Langkah

1. **Identifikasi Peluang**  
   Pengguna menemukan perbedaan harga (arbitrase) antara dua DEX.

2. **Eksekusi Flash Loan**  
   Pengguna meminjam dana dalam jumlah besar melalui protokol flash loan.

3. **Melakukan Arbitrase**  
   Dengan dana pinjaman tersebut, pengguna membeli aset di DEX yang harganya lebih murah dan langsung menjualnya di DEX yang harganya lebih tinggi.

4. **Meraup Keuntungan**  
   Selisih harga antar DEX menghasilkan keuntungan.

5. **Melunasi Pinjaman**  
   Pengguna mengembalikan dana pinjaman beserta biaya kecil.

> **Poin Kunci:** Langkah 2 hingga 5 terjadi dalam **satu transaksi tunggal**. Jika salah satu langkah gagal (misalnya tidak cukup untung untuk membayar pinjaman), seluruh transaksi akan dibatalkan (revert).

## Ilustrasi Alur

```
Pengguna (1 ETH, $5)
     │
     ├─► Melihat peluang arbitrase (DEX A harga murah, DEX B harga mahal)
     │
     ├─► Mengambil flash loan (dana besar)
     │
     ├─► Beli di DEX A → jual di DEX B → untung
     │
     ├─► Kembalikan pinjaman + fee
     │
     └─► Sisa keuntungan menjadi milik pengguna
```

## Kesimpulan

- Flash loan **menyamakan peluang** bagi pengguna kecil untuk bersaing dalam arbitrase DeFi.
- Proses yang rumit (pinjam → trading → bayar) bisa disederhanakan menjadi satu transaksi atomik.
- Dengan flash loan, modal awal yang kecil tidak lagi menjadi penghalang untuk memanfaatkan ketidakefisienan pasar.