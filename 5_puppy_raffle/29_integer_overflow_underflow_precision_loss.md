# Exploit: Integer Overflow, Underflow, & Precision Loss

## Pendahuluan

Dalam Smart Contract, perhitungan matematika harus ditangani dengan sangat hati-hati. Sebelum Solidity versi 0.8.0, kesalahan perhitungan seperti **Overflow** dan **Underflow** sering terjadi tanpa peringatan. Selain itu, karena Solidity tidak mendukung angka desimal (*floating point*), masalah **Precision Loss** juga sering muncul.

---

## 1. Integer Overflow

**Overflow** terjadi ketika sebuah angka melampaui kapasitas maksimum tipe datanya, sehingga nilainya "berputar" kembali ke nol.

### Contoh di PuppyRaffle:
```solidity
totalFees = totalFees + uint64(fee);
```
Di sini, `totalFees` bertipe `uint64`. Jika nilai `totalFees` sudah mendekati batas maksimum `uint64` dan kita menambahkan `fee`, maka nilainya bisa berputar kembali ke angka kecil (misal: 0, 4, dsb). Ini akan menyebabkan pemilik protokol kehilangan catatan biaya yang seharusnya mereka terima.

### Analogi uint8 (Max 255):
- Jika `count = 255`
- `count + 1` akan menghasilkan **0** (bukan 256).
- `count + 5` akan menghasilkan **4**.

---

## 2. Integer Underflow

**Underflow** adalah kebalikan dari overflow. Ini terjadi ketika kita mengurangi angka melampaui batas minimum (biasanya 0), sehingga nilainya berputar ke angka maksimum.

### Contoh:
- Jika `count = 0` (tipe `uint8`)
- `count - 1` akan menghasilkan **255**.

---

## 3. Perubahan pada Solidity 0.8.0+

Sejak Solidity versi 0.8.0, kompiler secara otomatis menyisipkan pengecekan untuk overflow dan underflow. Jika terjadi kesalahan aritmatika, transaksi akan langsung **revert**.

Namun, pengembang masih bisa mengekspos kerentanan ini dengan menggunakan keyword `unchecked`:
```javascript
unchecked {
    count = count + 1; // Bisa overflow jika melampaui batas
}
```
*Catatan: PuppyRaffle menggunakan versi Solidity lama atau melakukan casting manual yang berisiko.*

---

## 4. Precision Loss (Kehilangan Presisi)

Solidity tidak mengenal angka koma (seperti 56.25). Semua hasil pembagian akan **dibulatkan ke bawah** (truncated).

### Contoh:
```javascript
uint256 money = 225;
uint256 users = 4;
uint256 share = money / users; // Hasilnya 56, bukan 56.25
```
Dalam protokol keuangan, kehilangan 0.25 mungkin terlihat kecil, namun jika dikalikan dengan volume transaksi yang besar, ini bisa mengakibatkan kerugian dana yang signifikan atau ketidakseimbangan akuntansi.

---

## 5. Mitigasi (Pencegahan)

1.  **Gunakan Solidity 0.8.0+**: Ini adalah cara termudah karena proteksi sudah bawaan dari kompiler.
2.  **SafeMath Library**: Untuk versi Solidity di bawah 0.8.0, selalu gunakan library `SafeMath` dari OpenZeppelin.
3.  **Urutan Operasi**: Untuk menghindari precision loss, selalu lakukan **perkalian sebelum pembagian**.
    - Buruk: `(a / b) * c`
    - Baik: `(a * c) / b`
4.  **Hati-hati dalam Casting**: Hindari mengubah `uint256` ke tipe yang lebih kecil (`uint64`, `uint8`) kecuali yakin nilainya tidak akan melampaui batas.

---

## Kesimpulan

Kesalahan aritmatika mungkin terlihat sepele, namun di dunia blockchain, ini bisa berarti kehilangan dana jutaan dolar. Sebagai auditor, selalu periksa setiap operasi matematika, terutama jika protokol menggunakan versi Solidity yang sudah usang.
