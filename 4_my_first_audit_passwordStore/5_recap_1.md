# Recap – Peran Security Researcher dan Persiapan Audit

## Pendahuluan

Materi ini adalah **recap** (ringkasan ulang) dari semua yang telah dipelajari di section sebelumnya. Fokus utamanya adalah memperkuat pemahaman tentang peran Anda sebagai **security researcher** dan pentingnya fase **scoping** yang matang sebelum memulai audit.

---

## 1. Peran Anda sebagai Security Researcher

Anda bukan hanya sekadar *coder* atau *developer* – Anda adalah **security researcher**, penjaga gerbang yang memastikan integritas smart contract.

> *"Smart contracts are the most adversarial environment on the planet, and we need to treat them as such."*

### Tanggung Jawab Utama:

| Tanggung Jawab | Penjelasan |
|----------------|------------|
| **Keamanan** | Menemukan kerentanan sebelum dieksploitasi. |
| **Dokumentasi** | Memastikan protokol memiliki dokumentasi yang jelas. |
| **Test suite** | Memastikan ada test suite yang memadai (coverage, fuzzing, dll). |
| **Edukasi** | Mengajari protokol tentang praktik terbaik keamanan dan persiapan audit. |

### Apa yang Tidak Cukup?

> *"A link to Etherscan is insufficient."*

Jika klien hanya memberikan link Etherscan tanpa:
- Repo GitHub dengan test suite
- Dokumentasi
- Scope yang jelas

**Maka protokol tersebut belum siap diaudit.** Tugas Anda adalah mendidik mereka.

> *"80% of the vulnerabilities out there are a product of business logic."*

Artinya, tanpa memahami **apa** yang protokol lakukan dan **bagaimana** cara kerjanya (melalui dokumentasi yang baik), Anda tidak akan bisa menemukan sebagian besar bug kritis.

---

## 2. Scoping: 7 Kategori Pertanyaan Wajib

Kita telah mempelajari [**Minimal Onboarding Questions**](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/minimal-onboarding-questions.md). Berikut ringkasan mengapa setiap bagian penting:

| Kategori | Mengapa Penting |
|----------|------------------|
| **About** | Ringkasan proyek. Semakin banyak dokumentasi, semakin baik. Anda perlu tahu tujuan dan invariant sistem. |
| **Stats** | Hitung nSLOC (dengan CLOC) untuk estimasi durasi audit. |
| **Setup** | Tools apa yang dibutuhkan? Bagaimana cara menjalankan test dan melihat coverage? Tanpa ini, Anda tidak bisa mereproduksi lingkungan. |
| **Scope** | Commit hash yang tepat dan daftar kontrak yang **in scope** (dan eksplisit yang out of scope). |
| **Compatibilities** | Versi Solidity, rantai target, token yang diintegrasikan – kerentanan bisa berbeda antar rantai. |
| **Roles** | Siapa saja aktor dalam sistem? Apa hak mereka? Apa yang boleh dan tidak boleh dilakukan? |
| **Known Issues** | Masalah yang sudah diketahui tim protokol dan tidak akan diperbaiki (atau sedang dalam proses). |

> Di masa depan, Anda akan menggunakan [**extensive onboarding form**](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/extensive-onboarding-questions.md) yang lebih lengkap. Namun untuk sekarang, minimal onboarding sudah cukup.

Anda juga dapat **menyesuaikan formulir ini** dengan kebutuhan Anda sendiri (terutama jika menjadi auditor independen atau membuka firma sendiri).

---

## 3. Pesan Penting untuk Perjalanan Audit

- Proses ini mungkin terasa **verbose** (panjang dan detail) dan **menakutkan**.
- Namun, **langkah-langkah ini sangat krusial**.
- Dengan melakukan scoping dengan benar, Anda akan **menghemat banyak waktu** di kemudian hari.
- Ini sangat penting terutama jika Anda bercita-cita menjadi **auditor independen** atau **mendirikan firma audit sendiri**.

---

## 4. Selanjutnya: Teknik Audit "The Tincho"

Setelah menguasai dasar-dasar scoping dan onboarding, kita akan mempelajari **teknik audit** yang digunakan oleh legenda **Tincho Abbate** – salah satu auditor terbaik di dunia.

> *"Keep sharp, in our next lesson we'll be going over 'The Tincho' – an auditing technique used by the legendary Tincho Abbate."*

---

## Kesimpulan

- Anda adalah **security researcher**, bukan sekadar programmer.
- **Link Etherscan saja tidak cukup** – protokol harus memiliki dokumentasi, test suite, dan scope yang jelas.
- **80% kerentanan berasal dari business logic** – pahami protokol secara mendalam.
- Gunakan **7 kategori pertanyaan onboarding** untuk memastikan klien siap diaudit.
- Proses scoping yang teliti akan **menghemat waktu** dan **meningkatkan kualitas audit**.

> Selamat telah mencapai titik ini! 🥳 Perjalanan masih panjang, tetapi fondasi yang kuat sudah Anda bangun.
