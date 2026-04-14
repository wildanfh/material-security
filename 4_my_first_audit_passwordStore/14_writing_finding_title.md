# Menulis Judul Laporan Temuan (Title)

## Pendahuluan

Setelah menyiapkan template laporan, langkah pertama dalam menulis temuan adalah mengisi **judul (title)**. Judul yang baik harus **ringkas** namun tetap mengomunikasikan detail penting dari kerentanan.

Aturan praktis (rule of thumb) yang direkomendasikan:

> **Root Cause + Impact**

Artinya, judul harus mencerminkan:
- **Apa akar penyebab** (root cause) dari kerentanan.
- **Apa dampaknya** (impact) terhadap protokol.

---

## Contoh Penerapan pada PasswordStore

Untuk temuan **"Private variables aren't actually private"**:

| Komponen | Deskripsi |
|----------|-----------|
| **Root Cause** | Storage variables on-chain are publicly visible (variabel penyimpanan di on-chain bersifat publik) |
| **Impact** | Anyone can view the stored password (siapa pun dapat melihat password yang disimpan) |

### Judul yang Dihasilkan:

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private
```

> **Catatan:** `[S-#]` akan diisi dengan kode severity (misal `[H-1]` untuk High) yang akan dijelaskan kemudian.

---

## Hal yang Perlu Diperhatikan

- **Jangan terlalu panjang** – tetap ringkas tetapi informatif.
- **Jangan terlalu pendek** – pastikan root cause dan impact tersampaikan.
- **Gunakan bahasa yang jelas** – hindari ambiguitas.

Contoh judul yang kurang baik:
- ❌ `"Password is not secure"` (tidak menjelaskan root cause)
- ❌ `"Storage visibility issue"` (tidak menjelaskan dampak)

Contoh judul yang baik:
- ✅ `"Storing the password on-chain makes it visible to anyone and no longer private"`

---

## Kesimpulan

- Judul adalah **komponen pertama** yang dilihat pembaca laporan.
- Ikuti aturan **Root Cause + Impact** untuk memastikan judul informatif.
- Judul yang jelas membantu protokol memahami **mengapa** temuan ini penting sebelum membaca detailnya.

> *"The easiest way to ensure a clear title of your report is to be concise and adhere to the rule of thumb: Root Cause + Impact."*
