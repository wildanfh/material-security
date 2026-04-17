# Struktur Laporan Temuan (Finding Report)

## Pendahuluan

Setelah menyelesaikan laporan temuan untuk kerentanan pertama (`PasswordStore::s_password`), saatnya merekap elemen-elemen penting yang membentuk sebuah laporan temuan yang profesional. Materi ini mengulas **lima komponen utama** dalam struktur laporan temuan: **Title**, **Description**, **Impact**, **Proof of Code**, dan **Recommended Mitigation**.

---

## Hasil Akhir Laporan Temuan (Contoh)

<details closed>
<summary>Finding Report (PasswordStore - Storage Public)</summary>

### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** All data stored on chain is public and visible to anyone. The `PasswordStore::s_password` variable is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function.  
I show one such method of reading any data off chain below.

**Impact:** Anyone is able to read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:** [Langkah-langkah dengan `make anvil`, `make deploy`, `cast storage`, `cast parse-bytes32-string`]

**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you would also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.

</details>

> **Catatan:** `[S-#]` adalah placeholder untuk severity (misal `[H-1]` untuk High, `[M-1]` untuk Medium, dll).

---

## Lima Komponen Utama Laporan Temuan

### 1. Title (Judul)

| Aturan | Penjelasan |
|--------|------------|
| **Ringkas dan jelas** | Judul harus langsung menjelaskan inti masalah. |
| **Root Cause + Impact** | Cantumkan akar penyebab dan dampak. Contoh: `Storing the password on-chain makes it visible to anyone and no longer private` |

> Judul yang baik memudahkan pembaca (terutama tim protokol) memahami tingkat keparahan tanpa membaca detail.

### 2. Description (Deskripsi)

- **Penjelasan singkat** tentang masalah.
- Gunakan **markdown** (backticks, `Kontrak::variabel`) untuk memperjelas referensi kode.
- Sebutkan mengapa kondisi ini seharusnya tidak terjadi (misal: berdasarkan invariant protokol).

> Contoh: `The PasswordStore::s_password variable is intended to be hidden and only accessible by the owner...`

### 3. Impact (Dampak)

- Jelaskan **konsekuensi nyata** dari kerentanan dalam bahasa yang jelas (non-teknis).
- Hubungkan dengan **janji protokol** (invariant) yang dilanggar.
- Hindari bahasa ambigu: "Severely breaking the functionality" lebih baik daripada "This is bad".

> Contoh: `Anyone is able to read the private password, severely breaking the functionality of the protocol.`

### 4. Proof of Code / Concept (PoC)

- **Bagian paling kritis** – bukti bahwa kerentanan benar-benar ada dan dapat dieksploitasi.
- Gunakan **langkah-langkah konkret** yang dapat direproduksi (perintah terminal, kode test, dll).
- Manfaatkan alat standar seperti `cast`, `forge`, `anvil`, atau kode Solidity.
- Tampilkan **output yang diharapkan** (misal: password terbaca).

> PoC yang kuat mencegah protokol mengabaikan temuan. Tanpa PoC, laporan hanya bersifat spekulatif.

### 5. Recommended Mitigation (Rekomendasi Mitigasi)

- **Tempat auditor menunjukkan keahlian** – berikan solusi, bukan hanya kritik.
- Rekomendasi harus **spesifik** dan **dapat ditindaklanjuti** oleh tim pengembang.
- Jika kerentanan bersifat fundamental, akui perlunya **redesain arsitektur** (seperti contoh PasswordStore: enkripsi off-chain).
- Tawarkan **trade-off yang realistis** (misal: user harus mengingat password tambahan).

> Nilai tambah auditor terletak pada kemampuan membantu protokol menjadi lebih aman, bukan sekadar melaporkan bug.

---

## Ringkasan Checklist Laporan Temuan

| Komponen | Pertanyaan yang Harus Dijawab |
|----------|-------------------------------|
| **Title** | Apa root cause + dampaknya? |
| **Description** | Apa masalahnya? Di mana tepatnya dalam kode? |
| **Impact** | Apa konsekuensi jika dieksploitasi? |
| **Proof of Code** | Bagaimana cara membuktikan eksploitasi secara teknis? |
| **Recommended Mitigation** | Bagaimana cara memperbaikinya? |

---

## Kesimpulan

- Laporan temuan yang baik adalah **alat komunikasi** yang meyakinkan protokol untuk mengambil tindakan.
- Gunakan **struktur lima bagian** (Title, Description, Impact, PoC, Mitigation) secara konsisten.
- **Proof of Code** adalah jantung laporan – tanpanya, klaim kerentanan mudah diabaikan.
- **Recommended Mitigation** membedakan auditor biasa dari auditor ahli – tunjukkan solusi, bukan hanya masalah.
- Setelah menyusun laporan untuk satu kerentanan, ulangi proses yang sama untuk temuan lainnya (misal: Access Control, NatSpec tidak akurat).

> *"Our report looks awesome, but there's more to do. No stopping now."*