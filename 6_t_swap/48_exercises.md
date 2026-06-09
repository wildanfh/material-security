# Ringkasan Materi: Latihan dan Tantangan (Section 5)

## Pendahuluan

Setelah menyelesaikan seluruh materi di Section 5 (audit TSwap, fuzzing, invariant, manual review), terdapat beberapa latihan tambahan untuk mengasah keterampilan. Latihan ini dirancang untuk membantu menulis test yang lebih baik dan menjadi security researcher yang lebih handal.

---

## Daftar Latihan

### 1. Write a fuzz test to find a bug in [this challenge](https://gist.github.com/tinchoabbate/67b195b95fe77a5b9e3c6cc4bf80b3f7)

**Deskripsi:**  
Terdapat sebuah tantangan (challenge) dari Tincho Abbate yang berisi kode smart contract yang sengaja memiliki bug. Tugas Anda adalah **menulis fuzz test** (stateless atau stateful) untuk menemukan bug tersebut.

**Tujuan:**  
- Menerapkan teknik fuzzing yang telah dipelajari (Foundry fuzz, invariant test, handler-based).
- Melatih kemampuan mengidentifikasi edge case dan pelanggaran invariant.
- Membiasakan diri dengan kode yang tidak sempurna (intentionally vulnerable).

**Langkah yang disarankan:**
1. Clone atau salin kode dari gist.
2. Buat proyek Foundry baru atau tambahkan ke repo latihan.
3. Definisikan invariant yang menurut Anda harus selalu benar.
4. Tulis fuzz test (stateful/stateless) untuk memverifikasi invariant.
5. Jalankan test hingga menemukan kegagalan.
6. Analisis akar masalah dan tulis laporan singkat.

> **Catatan:** Tantangan ini adalah kesempatan untuk berlatih tanpa tekanan, mirip dengan CTF (Capture The Flag).

### 2. Write a tweet thread about an [interesting finding from Solodit](https://solodit.xyz/)

**Deskripsi:**  
Buatlah utas tweet (thread) yang membahas **satu temuan menarik** dari laporan audit yang tersedia di [Solodit](https://solodit.xyz/). Tujuan utas ini adalah untuk berbagi pengetahuan dengan komunitas Web3 dan menunjukkan pemahaman Anda tentang kerentanan smart contract.

**Struktur utas yang disarankan:**
1. Tweet pertama: Perkenalan temuan (nama, severity, protokol).
2. Tweet kedua: Penjelasan singkat tentang akar masalah (root cause).
3. Tweet ketiga: Dampak (impact) dan contoh skenario eksploitasi.
4. Tweet keempat: Rekomendasi mitigasi atau pelajaran yang bisa diambil.
5. (Opsional) Sertakan tautan ke laporan asli di Solodit.

**Manfaat:**
- Membangun personal branding sebagai security researcher.
- Memperdalam pemahaman dengan menjelaskan ke publik.
- Terhubung dengan komunitas dan mendapatkan umpan balik.

---

## Section 5 NFT – Tantangan On-Chain

Terdapat dua kontrak (satu di Arbitrum, satu di Sepolia) yang merupakan **legit DeFi on-chain hack** – kemungkinan besar simulasi atau rekaman serangan DeFi nyata yang dapat dipelajari.

| Jaringan | Alamat Kontrak | Keterangan |
|----------|----------------|-------------|
| **Arbitrum** | [`0xbdaab68a462db80fb0052947bdadba7a87fcd0fb`](https://arbiscan.io/address/0xbdaab68a462db80fb0052947bdadba7a87fcd0fb#code) | A legit DeFi On-Chain Hack |
| **Sepolia** | [`0xdeb8d8efef7049e280af1d5fe3a380f3be93b648`](https://sepolia.etherscan.io/address/0xdeb8d8efef7049e280af1d5fe3a380f3be93b648) | A legit DeFi On-Chain Hack |

**Tugas (tantangan NFT):**  
Pelajari kontrak tersebut, analisis bagaimana serangan terjadi, dan coba pahami vektor serangan yang digunakan. Ini adalah latihan dunia nyata yang sangat berharga.

> **Tips:** Gunakan block explorer untuk melihat transaksi, trace, dan event. Cocokkan dengan kode kontrak.

---

## Pesan Penutup dan Motivasi

> *"If you've made it this far in the course and you understand what's going on, you have the skills to start getting paid as a security researcher, doing competitive audits, bug bounties, or even get hired!"*

**Kesimpulan:**
- Dengan pemahaman yang sudah diperoleh (fuzzing, invariant, manual review), Anda siap untuk mengikuti **competitive audits**, **bug bounties**, atau melamar pekerjaan sebagai security researcher.
- Namun, untuk menjadi salah satu yang terbaik di dunia dan benar-benar mengamankan Web3, teruslah belajar dan jangan berhenti di sini.

---

## Tindak Lanjut

- Kerjakan kedua latihan di atas.
- Pelajari NFT challenge.
- Bergabunglah dengan komunitas (Cyfrin Discord, CodeHawks, Immunefi) untuk mulai berpraktik secara nyata.
- Lanjutkan ke **Section 6** untuk materi yang lebih mendalam (formal verification, advanced attacks, dll).