# Ringkasan Materi: Centralization Risk – Temuan Aderyn pada ThunderLoan

## Pendahuluan

Setelah menjalankan Aderyn pada ThunderLoan, ditemukan beberapa temuan. Yang pertama dan paling signifikan adalah **Centralization Risk** (L-1) – risiko sentralisasi karena adanya fungsi-fungsi yang hanya dapat dipanggil oleh `owner`. Temuan ini penting untuk dicatat meskipun sering dianggap "known issue" oleh tim protokol.

---

## Centralization Risk (L-1)

### Deskripsi

Aderyn mendeteksi bahwa kontrak ThunderLoan memiliki beberapa fungsi dengan modifier `onlyOwner`, yang berarti **hanya pemilik (owner)** yang dapat memanggilnya. Jika owner adalah satu orang atau entitas yang tidak terpercaya, hal ini menimbulkan risiko sentralisasi yang signifikan.

### Fungsi dengan `onlyOwner`

| Fungsi | Lokasi | Risiko |
|--------|--------|--------|
| `setAllowedToken` | `ThunderLoan.sol` line 239 | Menambah/menghapus token yang didukung – dapat merusak pasar atau memblokir aset tertentu. |
| `updateFlashLoanFee` | `ThunderLoan.sol` line 265 | Mengubah biaya flash loan – dapat mengambil keuntungan tidak adil atau membuat pinjaman terlalu mahal. |
| `_authorizeUpgrade` | `ThunderLoan.sol` line 292 | Mengotorisasi upgrade kontrak – dapat mengubah seluruh logika protokol. |

### Mengapa Ini Risiko?

- **Risiko rug pull (penarikan dana):** Owner dapat meng-upgrade kontrak dengan implementasi jahat yang menguras dana.
- **Risiko manipulasi:** Owner dapat mengubah fee atau token yang didukung untuk keuntungan pribadi.
- **Trust assumption:** Protokol mengharuskan pengguna mempercayai owner untuk tidak bertindak jahat.

### Tanggapan Umum dari Protokol

Biasanya tim protokol akan merespons:

> *"That's a known issue."*

Namun, sebagai auditor, kita tetap wajib mencatatnya. Ini penting untuk melindungi diri sendiri (coverage) jika protokol ternyata adalah *rug pull* atau terjadi penyalahgunaan di masa depan.

---

## Temuan Lain dari Aderyn

Selain centralization, Aderyn juga mendeteksi beberapa temuan lain (akan dibahas lebih lanjut di pelajaran berikutnya):

| ID | Temuan | Severity |
|----|--------|----------|
| L-2 | Missing zero address checks | Low |
| L-3 | Fungsi `public` bisa diubah menjadi `external` | Low |
| L-4 | Event missing `indexed` fields | Low |
| L-5 | PUSH0 tidak didukung semua rantai | Low |
| L-6 | Empty block (`_authorizeUpgrade`) | Low |

---

## Visualisasi Risiko Sentralisasi

Untuk memahami lebih dalam, kunjungi repositori [**sc-exploits-minimized**](https://github.com/Cyfrin/sc-exploits-minimized) – bagian `centralization` menyediakan contoh kontrak dan lingkungan Remix untuk bereksperimen.

---


**Kesimpulan:** Temuan centralization adalah peringatan penting yang harus selalu dicatat dalam laporan audit, meskipun sering diabaikan oleh protokol.