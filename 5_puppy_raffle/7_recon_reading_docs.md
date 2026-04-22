# Recon: Reading Documentation

## Pendahuluan

**Recon** atau *reconnaissance* adalah langkah pertama dalam metodologi audit **Tincho** setelah scoping dan tooling. Tahap ini melibatkan pemahaman protokol melalui dokumentasi yang tersedia.

Langkah pertama adalah **membaca dokumentasi** – memahami apa yang seharusnya dilakukan protokol sebelum menganalisis kode.

---

## Sumber Dokumentasi

Dokumentasi protokol **Puppy Raffle** tersedia di repositori GitHub:
- Repository: [4-puppy-raffle-audit](https://github.com/Cyfrin/4-puppy-raffle-audit)
- File: README.md

---

## Isi Dokumentasi Puppy Raffle

### Deskripsi Protokol

Puppy Raffle adalah protokol yang memungkinkan pengguna berpartisipasi dalam raffle untuk memenangkan NFT anjing lucu.

### Fungsi yang Seharusnya Dilakukan

1. **Enter Raffle**
   - Panggil fungsi `enterRaffle(address[] participants)`
   - Parameter: array alamat yang ingin berpartisipasi
   - Dapat memasukkan diri sendiri beberapa kali, atau bersamaan dengan teman

2. **Anti-Duplikasi**
   - Alamat duplikat tidak diperbolehkan

3. **Refund**
   - Pengguna dapat mendapatkan refund ticket + value dengan memanggil fungsi `refund`

4. **Pengundian**
   - Setiap X detik, raffle dapat memilih pemenang
   - Pemenang akan dicetak (mint) puppy NFT secara acak

5. **Fee Distribution**
   - Owner protokol menentukan `feeAddress` untuk mengambil potongan
   - Sisa funds dikirim ke pemenang

---

## Membuat Catatan Audit

Selama proses audit, sangat disarankan untuk membuat file `notes.md` untuk mencatat:
- Hipotesis tentang fungsionalitas
- Temuan awal dari tooling
- Catatan penting lainnya

### Contoh Catatan

```markdown
## About

> The project allows users to enter a raffle to win a dog NFT.
```

Catatan ringkas ini akan memudahkan saat menyusun laporan akhir.

---

## Mengapa Documentation Penting?

1. **Memahami Intent** mengetahui apa yang direncanakan developer, berbeda dari implementasi
2. **Mengidentifikasi Missing Features** – fitur yang seharusnya ada tapi tidak ada di dokumentasi
3. **Menemukan Inconsistency** – perbedaan antara dokumentasi dan kode
4. **Membantu Risk Assessment** – memahami bisnis logic untuk menilai dampak finding

---


## Kesimpulan

- **Recon** dimulai dengan membaca dokumentasi protokol.
- Pahami **apa yang seharusnya dilakukan** sebelum menganalisis **bagaimana implementasinya**.
- Buat catatan (`notes.md`) untuk mencatat pemikiran selama audit.
- Dokumentasi adalah acuan untuk memvalidasi apakah kode sesuai dengan intent.