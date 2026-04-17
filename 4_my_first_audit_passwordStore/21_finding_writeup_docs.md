# Laporan untuk Temuan Informational (NatSpec Tidak Akurat)

## Pendahuluan

Temuan ketiga dalam audit PasswordStore bersifat **informational** – tidak berdampak besar pada keamanan, tetapi tetap perlu dilaporkan karena menyangkut kualitas dokumentasi. Materi ini memandu pembuatan laporan untuk temuan semacam ini dengan struktur yang lebih ringkas (tanpa Proof of Concept).

---

## Kode yang Bermasalah

```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
```

**Masalah:** NatSpec mencantumkan parameter `newPassword`, tetapi fungsi sebenarnya tidak memiliki parameter apa pun (`getPassword()` vs `getPassword(string)`).

---

## Template Laporan untuk Temuan Informational

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:**

**Impact:**

**Proof of Concept:** (opsional, sering dihilangkan)

**Recommended Mitigation:**
```

> Untuk temuan informational, **Proof of Concept** tidak diperlukan karena tidak ada eksploitasi teknis yang perlu dibuktikan.

---

## Langkah 1: Mengisi Title

Aturan praktis: **Root Cause + Impact**

| Komponen | Jawaban |
|----------|---------|
| Root Cause | NatSpec describes a parameter that doesn't exist |
| Impact | NatSpec is incorrect |

### Judul yang Dihasilkan

```markdown
### [S-#] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.
```

---

## Langkah 2: Mengisi Description

Cukup tempelkan bagian kode yang bermasalah (dengan penanda `@>` untuk menunjukkan baris spesifik) dan jelaskan singkat.

```markdown
**Description:**
```solidity
/*
 * @notice This allows only the owner to retrieve the password.
@> * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
```

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it should be `getPassword(string)`.
```

> Tanda `@>` tidak diperlukan secara teknis, tetapi membantu menyorot baris yang salah.

---

## Langkah 3: Mengisi Impact

Dampak dari temuan informational biasanya ringan dan berkaitan dengan kualitas dokumentasi.

```markdown
**Impact:** The natspec is incorrect.
```

---

## Langkah 4: Mengisi Recommended Mitigation

Rekomendasi mitigasi untuk masalah dokumentasi sederhana: **hapus baris yang salah**.

### Menggunakan Diff Syntax untuk Menunjukkan Perubahan

Markdown mendukung `diff` highlighting untuk menunjukkan baris yang ditambahkan (`+`) atau dihapus (`-`).

```diff
    /*
     * @notice This allows only the owner to retrieve the password.
-    * @param newPassword The new password to set.
     */
```

**Sintaks:**

````markdown
```diff
+ line you want to add (shown in green)
- line you want to remove (shown in red)
```
````

### Rekomendasi Lengkap

```markdown
**Recommended Mitigation:** Remove the incorrect natspec line.

```diff
    /*
     * @notice This allows only the owner to retrieve the password.
-    * @param newPassword The new password to set.
     */
```
```

---

## Laporan Akhir (Lengkap)

<details>
<summary>Finding #3 Report - Informational</summary>

```markdown
### [S-#] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.

**Description:**
```solidity
/*
 * @notice This allows only the owner to retrieve the password.
@> * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
```

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it should be `getPassword(string)`.

**Impact:** The natspec is incorrect.

**Recommended Mitigation:** Remove the incorrect natspec line.

```diff
    /*
     * @notice This allows only the owner to retrieve the password.
-    * @param newPassword The new password to set.
     */
```
```

</details>

---

## Perbedaan dengan Temuan High/Critical

| Aspek | Temuan High/Critical | Temuan Informational |
|-------|----------------------|----------------------|
| **Proof of Concept** | Wajib (PoC teknis) | Tidak diperlukan |
| **Impact** | Kehilangan dana, pelanggaran invariant | Dokumentasi tidak akurat |
| **Mitigasi** | Perubahan kode, redesign arsitektur | Perbaikan komentar |
| **Prioritas perbaikan** | Segera (sebelum deploy) | Bisa ditunda |

---

## Kesimpulan

- Temuan informational tetap **perlu dilaporkan** karena dokumentasi yang akurat adalah bagian dari kode yang profesional.
- Gunakan **struktur yang sama** (Title, Description, Impact, Recommended Mitigation) tetapi **PoC dapat dihilangkan**.
- **Diff syntax** (` ```diff `) sangat berguna untuk menunjukkan perubahan yang direkomendasikan pada dokumentasi atau kode.
- Laporan yang bersih dan profesional meningkatkan kredibilitas auditor, bahkan untuk temuan kecil.