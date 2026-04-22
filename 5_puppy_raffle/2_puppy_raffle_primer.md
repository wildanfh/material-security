# Puppy Raffle – Primer Sebelum Memulai

## Pendahuluan

Sebelum memulai proses audit Puppy Raffle, ada beberapa hal penting yang perlu diperhatikan.

---

## Dua Hal yang Harus Diingat

1. **Jangan lihat branch `audit-data`** dari repositori kursus [**Cyfrin/4-puppy-raffle-audit**](https://github.com/Cyfrin/4-puppy-raffle-audit).  
   Branch ini adalah **kunci jawaban** (answer key) yang berisi semua temuan dan laporan lengkap. Melihatnya sebelum waktunya akan menghambat proses belajar.

2. **Luangkan waktu untuk melakukan scoping sendiri** sebelum melanjutkan.  
   Coba terapkan proses yang sudah dipelajari saat audit PasswordStore:
   - Baca dokumentasi
   - Pahami kontrak dan fungsinya
   - Catat potensi kerentanan
   - Cari bug sebanyak mungkin

---

## Panduan Waktu

| Skenario | Tindakan |
|----------|----------|
| **Merasa tidak tahu harus mulai dari mana atau mentok** | Berhenti setelah 20-30 menit. Tidak masalah – kita akan mengerjakan bersama. |
| **Merasa "sedang on fire" dan menemukan beberapa bug** | Lanjutkan! Ulangi proses ini sampai nyaman melakukannya sendiri. |

> *"If you feel like you're cooking and you've found a few bugs - keep going. Repeating this process and becoming comfortable with doing it yourself is an important part of learning."*

---

## Mengapa Puppy Raffle?

Puppy Raffle adalah *codebase* yang **luar biasa** untuk mendapatkan pengalaman *security review* yang berharga. Ukurannya lebih besar dari PasswordStore, dengan lebih banyak bug (minimal 4 HIGH), sehingga cocok untuk melatih keterampilan manual review dan pengenalan static analysis.

---

## Kesimpulan

- **Jangan curang** dengan melihat kunci jawaban sebelumnya.
- **Coba sendiri dulu** – ini bagian penting dari proses belajar.
- **Jangan takut gagal** – jika mentok, kita akan lanjutkan bersama di video berikutnya.