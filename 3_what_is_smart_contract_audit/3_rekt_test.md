
# Audit Readiness dan Post-Deployment Planning

## Pendahuluan

Kesalahan umum: menganggap bahwa setelah melakukan audit, protokol siap untuk diluncurkan (deploy). Padahal, ada beberapa hal yang harus dipenuhi **sebelum** audit untuk memastikan protokol benar-benar siap.

Dua alat utama untuk menilai kesiapan audit:
1. [**nacentxyz simple-security-toolkit**](https://github.com/nascentxyz/simple-security-toolkit)
2. [**The Rekt Test**](https://blog.trailofbits.com/2023/08/14/can-you-pass-the-rekt-test/) oleh Trail of Bits

---

## The Rekt Test

Rekt Test adalah serangkaian pertanyaan untuk mengukur kesiapan protokol dalam hal keamanan. Jika protokol **tidak dapat menjawab** pertanyaan-pertanyaan ini, kemungkinan besar protokol tersebut **belum siap audit** (dan belum siap deploy).

### 12 Pertanyaan Rekt Test

| # | Pertanyaan |
|---|------------|
| 1 | Apakah semua peran dan hak istimewa (roles & privileges) dari setiap aktor telah didokumentasikan? |
| 2 | Apakah Anda memiliki dokumentasi untuk layanan eksternal, kontrak, dan oracle? |
| 3 | Apakah Anda memiliki rencana respons insiden yang tertulis dan telah diuji? |
| 4 | Apakah Anda mendokumentasikan cara terbaik untuk menyerang sistem Anda sendiri? |
| 5 | Apakah Anda melakukan verifikasi identitas dan pemeriksaan latar belakang untuk semua karyawan? |
| 6 | Apakah ada anggota tim yang memiliki tanggung jawab keamanan yang didefinisikan dalam perannya? |
| 7 | Apakah Anda mewajibkan hardware security keys (misal YubiKey) untuk sistem produksi? |
| 8 | Apakah sistem manajemen kunci Anda memerlukan banyak manusia dan langkah fisik? |
| 9 | Apakah Anda mendefinisikan invariant utama untuk sistem Anda dan mengujinya setiap kali commit? |
| 10 | Apakah Anda menggunakan alat otomatis terbaik untuk menemukan masalah keamanan dalam kode? |
| 11 | Apakah Anda menjalani audit eksternal dan mempertahankan program vulnerability disclosure atau bug bounty? |
| 12 | Apakah Anda telah mempertimbangkan dan memitigasi jalur penyalahgunaan oleh pengguna sistem? |

### Tindak Lanjut

- Jika protokol **tidak bisa menjawab** pertanyaan-pertanyaan ini, auditor harus memberi tahu bahwa protokol **belum siap deploy** (atau bahkan belum siap audit).
- Delegasikan tanggung jawab keamanan kepada seseorang di tim.
- Berikan proyek Anda rasa kepemilikan dan satu orang yang bertanggung jawab untuk menangani pelanggaran keamanan.

> *"Delegate responsibility to someone on your team for security - Give your project a sense of ownership and a point person to handle any security breaches."*

---

## Nascent Audit Readiness Checklist

[Repository simple-security-toolkit](https://github.com/nascentxyz/simple-security-toolkit) menyediakan checklist lain untuk menilai kesiapan audit.

Meskipun memberikan perspektif yang sedikit berbeda, tujuannya sama: membantu menentukan apakah protokol benar-benar siap untuk menjalani security review.

> Gunakan kedua alat ini (Rekt Test dan Nascent Checklist) secara komplementer.

---

## Post-Deployment Planning (Perencanaan Pasca-Deploy)

Memikirkan langkah-langkah **setelah deployment** sangat penting untuk keamanan holistik. Ini memastikan protokol siap menghadapi potensi eksploitasi dan mampu merespons dengan cepat.

### Komponen Post-Deployment:

| Komponen | Deskripsi |
|----------|-----------|
| **Bug Bounty Programs** | Program terbuka untuk memberi hadiah bagi peneliti keamanan yang menemukan kerentanan. |
| **Disaster Recovery Drills** | Latihan rutin untuk menguji rencana respons insiden dan pemulihan bencana. |
| **Monitoring** | Pemantauan on-chain dan off-chain untuk mendeteksi aktivitas mencurigakan secara real-time. |

### Mengapa Ini Penting?

- Audit bukanlah akhir dari perjalanan keamanan.
- Setelah deploy, protokol terus beroperasi di lingkungan adversarial.
- Memiliki rencana pasca-deploy yang matang **membingkai keamanan secara holistik** dan memastikan kesiapan menghadapi insiden.

---

## Kesimpulan

- **Audit readiness** tidak otomatis terpenuhi hanya karena akan melakukan audit.
- Gunakan **Rekt Test** dan **Nascent Checklist** untuk menilai kesiapan.
- Pastikan semua dokumentasi, peran, invariant, dan alat keamanan sudah siap.
- Rencanakan **post-deployment** termasuk bug bounty, disaster recovery, dan monitoring.
- Keamanan adalah proses berkelanjutan yang dimulai **sebelum** audit dan berlanjut **setelah** deploy.

> Protokol yang siap audit adalah protokol yang sudah memikirkan skenario terburuk, mendokumentasikannya, dan memiliki rencana respons yang teruji.
