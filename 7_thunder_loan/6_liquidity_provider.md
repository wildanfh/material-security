# Ringkasan: Liquidity Providers pada Flash Loan

## Pertanyaan Dasar
Dari mana asal dana yang digunakan untuk pinjaman kilat (*flash loan*)?

## Jawaban: Liquidity Providers (Penyedia Likuiditas)

Sama seperti pada protokol `TSwap`, di mana penyedia likuiditas menyetor dana ke dalam *pool* untuk mendapatkan imbalan berupa biaya (*fees*), konsep yang sama juga berlaku pada protokol *flash loan*.

## Mekanisme Kerja

1. **Deposit Dana**  
   Seorang *Liquidity Provider* menyetorkan sejumlah dana ke dalam protokol *flash loan*.

2. **Menerima Token LP**  
   Sebagai bukti kontribusi, penyedia likuiditas akan menerima token LP (*Liquidity Provider Token*) yang merepresentasikan bagiannya dari total *pool*.

3. **Akrual Biaya**  
   Setiap kali pengguna melakukan *flash loan*, protokol akan membebankan biaya (biasanya kecil, misalnya 0.09%). Biaya ini terakumulasi di dalam *pool*.

4. **Peningkatan Nilai**  
   Karena biaya terus bertambah, nilai dari setiap token LP akan meningkat seiring waktu. Dengan kata lain, porsi kepemilikan penyedia likuiditas di dalam *pool* menjadi lebih berharga dibandingkan saat awal menyetor.

## Ilustrasi

```
[Liquidity Provider] --- deposit dana --> [Flash Loan Protocol]
                                            |
                                            ├── menerbitkan LP Token
                                            │
[Pengguna Flash Loan] --- meminjam + biaya ---> [Protocol]
                                            │
                                            └── biaya ditambahkan ke pool
                                               → nilai LP Token meningkat
```

## Kesimpulan

- **Liquidity Providers** adalah sumber utama likuiditas untuk *flash loan*.
- Mereka diinsentif melalui akumulasi biaya yang meningkatkan nilai kepemilikan mereka.
- Model ini memungkinkan *flash loan* berjalan tanpa memerlukan modal besar dari protokol itu sendiri.