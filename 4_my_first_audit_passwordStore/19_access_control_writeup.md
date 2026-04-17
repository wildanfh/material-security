# Menulis Laporan Temuan untuk Access Control (PasswordStore)

## Pendahuluan

Setelah berhasil menyusun laporan untuk kerentanan storage publik, sekarang kita akan membuat laporan untuk temuan kedua: **Access Control** pada fungsi `setPassword`. Materi ini memandu langkah demi langkah mengisi template laporan dengan fokus pada **Title**, **Description**, dan **Impact**.

---

## Kode yang Bermasalah

```solidity
/*
 * @notice This function allows only the owner to set a new password.
 * @param newPassword The new password to set.
 */
function setPassword(string memory newPassword) external {
    s_password = newPassword;
    emit SetNetPassword();
}
```

- **NatSpec** menyatakan: hanya owner yang boleh memanggil.
- **Implementasi**: tidak ada pemeriksaan akses sama sekali.

---

## Template Laporan Kosong

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:**

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
```

---

## Langkah 1: Mengisi Title

Aturan praktis: **Root Cause + Impact**

| Pertanyaan | Jawaban |
|------------|---------|
| Root Cause (akar penyebab) | `setPassword` has no access control |
| Impact (dampak) | non-owner can change the password |

### Judul yang Dihasilkan

```markdown
### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password
```

> Judul ini langsung menyebutkan lokasi spesifik (`PasswordStore::setPassword`), akar masalah, dan dampaknya.

---

## Langkah 2: Mengisi Description

Deskripsi harus:
- **Jelas dan ringkas**.
- Menggunakan bahasa yang mudah dipahami.
- Menyebutkan kontradiksi antara NatSpec dan implementasi.
- (Opsional) menyertakan kode dengan komentar audit.

### Contoh Deskripsi (dari materi)

```markdown
**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

```solidity
function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}
```
```

> **Catatan:** Komentar `// @Audit` membantu menandai lokasi masalah secara visual.

---

## Langkah 3: Mengisi Impact

Dampak harus:
- Menjelaskan konsekuensi nyata.
- Dihubungkan dengan **invariant** protokol yang dilanggar.

### Contoh Impact

```markdown
**Impact:** Anyone can set/change the stored password, severely breaking the contract's intended functionality.
```

- **Anyone can set/change** → siapa pun bisa mengubah.
- **Severely breaking the intended functionality** → merusak fungsi utama kontrak.

---

## Hasil Sementara Laporan

```markdown
### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

```solidity
function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}
```

**Impact:** Anyone can set/change the stored password, severely breaking the contract's intended functionality

**Proof of Concept:**

**Recommended Mitigation:**
```

---

## Ringkasan Poin Penting

| Komponen | Tips |
|----------|------|
| **Title** | Root Cause + Impact, sebut lokasi spesifik (`Kontrak::fungsi`) |
| **Description** | Bandingkan NatSpec dengan implementasi, gunakan kode dengan komentar `@Audit` |
| **Impact** | Jelaskan siapa yang bisa melakukan apa dan seberapa parah kerusakannya |
| **Proof of Concept** | (akan diisi di pelajaran berikutnya) |
| **Recommended Mitigation** | (akan diisi di pelajaran berikutnya) |

---

## Kesimpulan

- Laporan temuan untuk Access Control mengikuti struktur yang sama dengan temuan sebelumnya.
- **Title** harus mencerminkan root cause dan impact secara langsung.
- **Description** perlu menyoroti kontradiksi antara dokumentasi dan implementasi.
- **Impact** menekankan pelanggaran terhadap invariant protokol.
- Setelah tiga komponen ini selesai, laporan siap dilanjutkan ke **Proof of Concept** dan **Recommended Mitigation**.

> *"Already our report looks incredibly professional. Next lesson we're applying our knowledge to construct a Proof of Code. Don't stop now!"*