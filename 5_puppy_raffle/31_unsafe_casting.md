# Exploit: Unsafe Casting (Casting yang Tidak Aman)

## Pendahuluan

Selain masalah **Integer Overflow**, baris kode `totalFees = totalFees + uint64(fee)` di `PuppyRaffle` menyimpan kerentanan lain yang disebut **Unsafe Casting**. Ini terjadi ketika kita mencoba memasukkan nilai dari tipe data besar ke dalam tipe data yang lebih kecil tanpa pengecekan yang memadai.

---

## 1. Apa itu Unsafe Casting?

Unsafe casting terjadi saat kita melakukan konversi tipe data secara paksa. Jika nilai asli lebih besar dari kapasitas maksimum tipe data tujuan, Solidity tidak akan memberikan error (pada versi lama atau jika tidak dicek), melainkan akan memotong bit-bit angka tersebut hingga pas ke dalam wadah yang baru.

### Contoh Kasus:
Bayangkan kita memiliki nilai **20 ETH** (dalam Wei) yang bertipe `uint256` dan ingin kita ubah menjadi `uint64`.

1. **20 ETH** = `20,000,000,000,000,000,000` Wei.
2. **uint64.max** = `18,446,744,073,709,551,615`.

Karena 20 ETH > uint64.max, maka nilainya tidak akan muat.

---

## 2. Eksperimen dengan Chisel

Kita bisa melihat dampak nyata dari unsafe casting menggunakan tool **Chisel**:

```bash
➜ uint256 twentyEth = 20e18
➜ uint64 castedValue = uint64(twentyEth)
➜ castedValue
Type: uint64
└ Decimal: 1553255926290448384
```

### Analisis Hasil:
- **Nilai Asli**: 20,000,000,000,000,000,000 (20 ETH)
- **Hasil Casting**: 1,553,255,926,290,448,384 (~1.55 ETH)

Hanya dengan melakukan casting, kita secara tidak sengaja "membuang" sekitar **18.45 ETH**! Nilainya terpotong karena `uint64` hanya bisa mengambil bit-bit terakhir yang muat di kapasitasnya.

---

## 3. Dampak pada PuppyRaffle

Dalam konteks `PuppyRaffle`, variabel `fee` dihitung sebagai `uint256`. Namun, saat ditambahkan ke `totalFees`, ia dipaksa menjadi `uint64`:

```javascript
totalFees = totalFees + uint64(fee);
```

Jika dalam satu putaran raffle total tiket yang terjual menghasilkan biaya (`fee`) lebih dari **18.44 ETH**, maka nilai yang ditambahkan ke `totalFees` akan jauh lebih kecil dari yang seharusnya. Pemilik protokol akan kehilangan banyak dana karena catatan biayanya salah akibat pemotongan angka ini.

---

## 4. Perbedaan dengan Overflow

| Fitur | Integer Overflow | Unsafe Casting |
| :--- | :--- | :--- |
| **Penyebab** | Hasil operasi matematika melampaui batas. | Konversi tipe data besar ke kecil secara paksa. |
| **Kapan Terjadi** | Saat `a + b > max`. | Saat `uint64(uint256_besar)`. |
| **Hasil** | Nilai berputar kembali dari 0. | Nilai terpotong (truncated) sesuai kapasitas wadah baru. |

---

## 5. Cara Pencegahan

1. **Gunakan Tipe Data yang Konsisten**: Jika variabel menampung nilai ETH/Wei, selalu gunakan `uint256`.
2. **Library SafeCast**: Gunakan library `SafeCast` dari OpenZeppelin jika Anda benar-benar perlu melakukan konversi tipe data. Library ini akan melakukan `revert` jika nilai yang di-cast melampaui kapasitas tipe data tujuan.
3. **Hindari Downcasting**: Jangan mengubah tipe data besar ke kecil kecuali ada alasan teknis yang sangat kuat dan sudah tervalidasi.

---

## Kesimpulan

Unsafe casting adalah "pencuri tersembunyi" dalam Smart Contract. Sebagai auditor, setiap kali Anda melihat kode seperti `uint64(...)` atau `uint8(...)`, Anda harus segera memeriksa apakah nilai di dalamnya memiliki kemungkinan untuk melampaui batas tipe data tersebut.
