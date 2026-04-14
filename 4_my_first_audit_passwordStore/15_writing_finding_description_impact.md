# Mengisi Deskripsi (Description) dan Dampak (Impact) dalam Laporan Temuan

## Pendahuluan

Setelah judul laporan selesai, langkah berikutnya adalah mengisi bagian **Description** (deskripsi) dan **Impact** (dampak). Kedua bagian ini sangat penting untuk meyakinkan protokol bahwa kerentanan yang ditemukan valid dan perlu ditangani.

---

## Template Laporan Saat Ini

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:**

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
```

---

## Bagian 1: Description (Deskripsi)

### Tujuan
- Menjelaskan kerentanan secara **ringkas** namun **jelas**.
- Mengilustrasikan **mengapa** masalah ini terjadi.
- Memberikan konteks tentang kode yang bermasalah.

### Contoh Deskripsi Awal (Kurang Optimal)

```markdown
**Description:** All data stored on chain is public and visible to anyone. The s_password variable is intended to be hidden and only accessible by the owner through the getPassword function.

I show one such method of reading any data off chain below.
```

### Masalah dari Contoh di Atas
- Nama variabel `s_password` dan fungsi `getPassword` tidak mencolok.
- Tidak jelas di kontrak mana variabel dan fungsi tersebut berada.
- Dalam codebase yang besar, referensi seperti ini mudah hilang atau membingungkan.

### Peningkatan dengan Markdown dan Konvensi Penamaan

Gunakan **backticks** (`) untuk membungkus nama variabel, fungsi, dan kontrak. Serta **prepend** nama kontrak sebelum variabel/fungsi.

```markdown
**Description:** All data stored on chain is public and visible to anyone. The `PasswordStore::s_password` variable is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function.

I show one such method of reading any data off chain below.
```

### Manfaat Format Ini
| Teknik | Manfaat |
|--------|---------|
| Backticks | Menyorot nama teknis agar mudah dikenali |
| `Kontrak::fungsi` | Menunjukkan lokasi tepat di mana kode berada |
| Konsistensi | Memudahkan pencarian dan referensi silang |

> *"This is the kind of clarity we should strive for in our reports!"*

---

## Bagian 2: Impact (Dampak)

### Tujuan
- Menjelaskan **konsekuensi** dari kerentanan jika dieksploitasi.
- Menunjukkan **severity** (tingkat keparahan) secara implisit.

### Contoh Penulisan Impact

```markdown
**Impact:** Anyone is able to read the private password, severely breaking the functionality of the protocol.
```

### Karakteristik Impact yang Baik
- **Spesifik** – bukan hanya "ini berbahaya", tapi jelaskan apa yang bisa dilakukan penyerang.
- **Terhubung ke invariant protokol** – tunjukkan bagaimana kerentanan melanggar janji protokol.
- **Menggambarkan kerusakan** – misal: kehilangan privasi, kehilangan dana, manipulasi sistem.

> Untuk PasswordStore, impact-nya adalah siapa pun bisa membaca password, yang **sepenuhnya merusak fungsi protokol** (karena tujuannya adalah menyimpan password secara privat).

---

## Hasil Akhir Sementara

Setelah mengisi Description dan Impact, laporan menjadi:

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** All data stored on chain is public and visible to anyone. The `PasswordStore::s_password` variable is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function.

I show one such method of reading any data off chain below.

**Impact:** Anyone is able to read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:**

**Recommended Mitigation:**
```

---

## Yang Perlu Diingat

- **Description** harus menjelaskan **apa** masalahnya dan **mengapa** itu terjadi.
- Gunakan **format markdown** (backticks, `Kontrak::fungsi`) untuk kejelasan.
- **Impact** harus menggambarkan **konsekuensi nyata** dari eksploitasi.
- Kedua bagian ini bekerja sama untuk **meyakinkan protokol** bahwa temuan valid dan serius.
