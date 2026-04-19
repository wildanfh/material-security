# Menentukan Severitas untuk Temuan Informational (PasswordStore)

## Pendahuluan

Temuan ketiga dalam audit PasswordStore berbeda dari dua temuan sebelumnya karena tidak berdampak langsung pada keamanan dana atau fungsionalitas protokol. Temuan ini bersifat **informational** – terkait kualitas dokumentasi. Materi ini menjelaskan bagaimana menilai severity untuk kasus seperti ini.

---

## Temuan #3: NatSpec Tidak Akurat

### Ringkasan Temuan
> The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.

**Kode bermasalah:**
```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.   // ← parameter ini tidak ada
 */
function getPassword() external view returns (string memory) {}
```

---

Analisis Impact (Dampak)

Mengacu pada kriteria yang telah ditetapkan:

Kriteria Apakah terpenuhi?
High Impact: Dana langsung berisiko atau gangguan parah pada protokol ❌ Tidak
Medium Impact: Dana tidak langsung berisiko atau ada gangguan tingkat sedang ❌ Tidak
Low Impact: Dana tidak berisiko, tetapi fungsi salah atau state tidak ditangani dengan benar ❌ Tidak (fungsi bekerja dengan benar, hanya dokumentasinya yang salah)

Kesimpulan Impact: NONE (tidak ada dampak terhadap keamanan atau fungsionalitas)

---

Analisis Likelihood (Kemungkinan)

Meskipun impact-nya tidak ada, likelihood-nya tidak relevan karena tidak ada eksploitasi yang mungkin terjadi. Namun, jika dipaksakan:

· Tidak ada skenario eksploitasi.
· Tidak ada kondisi yang memungkinkan kerusakan.

Likelihood tidak menjadi faktor penentu ketika impact-nya nol.

---

Penentuan Severity: Informational

Ketika suatu temuan bukan bug (tidak menyebabkan kerusakan) tetapi tetap perlu dilaporkan untuk meningkatkan kualitas kode, maka dimasukkan ke dalam kategori:

Kategori Jenis Masalah
Informational (atau Non-Critical) Dokumentasi yang salah, kesalahan ejaan, pola desain yang bisa ditingkatkan, cakupan tes yang kurang, dll.
Gas Optimasi biaya gas (boros, tetapi tidak mempengaruhi keamanan).

Untuk temuan ini:

· Bukan bug – fungsi berjalan dengan benar.
· Hanya dokumentasi – NatSpec mencantumkan parameter yang tidak ada.
· Severity: Informational

---

Label Akhir

Setelah menetapkan severity, label temuan menjadi:

```markdown
### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.
```

· I = Informational
· 1 = nomor urut temuan dalam kategori Informational

---

Status Laporan Lengkap

```markdown
### [H-1] Storing the password on-chain makes it visible to anyone and no longer private

### [H-2] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect
```

Pro-tip: Temuan disusun dari severity tertinggi ke terendah (High → Medium → Low → Informational/Gas).

---

Ringkasan Kategori Severity untuk Temuan Non-Bug

Kategori Contoh
Informational Dokumentasi salah, komentar usang, penamaan variabel tidak jelas, kurangnya event, potensi perbaikan pola desain.
Gas Penggunaan storage yang boros, loop yang tidak efisien, operasi yang bisa dioptimalkan.

Catatan: Dalam kompetisi audit seperti CodeHawks, submission Informational dan Gas tidak lagi diterima untuk mendapatkan reward. Namun, dalam private audit, tetap profesional untuk melaporkannya.

---

Kesimpulan

· Temuan ketiga pada PasswordStore tidak memiliki impact terhadap keamanan atau fungsionalitas.
· Karena itu, severity ditetapkan sebagai Informational (bukan High/Medium/Low).
· Laporan tetap mencantumkannya untuk menunjukkan perhatian terhadap kualitas kode dan dokumentasi.
· Setelah semua temuan dinilai, laporan siap untuk dikonversi ke PDF profesional.