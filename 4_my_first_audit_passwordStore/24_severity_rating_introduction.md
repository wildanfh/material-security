# Cara Menentukan Tingkat Keparahan (Severity) Suatu Temuan

## Pendahuluan

Menentukan tingkat keparahan (severity) dari sebuah kerentanan adalah langkah penting dalam proses audit. Severity membantu tim pengembang memprioritaskan perbaikan dan membantu auditor menentukan bobot temuan mereka. Dalam materi ini, kita akan menggunakan panduan dari [**Dokumentasi CodeHawks**](https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity) sebagai acuan.

Tingkat keparahan dibagi menjadi tiga kategori utama: **High**, **Medium**, dan **Low**. Beberapa peneliti juga menambahkan **Critical** untuk situasi yang sangat ekstrem.

---

## Dua Dimensi Penentu Severity

Severity ditentukan berdasarkan dua faktor utama:

1.  **Impact (Dampak)**: Seberapa parah kerusakan yang bisa terjadi jika kerentanan dieksploitasi.
2.  **Likelihood (Kemungkinan)**: Seberapa besar kemungkinan kerentanan tersebut benar-benar dieksploitasi.

Kombinasi dari kedua faktor ini menentukan apakah suatu temuan tergolong High, Medium, atau Low.

---

## 1. Menilai Impact (Dampak)

| Tingkat Impact | Kriteria |
|---|---|
| **High Impact** | Dana secara langsung atau hampir langsung berisiko. Terjadi gangguan parah pada fungsionalitas atau ketersediaan protokol. |
| **Medium Impact** | Dana berisiko secara tidak langsung. Ada beberapa tingkat gangguan pada fungsionalitas protokol. |
| **Low Impact** | Dana tidak berisiko. Namun, suatu fungsi mungkin tidak berjalan dengan benar, atau state tidak ditangani dengan tepat. |

> **Sudut Pandang Alternatif:** Coba bayangkan dari sisi pengguna: *seberapa kesal para pengguna jika serangan ini benar-benar terjadi?* Hal ini dapat membantu Anda merasakan dampaknya.

---

## 2. Menilai Likelihood (Kemungkinan)

| Tingkat Likelihood | Kriteria & Contoh |
|---|---|
| **High Likelihood** | Sangat mungkin terjadi. Contoh: Seorang peretas dapat memanggil suatu fungsi secara langsung dan menarik uang. |
| **Medium Likelihood** | Mungkin terjadi dalam kondisi spesifik. Contoh: Token ERC20 yang tidak biasa digunakan di platform tersebut. |
| **Low Likelihood** | Tidak mungkin terjadi. Contoh: Variabel yang sulit diubah disetel ke nilai unik pada waktu tertentu. |

### Catatan Penting tentang Likelihood

Ada situasi di mana kemungkinan dianggap **"computationally infeasible"** (tidak layak secara komputasi). Contohnya: *"Penyerang bisa menebak kunci privat pengguna."* Dalam kasus seperti ini, penyerang harus **mendemonstrasikan** bahwa temuan mereka layak secara komputasi. Jika tidak, temuan tersebut tidak dianggap sebagai vektor serangan yang valid.

---

## Matriks Severity (Impact vs Likelihood)

Berikut adalah panduan untuk menentukan severity berdasarkan kombinasi Impact dan Likelihood:

| (Impact \ Likelihood) | High Likelihood | Medium Likelihood | Low Likelihood |
|---|---|---|---|
| **High Impact** | **High** | High / Medium | Medium |
| **Medium Impact** | High / Medium | Medium | Medium / Low |
| **Low Impact** | Medium | Medium / Low | **Low** |

---

## Contoh Temuan Berdasarkan Severity

### ✅ High Severity (Severitas Tinggi)

- **Dampak:** Berdampak langsung pada dana atau fungsi utama protokol.
- **Jalur Serangan:** Jelas dan mudah dieksploitasi.
- **Konsekuensi:** Kerusakan yang signifikan.

### ⚠️ Medium Severity (Severitas Menengah)

- **Dampak:** Dana berisiko secara tidak langsung atau terjadi gangguan fungsional.
- **Jalur Serangan:** Membutuhkan kondisi khusus untuk dieksploitasi.

### ℹ️ Low Severity (Severitas Rendah)

- **Dampak:** Dana tidak berisiko. Namun, ada fungsi yang tidak tepat atau state yang ditangani secara tidak benar.

---

## Informasi Penting dari CodeHawks

> **Per 18 Agustus 2023, CodeHawks tidak lagi menerima submission dengan severity Gas, QA, atau Informational.**

Artinya, dalam konteks kompetisi audit seperti CodeHawks, hanya temuan yang berdampak langsung atau tidak langsung pada protokol yang diterima. Optimasi gas, jaminan kualitas, atau catatan informasional tidak lagi dianggap sebagai submission yang valid untuk mendapatkan reward.

---

## Penerapan pada Audit PasswordStore

Dengan pemahaman tentang impact dan likelihood, kita dapat mulai menerapkan metodologi ini pada temuan-temuan di PasswordStore:

| Temuan | Impact | Likelihood | Severity |
|---|---|---|---|
| **Access Control pada `setPassword()`** | High (dana berisiko) | High (siapa pun bisa memanggil) | **High** |
| **Penyimpanan password di on-chain** | High (privasi rusak total) | High (data bisa dibaca siapa pun) | **High** |
| **NatSpec tidak akurat pada `getPassword()`** | Low (dana tidak berisiko) | Low (hanya masalah dokumentasi) | **Informational** |

> **Catatan:** Temuan NatSpec yang tidak akurat umumnya dimasukkan ke kategori **Informational** atau **Low**, tergantung pada kebijakan platform atau preferensi klien. CodeHawks sendiri tidak lagi menerima submission Informational.

---

## Kesimpulan

- Severity suatu temuan ditentukan oleh **Impact** dan **Likelihood**.
- Gunakan **matriks** untuk membantu menentukan tingkat keparahan.
- **High Severity** → dana langsung berisiko, mudah dieksploitasi.
- **Medium Severity** → dana berisiko tidak langsung, perlu kondisi khusus.
- **Low Severity** → dana tidak berisiko, tetapi ada ketidaksesuaian fungsional.
- Dalam kompetisi audit modern (seperti CodeHawks), submission **Gas, QA, dan Informational** tidak lagi diterima.
- Selalu pertimbangkan **sudut pandang pengguna**: seberapa parah dampaknya bagi mereka?