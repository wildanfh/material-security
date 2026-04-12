# Manual Review: The Tincho Method

> Tincho adalah anggota [The Red Guild](https://theredguild.org/) dan mantan lead auditor OpenZeppelin — salah satu legenda di dunia Web3 security.

---

## Filosofi Dasar

> *"I don't have a super formal auditing process. I will just show you briefly some things that I do..."* — Tincho

Tidak ada satu metode baku yang sempurna. Yang penting: **sistematis, konsisten, dan jujur terhadap batas kemampuanmu sendiri.**

---

## Langkah 1: Baca Dokumentasi (Wajib)

**Ini bukan opsional.**

Sebelum menyentuh satu baris kode pun, baca dokumentasi protokol secara menyeluruh:
- Pelajari jargon dan terminologi yang akan kamu temui di kode
- Pahami apa yang *seharusnya* dilakukan protokol
- Bangun mental model tentang expected behavior

> Ingat: **80% bug adalah business logic bug** — kamu tidak bisa menemukan bug logic tanpa tahu apa yang seharusnya terjadi.

---

## Tools yang Digunakan Tincho

| Tool | Fungsi |
|---|---|
| **VS Codium** | Text editor berbasis VS Code dengan fokus privasi (tanpa telemetri) |
| **Foundry** | Framework untuk review dan testing — cepat dan powerful |
| **CLOC** | Hitung lines of code untuk estimasi kompleksitas per kontrak |
| **Solidity Metrics** | Tool Consensys untuk metrik mendalam tentang codebase Solidity |

### Cara Menggunakan CLOC + Solidity Metrics

Kombinasi dua tool ini memungkinkan kamu untuk:
1. **Urutkan kontrak berdasarkan kompleksitas** — dari yang paling sederhana ke paling kompleks
2. **Tandai kontrak yang sudah selesai di-review** — sistematis, tidak ada yang terlewat
3. **Mulai dari kontrak kecil** — bangun pemahaman bertahap sebelum masuk ke kontrak inti

```bash
# Contoh penggunaan CLOC
cloc ./src
```

---

## Proses Review

### Fase 1: Orientasi (Mindset Normal)
- Baca kode dari atas ke bawah
- Pahami alur data dan interaksi antar kontrak
- Catat pertanyaan dan asumsi

### Fase 2: Adversarial (Mindset Attacker)
Di titik tertentu, **ganti sudut pandang** menjadi penyerang:

> *"How can I break this?"*

Untuk setiap fungsi, tanyakan:
- **"Apakah ini akan bekerja untuk setiap jenis token?"**
  - Contoh: USDT adalah *weird ERC20* — `transferFrom` tidak return boolean. Kalau kode mengasumsikan return value, ini bisa jadi bug.
- **"Apakah access control modifier sudah diimplementasikan dengan benar?"**
- **"Apa yang terjadi jika input ekstrem atau tidak terduga diberikan?"**

---

## Sistem Pencatatan

Tincho merekomendasikan **dua lapis pencatatan**:

```
Layer 1: Anotasi langsung di kode
         → Komentar inline untuk temuan spesifik per baris

Layer 2: File catatan terpisah
         → Raw notes, ide, pertanyaan, pattern yang dicurigai
```

### Jangan Terlalu Dalam di Satu Bagian

> Ada risiko nyata: *rabbit hole* — menyelam terlalu dalam ke satu bagian dan kehilangan gambaran besar.

Strategi: **Pop back up secara berkala** — ingat bahwa kamu sedang mereview keseluruhan sistem, bukan hanya satu fungsi.

---

## Testing Selama Review

Manual review bukan satu-satunya yang kamu lakukan. Ketika kamu punya kecurigaan terhadap suatu fungsi:

1. **Tulis test untuk verifikasi kecurigaan** — jangan hanya berasumsi
2. **Gunakan fuzz test** untuk fungsi yang menerima input dari luar
3. Tincho sendiri menerapkan fuzz test pada assessment fungsi-fungsi dalam codebase ENS

> Kemampuan menulis test adalah senjata utama auditor — gunakan setiap saat kamu ragu.

---

## Komunikasi dengan Klien

Tincho menyebut komunikasi terbuka sebagai sesuatu yang **fundamental**.

> *"I would advise to keep the clients at hand. Ask questions, but also be detached enough."* — Tincho

**Prinsip: `Trust but validate`**

| Lakukan | Hindari |
|---|---|
| Tanya jika ada perilaku yang ambigu | Jangan asumsikan tanpa konfirmasi |
| Gunakan klien sebagai kolaborator | Jangan terlalu bergantung pada penjelasan klien |
| Validasi jawaban klien dengan kode | Jangan terima penjelasan tanpa verifikasi |

Klien selalu punya konteks lebih dalam tentang intended behavior — manfaatkan ini, tapi tetap independen dalam analisis.

---

## Time-Boxing: Batasi Waktu Review

> *"The thing is...I always get the feeling that you can be looking at a system forever."* — Tincho

Kode bisa di-review selamanya — selalu ada celah yang belum diperiksa. Solusinya:

- **Tetapkan batas waktu** untuk setiap bagian review
- **Jadilah sethorough mungkin dalam batas tersebut**
- Akui bahwa tidak ada audit yang 100% exhaustive

---

## Audit Report

Laporan akhir harus:
- **Jelas dan concise** dalam mendeskripsikan vulnerability yang ditemukan
- **Memberikan rekomendasi mitigasi** yang actionable
- Ditulis sedemikian rupa sehingga tim developer bisa langsung menggunakannya

### Mitigation Review
Setelah klien mengimplementasikan fix:
- Review implementasi mitigasi mereka
- Pastikan fix sudah benar
- Pastikan **tidak ada bug baru** yang masuk akibat perubahan

---

## Tanggung Jawab Auditor: Ketika Ada Bug yang Lolos

Selalu ada kemungkinan vulnerability yang terlewat. Respons yang tepat:

**Jangan hanya fokus mencari bug — jadilah mitra keamanan yang sesungguhnya:**
- Edukasi protokol tentang best practices
- Bantu mereka membangun budaya keamanan yang lebih kuat
- Persiapkan mereka untuk menghadapi ancaman secara holistik

**Soal tanggung jawab:**
> Auditor tidak menanggung seluruh beban ketika exploit terjadi. Tanggung jawab ini **dibagi bersama klien**.

Tapi ini bukan berarti kamu boleh bekerja sembarangan — reputasi dibangun dari kualitas kerja.

---

## Takeaway Utama dari Tincho

> *"Knowing that you're doing your best in that, knowing that you're putting your best effort every day, growing your skills, learning grows an intuition and experience in you."*

**Rangkuman mindset auditor yang baik:**

```
1. Selalu baca dokumentasi sebelum kode
2. Mulai dari kontrak sederhana, bangun ke yang kompleks  
3. Catat semua — di kode dan di file terpisah
4. Ganti ke adversarial mindset di pertengahan review
5. Tulis test untuk verifikasi kecurigaan
6. Komunikasi terbuka dengan klien — trust but validate
7. Time-box dirimu sendiri
8. Jadilah mitra keamanan, bukan hanya bug hunter
```
