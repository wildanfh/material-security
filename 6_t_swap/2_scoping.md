# Persiapan Audit TSwap

## Langkah Awal

Clone repositori dan masuk ke direktori:

```bash
git clone https://github.com/Cyfrin/5-t-swap-audit
cd 5-t-swap-audit
```

Verifikasi bahwa Anda berada di branch `main` dengan perintah `git branch`.

## Extensive Onboarding Form

TSwap menggunakan **Extensive Onboarding Form** (berbeda dengan minimal onboarding di PasswordStore). Formulir ini berisi informasi lebih detail yang sangat membantu auditor. Prinsipnya:

> *Information is **currency** in a security review* – semakin banyak informasi dan konteks, semakin mudah audit dilakukan.

### Poin-poin Penting dalam Formulir

- **Website, Dokumentasi, Kontak** – Dokumentasi sangat krusial untuk memahami konteks.
- **Commit hash** – Versi spesifik kode yang akan diaudit. Perintah yang digunakan:
  ```bash
  git checkout <COMMIT_HASH>
  ```
  Setelah checkout, gunakan `git diff <COMMIT_ID> <BRANCH>` untuk melihat perubahan antara branch dan commit hash.

### Statistik yang Dicatat

- **SLOC (Source Lines of Code)** – TSwap memiliki **374 SLOC**, hampir dua kali lipat dari Puppy Raffle. Ini mempengaruhi estimasi waktu audit.
- **Test coverage** – Dilaporkan **sangat rendah (abysmal)** – perlu diwaspadai.

### Pertanyaan Tambahan

Formulir juga mencakup:
- Pertanyaan tentang arsitektur, ketergantungan, dan risiko spesifik.
- **Known issues** (masalah yang sudah diketahui) – membantu auditor fokus pada area yang belum diperiksa.
- **Laporan audit sebelumnya** – sangat berharga untuk mempelajari temuan lama dan celah yang belum diperbaiki.

> Jika protokol tidak menyediakan dokumen ini, auditor **harus bertanya**. Tim pengembang memiliki konteks lebih banyak, manfaatkan mereka sebagai sumber informasi.

### Rekt Test & Post Deployment Plan

Bagian akhir formulir mengacu pada Rekt Test (12 pertanyaan kesiapan keamanan), termasuk rencana pasca-deploy. Protokol yang baik harus sudah memiliki jawaban untuk ini.

## Kesimpulan

- **Extensive Onboarding** memberikan pemahaman menyeluruh tentang protokol sebelum menyentuh kode.
- Komunikasi dengan protokol adalah kunci – jangan ragu bertanya.
- Data seperti SLOC, coverage, commit hash, dan known issues membantu auditor mengatur strategi dan prioritas.
- Setelah onboarding selesai, auditor siap melanjutkan ke fase berikutnya (reconnaissance dengan tools seperti fuzzing dan invariant testing).

> *"You need to be asking questions! The developers will always have more context than you, use them as a resource."*