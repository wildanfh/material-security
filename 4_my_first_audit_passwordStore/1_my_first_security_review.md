## Audit Pertama: PasswordStore

Untuk audit pertama, kita akan meninjau protokol **PasswordStore**. Skenario ini dirancang seperti situasi nyata di mana Anda adalah seorang security researcher yang baru menerima permintaan audit dari sebuah protokol (misalnya dari firma seperti Cyfrin).

> Pendekatan ini **immersive dan experiential** – Anda akan benar-benar merasakan peran sebagai auditor.

Dalam pelajaran selanjutnya, kita juga akan mempelajari proses **submission findings** dalam skenario kompetitif seperti **CodeHawks**.

---

## Target Akhir (End Goal)

Berikut adalah tautan ke repositori PasswordStore pada berbagai fase audit. Lihat secara khusus pada **Security Review Final**:

| Fase | Deskripsi | Tautan |
|------|-----------|--------|
| **Security Review CodeV1** | Kode awal yang akan diaudit (deployed di Sepolia). | [Etherscan](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b) |
| **Security Review CodeV2** | Setelah perbaikan awal. | [GitHub](https://github.com/Cyfrin/3-passwordstore-audit) |
| **Security Review CodeV3** | Setelah proses onboard. | [GitHub (branch onboarded)](https://github.com/Cyfrin/3-passwordstore-audit/tree/onboarded) |
| **Security Review Final** | Hasil akhir audit, termasuk laporan PDF profesional. | [GitHub (branch audit-data)](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data) |

> Folder `audit-data` berisi semua yang akan Anda hasilkan di bagian ini, termasuk **laporan audit PDF profesional**.

---

## Mengingat Kembali Tiga Fase Audit

Setiap audit atau security review mengikuti tiga fase utama:

| Fase | Sub-fase |
|------|-----------|
| **1. Initial Review** | Scoping → Reconnaissance → Vulnerability Detection → Reporting |
| **2. Protocol Fixes** | Protocol fixes issues → retests dan menambahkan test |
| **3. Mitigation Review** | Reconnaissance → Vulnerability Detection → Reporting |

### Fokus Kursus Ini

Dalam kursus ini, kita akan fokus pada **bagaimana melakukan initial review** secara efektif. Kita mulai dari yang kecil – codebase kurang dari 20 baris – tetapi ini baru awal.

> *"You are the security researcher. Often times what may be clear or obvious to you, isn't to a protocol. Your expertise is valuable."*

---

## Apa yang Perlu Disiapkan

- **Mindset:** Anda adalah auditor profesional.
- **Ekspektasi:** Tidak ada audit yang sempurna, tetapi setiap temuan berharga.
- **Target:** Menghasilkan laporan audit yang profesional pada akhir bagian ini.
