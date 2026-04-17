# Menggunakan AI untuk Memoles Laporan Temuan

## Pendahuluan

AI (seperti ChatGPT) dapat menjadi alat yang sangat membantu dalam menulis laporan audit, terutama untuk:
- Memeriksa tata bahasa dan ejaan.
- Memperbaiki format markdown.
- Menyusun ulang kalimat agar lebih jelas dan profesional.

Namun, **AI tidak boleh diandalkan sepenuhnya** karena:
- AI dapat **berhalusinasi** (menghasilkan informasi yang salah).
- AI bisa membuat kesalahan logika atau teknis.
- AI tidak memahami konteks keamanan secara mendalam.

> Gunakan AI sebagai **pemeriksa kewarasan (sanity check)** , bukan sebagai pengganti penilaian manusia.

---

## Prompting yang Tepat

Kunci mendapatkan hasil yang baik dari AI adalah memberikan **prompt yang jelas dan terstruktur**.

### Contoh Prompt yang Efektif

```text
The following is a markdown write-up of a finding in a smart contract codebase, can you help me make sure it is grammatically correct and formatted nicely?

---
[PASTE REPORT DI SINI]
---
```

### Elemen Prompt yang Baik

| Elemen | Penjelasan |
|--------|------------|
| **Konteks** | Beri tahu AI apa yang sedang dikerjakan (misal: laporan temuan smart contract). |
| **Instruksi spesifik** | Sebutkan apa yang diinginkan (proofread grammar, format markdown). |
| **Pemisah yang jelas** | Gunakan `---` atau pembatas lain antara instruksi dan data. |
| **Data mentah** | Tempelkan laporan yang ingin diperiksa. |

---

## Hal yang Perlu Diperhatikan

### ✅ Lakukan
- Gunakan AI untuk **memeriksa ulang** laporan Anda.
- Manfaatkan AI untuk **memperbaiki tata bahasa** dan **format**.
- Baca kembali hasil saran AI sebelum menggunakannya.

### ❌ Jangan Lakukan
- Jangan **menyalin begitu saja** saran AI tanpa verifikasi.
- Jangan mengandalkan AI untuk **konten teknis** (misal: kode PoC, severity).
- Jangan percaya bahwa AI selalu benar – **AI bisa salah**.

> *"Don't simply copy over its suggested implementation, this is very risky."*

---

## Kesimpulan

- AI adalah alat yang **sangat berguna** untuk memoles laporan, terutama dari segi bahasa dan format.
- **Prompt yang baik** adalah kunci mendapatkan hasil yang relevan.
- **Selalu verifikasi** saran AI – jangan mengandalkannya secara membabi buta.
- Gunakan AI sebagai **asisten**, bukan pengambil keputusan akhir.

> *"Remember to use these tools to your advantage when drafting complex technical reports. But always remember to cross-check their work to ensure it is free from errors."*