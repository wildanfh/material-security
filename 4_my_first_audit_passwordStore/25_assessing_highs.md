# Menentukan Severitas untuk Temuan PasswordStore (Bagian 1)

## Pendahuluan

Setelah memahami konsep **impact** (dampak) dan **likelihood** (kemungkinan), kita sekarang menerapkannya pada tiga temuan di PasswordStore. Dua temuan pertama (storage publik dan access control) dinilai memiliki severity **High**. Materi ini menjelaskan proses penilaian tersebut.

---

## Daftar Temuan yang Akan Dinilai

```

1. Storing the password on-chain makes it visible to anyone and no longer private
2. PasswordStore::setPassword has no access controls, meaning a non-owner could change the password
3. The PasswordStore::getPassword natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect

```

---

## Penilaian Temuan #1: Storage Publik

### Ringkasan Temuan
> Storing the password on-chain makes it visible to anyone and no longer private

### Analisis Impact (Dampak)

| Kategori Impact | Kriteria |
|----------------|----------|
| **High Impact** | Dana secara langsung atau hampir langsung berisiko, ATAU terjadi gangguan parah pada fungsionalitas protokol. |
| Medium Impact | Dana berisiko tidak langsung, ATAU ada gangguan tingkat sedang. |
| Low Impact | Dana tidak berisiko, tetapi ada fungsi yang salah atau state tidak ditangani dengan benar. |

**Pertimbangan:**
- Meskipun **dana tidak secara langsung berisiko**, protokol diharapkan dapat menyembunyikan password dari publik.
- Password yang seharusnya privat menjadi **terlihat oleh siapa saja** → ini adalah **pelanggaran parah terhadap fungsionalitas yang dijanjikan**.
- Dampaknya terhadap kepercayaan pengguna dan tujuan protokol sangat besar.

**Kesimpulan Impact: `High`**

### Analisis Likelihood (Kemungkinan)

| Kategori Likelihood | Kriteria |
|---------------------|----------|
| **High Likelihood** | Sangat mungkin terjadi (misal: peretas bisa memanggil fungsi langsung dan mengambil uang). |
| Medium Likelihood | Mungkin terjadi dalam kondisi spesifik. |
| Low Likelihood | Tidak mungkin terjadi. |

**Pertimbangan:**
- Tidak ada hambatan teknis yang mencegah siapa pun membaca storage.
- Setiap pengguna dengan akses ke node Ethereum dapat menjalankan `eth_getStorageAt` atau `cast storage`.
- **Kepastian hampir 100%** bahwa password akan terbaca jika ada yang berniat.

**Kesimpulan Likelihood: `High`**

### Hasil Severity

| Impact | Likelihood | Severity |
|--------|------------|----------|
| High | High | **High** |

### Label Akhir

```

[H-1] Storing the password on-chain makes it visible to anyone and no longer private

```

> **Pro-tip:** Susun temuan dalam laporan dari **High ke Low** dan dari yang **paling parah ke paling tidak parah**.

---

## Penilaian Temuan #2: Access Control pada `setPassword()`

### Ringkasan Temuan
> `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

### Analisis Impact (Dampak)

**Pertimbangan:**
- Fungsi `setPassword()` seharusnya **hanya bisa dipanggil oleh owner** (berdasarkan NatSpec).
- Dengan tidak adanya access control, **siapa pun dapat mengubah password kapan saja**.
- Ini adalah **gangguan parah terhadap fungsionalitas protokol** – owner kehilangan kendali penuh.

**Kesimpulan Impact: `High`**

### Analisis Likelihood (Kemungkinan)

**Pertimbangan:**
- Tidak ada pemeriksaan `msg.sender` sama sekali.
- **Setiap alamat** dapat memanggil fungsi `setPassword()` secara langsung.
- Tidak diperlukan kondisi khusus atau persiapan rumit.

**Kesimpulan Likelihood: `High`**

### Hasil Severity

| Impact | Likelihood | Severity |
|--------|------------|----------|
| High | High | **High** |

### Label Akhir

```

[H-2] PasswordStore::setPassword has no access controls, meaning a non-owner could change the password

```

---

## Status Laporan Saat Ini

```markdown
### [H-1] Storing the password on-chain makes it visible to anyone and no longer private

### [H-2] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

### [S-#] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect
```

Temuan ketiga (NatSpec tidak akurat) akan dinilai secara terpisah karena bersifat informational (bukan High).

---

Poin Penting

· Impact High tidak selalu berarti kehilangan dana – gangguan parah pada fungsionalitas juga termasuk.
· Likelihood High berarti tidak ada hambatan teknis yang signifikan.
· Kombinasi High + High menghasilkan severity High.
· Susun temuan dalam laporan dari High ke Low untuk memudahkan klien memprioritaskan perbaikan.

---

Kesimpulan

Dua temuan pertama pada PasswordStore dinilai sebagai High Severity karena:

1. Impact-nya tinggi – merusak fungsi utama protokol (privasi password dan kontrol akses).
2. Likelihood-nya tinggi – siapa pun dapat mengeksploitasi tanpa syarat khusus.