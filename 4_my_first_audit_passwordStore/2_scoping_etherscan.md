# Fase Scoping dalam Security Review

## Pendahuluan

Fase **scoping** adalah langkah awal dalam security review. Pada fase ini, auditor menerima codebase untuk ditinjau dan melakukan penilaian tingkat tinggi (*high level assessment*). Scoping merupakan bagian dari **Initial Review** dan sangat krusial untuk menentukan apakah protokol siap diaudit.

---

## Skenario yang Terlalu Umum (dan Buruk)

Berikut adalah contoh interaksi yang sering terjadi antara klien dan auditor pemula:

> *CLIENT: "Hi, kami tim dev PasswordStore ingin codebase kami diaudit segera agar bisa listing resmi."*
>
> *AUDITOR: "Halo PasswordStore, saya auditor pemula. Sangat antusias membantu. Bisakah kirim codebase-nya?"*
>
> *CLIENT: "Tentu, ini link Etherscan codebase kami."* → [PasswordStore CodeV1](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b)

**Ini adalah pendekatan yang buruk.** Sebagai security researcher, Anda **tidak boleh** mengaudit codebase yang diberikan hanya melalui Etherscan.

---

## Mengapa Etherscan-Only Tidak Cukup?

Security researcher mencari lebih dari sekadar bug. Mereka mencari **kematangan kode** (*code maturity*).

| Pertanyaan yang Harus Diajukan | Implikasi |
|-------------------------------|-----------|
| Apakah ada test suite? | Tanpa test suite, bagaimana Anda tahu kode berfungsi seperti yang diharapkan? |
| Apakah ada deployment suite? | Tanpa deployment script, bagaimana Anda memverifikasi lingkungan produksi? |
| Apakah ada dokumentasi? | Tanpa dokumentasi, Anda tidak bisa memahami invariant dan asumsi desain. |
| Apakah ada CI/CD? | Menunjukkan profesionalisme dan perhatian terhadap kualitas. |

Jika klien hanya memberikan link Etherscan, jawaban atas pertanyaan-pertanyaan di atas hampir pasti **TIDAK**. Maka, **Anda tidak dapat menjamin keamanan protokol tersebut**.

> *"Secure protocols not only safeguard the code but also our reputation as researchers. They will likely blame us for a security breach if we've audited a compromised codebase."*

---

## Audit Readiness dan Rekt Test

Pada materi sebelumnya, kita membahas **Rekt Test** dari Trail of Bits. Ini adalah 12 pertanyaan yang mengukur kesiapan protokol untuk diaudit.

Jika klien hanya memberikan link Etherscan, bagaimana protokol mereka menjawab pertanyaan-pertanyaan seperti:

- Apakah Anda memiliki rencana respons insiden yang tertulis dan diuji?
- Apakah Anda mendokumentasikan invariant utama dan mengujinya setiap commit?
- Apakah Anda menggunakan alat otomatis terbaik untuk menemukan masalah keamanan?
- Apakah Anda memiliki program bug bounty?

**Jawabannya: buruk (poorly).**

---

## Tanda Bahaya (Red Flag)

> *"If you're offered monetary reward to audit an Etherscan-only codebase, that's a red flag. Say NO."*

Menerima bayaran untuk mengaudit codebase tanpa test suite, tanpa repo GitHub, tanpa dokumentasi adalah **kontradiksi dengan misi kita untuk mempromosikan protokol yang aman**.

### Apa yang Harus Dilakukan?

| Tindakan | Penjelasan |
|----------|------------|
| **Tolak klien tersebut** | Jangan bekerja dengan klien yang tidak menunjukkan komitmen yang sama terhadap keamanan. |
| **Edukasi klien** | Jika Anda memutuskan untuk bekerja dengan mereka, gunakan kesempatan untuk mengajari mereka cara menulis test yang baik, menyiapkan framework, dan mempersiapkan kode untuk review. |

### Contoh Respons yang Baik dari Auditor

> *"Halo PasswordStore, terima kasih banyak untuk link Etherscan ini, ini awal yang baik. Namun, apakah Anda memiliki test suite? Kami ingin memastikan codebase Anda seaman mungkin. Apakah Anda memiliki repo Git atau GitHub dengan testing framework?"*

### Respons Klien yang Baik

> *"Ah! Ya, maaf. Kami punya repo Foundry Test yang sudah disiapkan untuk ini, biar saya kirimkan codebase Git-nya."*

---

## Jika Klien Memberi Tekanan

Jika klien merespons pertanyaan Anda dengan tekanan (misal: "Kami butuh audit cepat, tidak punya waktu untuk itu"), maka **berjalanlah (walk away)**.

> *"It's evidence that security isn't their focus."*

Klien yang serius tentang keamanan akan menghargai pertanyaan Anda dan bersedia melengkapi semua yang diperlukan untuk audit yang berkualitas.

---

## Kesimpulan

- **Scoping** adalah fase awal yang menentukan apakah audit layak dilanjutkan.
- **Jangan pernah mengaudit codebase yang hanya diberikan sebagai link Etherscan** tanpa test suite, repo, atau dokumentasi.
- **Gunakan Rekt Test** sebagai alat untuk menilai kesiapan klien.
- **Tolak tawaran audit dari klien yang tidak menunjukkan komitmen terhadap keamanan** – itu adalah red flag.
- **Edukasi klien** jika mereka mau belajar, tetapi **jangan terima tekanan** untuk mengorbankan standar keamanan.

> Sebagai security researcher, reputasi adalah aset paling berharga. Jangan pertaruhkan dengan mengaudit kode yang belum matang hanya demi imbalan finansial.
