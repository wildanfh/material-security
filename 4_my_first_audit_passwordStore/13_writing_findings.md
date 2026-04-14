# Fase Reporting dalam Security Review

## Pendahuluan

Setelah fase identifikasi kerentanan, tugas auditor selanjutnya adalah **mengomunikasikan temuan** kepada protokol. Fase **Reporting** ini sangat krusial karena:

1. **Meyakinkan protokol** bahwa kerentanan yang ditemukan valid.
2. **Menjelaskan tingkat keparahan** dan dampak dari masalah tersebut.
3. **Memberikan strategi mitigasi** yang dapat diterapkan.

Dengan komunikasi yang efektif, auditor berperan sebagai **pendidik** – membantu protokol memahami **mengapa** kerentanan itu berbahaya, **mengapa** bisa terlewat, dan **bagaimana** memperbaikinya agar tidak terulang di masa depan.

---

## Menulis Temuan Pertama

Auditor perlu menulis laporan temuan secara minimalis namun informatif. Template tersedia di [GitHub Repo kursus](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/finding_layout.md).

### Langkah-langkah:

1. Buat folder baru bernama `audit-data` di dalam proyek.
2. Download dan salin `finding_layout.md` ke dalam folder tersebut.
3. Buka file dengan preview (`CTRL + SHIFT + V` di VS Code).

### Template Dasar Temuan

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:**

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
```

- `[S-#]` adalah kode severity (misal: `[H-1]` untuk High, `[M-1]` untuk Medium, `[L-1]` untuk Low, `[I-1]` untuk Informational).
- Template ini dapat disesuaikan sesuai kebutuhan, tetapi merupakan titik awal yang baik.

---

## Tujuan Laporan Temuan

| Tujuan | Penjelasan |
|--------|------------|
| **Validasi** | Buktikan bahwa kerentanan benar-benar ada (dengan kode, skenario, atau PoC). |
| **Severity & Impact** | Jelaskan seberapa parah dampaknya jika dieksploitasi (kehilangan dana, pelanggaran invariant, dll). |
| **Mitigasi** | Berikan rekomendasi perbaikan yang konkret dan dapat diimplementasikan. |

> Auditor harus memposisikan diri sebagai **mitra edukatif**, bukan sekadar pencari bug.

---

## Contoh Penerapan

Untuk temuan pertama pada PasswordStore: **"Private variables aren't actually private!"**

Template akan diisi dengan:
- **Description:** Penjelasan bahwa `private` di Solidity tidak menyembunyikan data dari dunia luar.
- **Impact:** Siapa pun dapat membaca password langsung dari storage via `eth_getStorageAt`.
- **Proof of Concept:** Kode atau perintah `cast storage` yang mendemonstrasikan pembacaan.
- **Recommended Mitigation:** Jangan simpan data sensitif di on-chain; gunakan enkripsi atau penyimpanan off-chain.

---

## Langkah Selanjutnya

1. Buat salinan `finding_layout.md` menjadi `findings.md`.
2. Mulai isi setiap temuan yang telah diidentifikasi sebelumnya:
   - Access Control pada `setPassword()`
   - NatSpec tidak akurat pada `getPassword()`
   - Penyimpanan password di on-chain (private tidak berarti rahasia)
3. Susun laporan akhir yang profesional.

---

## Kesimpulan

- **Reporting adalah fase kunci** untuk memastikan protokol memahami dan memperbaiki kerentanan.
- Gunakan **template terstruktur** agar laporan jelas dan konsisten.
- Fokus pada **validasi, dampak, dan mitigasi**.
- Auditor adalah **pendidik** – bantu protokol belajar dari kesalahan.

> *"By effectively communicating this information, we position ourselves as educators."*
