# Metode Audit "The Tincho" – Pendekatan Tincho Abbate dalam Security Review

## Pendahuluan: Siapa Itu Tincho?

**Tincho Abbate** adalah legenda di bidang keamanan Web3. Ia adalah anggota [**The Red Guild**](https://theredguild.org/), firma keamanan smart contract dan EVM. Sebelumnya, ia pernah menjabat sebagai *lead auditor* di OpenZeppelin. Tincho juga berkontribusi dalam pembuatan kursus ini.

Metode audit yang akan dipelajari berasal dari rekaman langsung (*live auditing*) yang dilakukan Tincho pada **Ethereum Name Service (ENS)**.

> *"I don't have a super formal auditing process. I will just show you briefly some things that I do..."* — Tincho

---

## 1. Langkah Pertama: Baca Dokumentasi

Sebelum menyentuh kode:

- **Download kode** dari repositori.
- **Baca dokumentasi** secara menyeluruh.
- Pahami konteks dan jargon yang digunakan.
- Kenali apa yang diharapkan dilakukan oleh protokol.

> **Penekanan:** *READ THE DOCUMENTATION* – ini adalah fondasi dari seluruh proses audit.

---

## 2. Tools yang Digunakan Tincho

Tincho menggunakan (dan merekomendasikan) alat-alat berikut:

| Alat | Kegunaan |
|------|----------|
| **VS Codeium** | Text editor berbasis VS Code tetapi menghilangkan telemetri (fokus privasi). |
| **Foundry** | Framework untuk testing dan fuzzing. Cepat dan memiliki test suite yang robust. |
| **CLOC** | Menghitung jumlah baris kode (nSLOC) untuk mengukur kompleksitas berbagai bagian codebase. |
| **Solidity Metric** | Alat dari Consensys yang memberikan metrik berguna tentang kode Solidity. |

### Strategi Penggunaan Tools

- Gunakan **CLOC** dan **Solidity Metric** untuk mengorganisasi codebase berdasarkan tingkat kompleksitas.
- Mulailah dengan kontrak yang **lebih kecil dan lebih mudah dikelola**, lalu tingkatkan secara sistematis.
- Tandai setiap kontrak yang sudah selesai direview agar tidak ada yang terlewat (*systematic approach*).

---

## 3. Mindset Adversarial (Berpikir Seperti Penyerang)

Pada titik tertentu dalam audit, pola pikir Anda harus **beralih menjadi adversarial**:

> *"How can I break this..."*

### Contoh Pertanyaan yang Harus Diajukan

Bahkan untuk fungsi yang sederhana, tanyakan:

- *"Will this work for every type of token?"*  
  (Contoh: USDT adalah *weird ERC20* yang tidak mengembalikan boolean pada `transferFrom`.)

- *"Have they implemented access control modifiers properly?"*

- *"Apa yang terjadi jika input ini bernilai nol? Negatif? Sangat besar?"*

> Jangan pernah berasumsi bahwa kode "pasti benar" hanya karena terlihat sederhana.

---

## 4. Mencatat Temuan (Note-Taking)

> *"Tincho recommends taking notes directly in the code AND maintaining a separate file for raw notes/ideas."*

Dua tempat mencatat:

| Lokasi | Tujuan |
|--------|--------|
| **Langsung di kode (komentar)** | Menandai bagian yang mencurigakan, pertanyaan, atau potensi bug. |
| **File terpisah (raw notes)** | Menampung ide-ide besar, skenario serangan, dan temuan yang belum terstruktur. |

### Peringatan

- Jangan terlalu larut dalam satu bagian kode hingga kehilangan **gambaran besar** (*big picture*).
- Secara berkala, **"pop back up"** dan tinjau ulang keseluruhan codebase.

---

## 5. Menggunakan Test (Fuzzing) untuk Memverifikasi

Tidak semua pekerjaan bersifat *manual review*. Tincho sangat menganjurkan penggunaan **fuzz test** untuk memverifikasi kecurigaan.

Contoh: Dalam audit ENS, Tincho menerapkan fuzz test untuk menilai fungsi-fungsi tertentu.

> Jika Anda ragu apakah suatu invariant selalu benar, **tulis fuzz test** dan biarkan Foundry yang menjalankan ribuan kasus acak.

---

## 6. Komunikasi dengan Klien

> *"I would advise to keep the clients at hand. Ask questions, but also be detached enough."* — Tincho

### Prinsip Komunikasi

| Do | Don't |
|----|-------|
| Tanyakan pertanyaan kepada klien. | Jangan terlalu bergantung pada klien untuk menemukan bug. |
| Gunakan klien sebagai **kolaborator** untuk memahami intended behavior. | Jangan berasumsi bahwa klien selalu benar. |
| **Trust but validate** – percaya, tetapi verifikasi. | |

> Klien memiliki konteks bisnis yang jauh lebih dalam daripada Anda. Manfaatkan mereka, tetapi tetap jaga independensi sebagai auditor.

---

## 7. Time-Boxing: Batasi Waktu Audit

> *"The thing is...I always get the feeling that you can be looking at a system forever."* — Tincho

- Tidak ada habisnya jika terus-menerus mencari bug.
- **Tetapkan batas waktu** (*time-bound yourself*).
- Lakukan yang terbaik dalam batasan waktu tersebut.
- Jangan perfeksionis hingga audit molor tak terbatas.

---

## 8. Laporan Audit dan Tindak Lanjut

Setelah audit selesai:

- **Buat laporan** yang jelas dan ringkas.
- Detailkan kerentanan yang ditemukan.
- Berikan **rekomendasi mitigasi**.
- **Review implementasi mitigasi** oleh klien – pastikan tidak ada *new bugs* yang diperkenalkan.

> Tanggung jawab auditor tidak berhenti di laporan; Anda harus memastikan perbaikan benar-benar aman.

---

## 9. Jika Ada Kerentanan yang Terlewat

> *"There will always be the fear of missing out on some vulnerabilities."*

### Sikap yang Benar

- Jangan terlalu cemas dengan bug yang lolos.
- Fokus pada **memberikan nilai lebih** dari sekadar menemukan bug:
  - Jadilah *collaborative security partner*.
  - Didik klien tentang praktik terbaik.
  - Bantu mereka bersiap secara holistik.

### Siapa yang Bertanggung Jawab?

- Auditor **tidak menanggung seluruh beban** ketika eksploitasi terjadi.
- Tanggung jawab **dibagi** dengan klien (tim pengembang, manajemen, proses internal mereka).

> *"This doesn't give you free reign to suck at your job. People will notice."*  
> Tetaplah profesional dan lakukan yang terbaik.

---

## Pesan Penutup dari Tincho

> *"Knowing that you’re doing your best in that, knowing that you’re putting your best effort every day, growing your skills, learning grows an intuition and experience in you."*

- Lakukan yang terbaik setiap hari.
- Terus tingkatkan keterampilan.
- Pengalaman dan intuisi akan tumbuh seiring waktu.

---

## Ringkasan Metode "The Tincho"

| Fase | Aktivitas Utama |
|------|-----------------|
| **Persiapan** | Baca dokumentasi, pahami konteks. |
| **Tools** | Gunakan CLOC, Solidity Metric, Foundry, VS Codeium. |
| **Pendekatan** | Mulai dari kontrak kecil → sistematis → beri tanda selesai. |
| **Mindset** | Adversarial: "How can I break this?" |
| **Pencatatan** | Komentar di kode + file raw notes terpisah. |
| **Verifikasi** | Gunakan fuzz test untuk menguji kecurigaan. |
| **Komunikasi** | Tanya klien, trust but validate. |
| **Batasan** | Time-boxing, jangan lihat sistem selamanya. |
| **Pelaporan** | Laporan jelas + review mitigasi. |
| **Sikap** | Jangan takut bug terlewat, fokus beri nilai lebih. |