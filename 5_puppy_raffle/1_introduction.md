# Ringkasan Materi: Puppy Raffle Audit (Section 4)

## Pendahuluan

Selamat datang di **Section 4: Puppy Raffle Audit**! Pada bagian ini, kita akan memperkuat keterampilan *manual review* dan memperkenalkan alat-alat canggih seperti **static analysis** untuk membantu mengamankan protokol. Kita juga akan mempelajari perbedaan antara **laporan audit privat** dan **submission audit kompetitif**, serta diperkenalkan dengan proses berkompetisi di **CodeHawks First Flight**.

---

## CodeHawks First Flights

**CodeHawks First Flights** adalah platform yang sangat baik bagi peneliti keamanan smart contract pemula. Platform ini menyediakan *codebase* yang relatif mudah dipahami, mirip dengan yang akan ditemui dalam kursus ini.

| Tingkat Keahlian | Manfaat |
|------------------|---------|
| **Pemula** | Kesempatan mendapatkan pengalaman audit langsung dan membangun keterampilan dalam lingkungan praktis. |
| **Berpengalaman** | Kesempatan untuk terlibat dalam komunitas dan mengasah keterampilan yang sudah dimiliki. |

> Dalam section ini, kita akan mempelajari cara mengirimkan temuan kompetitif yang luar biasa.

---

## Tooling yang Akan Digunakan

Kita akan menggunakan **static analysis** untuk membantu menemukan kerentanan. Dua alat utama:

| Alat | Bahasa | Fungsi |
|------|--------|--------|
| **[Slither](https://github.com/crytic/slither)** | Python | Static analysis untuk Solidity dan Vyper. Mendeteksi berbagai pola kerentanan umum. |
| **[Aderyn](https://github.com/Cyfrin/aderyn)** | Rust (dibuat oleh *Alex Roan*) | Menelusuri *Abstract Syntax Trees* (AST) untuk menyoroti kerentanan yang dicurigai. |

### Tujuan Pembelajaran di Section Ini

- **Mengenal** alat-alat static analysis pertama Anda.
- **Memahami** apa itu static analysis dan perannya dalam meningkatkan keamanan protokol.
- **Mendapatkan wawasan** tentang berbagai eksploitasi dalam *codebase* Puppy Raffle.
- **Mempelajari** cara menulis laporan untuk audit kompetitif dan membedakannya dengan audit privat.

---

## Puppy Raffle: Lebih Banyak Bug!

Berbeda dengan *codebase* PasswordStore yang kecil, **Puppy Raffle memiliki lebih banyak kode** dan konsekuensinya, **lebih banyak bug** yang harus ditemukan.

> **Target:** Setidaknya **EMPAT temuan HIGH** dalam repositori ini (dan mungkin beberapa yang bahkan tidak diperhitungkan oleh pembuat kursus 😋).

---

## Studi Kasus (Case Studies)

Saat kita menemukan kerentanan di *codebase* Puppy Raffle, kita akan menyelami **studi kasus dunia nyata** yang merinci saat-saat kerentanan serupa dieksploitasi di alam liar.

> Ini memberikan wawasan nyata tentang apa yang dipertaruhkan saat melakukan *security review* dan menanamkan bahwa upaya kita **sangat berarti**.

---

## Latihan (Exercises)

Di akhir section, akan ada **lebih banyak latihan** untuk memperluas pengetahuan dan menantang diri di luar ajaran kursus. Ini adalah kesempatan untuk:

- **Berkembang** (branch out)
- **Membangun jaringan** (network)
- **Mendapatkan pengalaman tambahan**

Termasuk berpartisipasi dalam **CodeHawks First Flight** atau **audit kompetitif**!

---

## Persiapan untuk Puppy Raffle

### Repositori

Repositori yang terkait dengan section ini: [**Cyfrin/4-puppy-raffle-audit**](https://github.com/Cyfrin/4-puppy-raffle-audit)

- README sudah cukup lengkap (asumsikan protokol telah melalui *onboarding* awal).
- Mirip dengan review sebelumnya, repositori ini memiliki **beberapa branch**, salah satunya adalah **`audit-data`**.

### Peringatan Penting

> **SANGAT DISARANKAN** untuk **tidak mengintip branch `audit-data` sampai akhir**!

Branch `audit-data` bertindak sebagai **kunci jawaban** (*answer key*) yang berisi semua kerentanan dan laporan lengkap.

### Mengapa Tidak Boleh Mengintip?

- **Membangun keterampilan** – Melewati proses dan menghargai setiap langkah adalah cara Anda membangun kemampuan.
- **Membangun keakraban** – Menemukan vektor serangan sendiri adalah cara Anda menjadi familiar dengan risiko.
- **Membangun kebiasaan** – Melewatkan langkah hanya akan merusak kemajuan Anda.

> *"Build the habits, do the work."*

---

## Ringkasan Poin Penting

| Topik | Poin Utama |
|-------|------------|
| **CodeHawks First Flights** | Platform untuk pemula dan berpengalaman; tempat berlatih audit kompetitif. |
| **Static Analysis Tools** | Slither (Python) dan Aderyn (Rust) – membantu menemukan kerentanan secara otomatis. |
| **Target Bug** | Minimal 4 temuan HIGH di Puppy Raffle. |
| **Studi Kasus** | Belajar dari eksploitasi nyata yang pernah terjadi. |
| **Latihan** | Kesempatan mengikuti kompetisi audit sungguhan. |
| **Branch `audit-data`** | Jangan diintip sampai selesai – itu adalah kunci jawaban. |

---

## Kesimpulan

Section 4 akan menjadi **tantangan yang lebih besar** dibandingkan PasswordStore. Kita akan menggunakan alat static analysis, mempelajari perbedaan audit privat vs kompetitif, dan berlatih dengan *codebase* yang lebih kompleks. Pastikan untuk **mengerjakan setiap langkah** dan tidak tergoda melihat jawaban sebelum waktunya.