# Ringkasan Materi: Static Analysis – Slither dan Aderyn pada ThunderLoan

## Pendahuluan

Setelah menyelesaikan fase reconnaissance, langkah selanjutnya adalah **menjalankan static analysis** untuk menemukan kerentanan awal ("low hanging fruit") sebelum melakukan manual review mendalam. ThunderLoan menggunakan `Slither` dan `Aderyn` dengan konfigurasi yang sudah disiapkan.

---

## Slither

### Konfigurasi Slither

Setiap repositori di kursus ini memiliki perintah `make slither` yang sudah dikonfigurasi. File konfigurasi `slither.config.json` menentukan perilaku alat, misalnya:

```json
{
  "detectors_to_exclude": "conformance-to-solidity-naming-conventions,incorrect-versions-of-solidity",
  "exclude_informational": false,
  "exclude_low": false,
  "exclude_medium": false,
  "exclude_high": false,
  "disable_color": false,
  "filter_paths": "(mocks/|test/|script/|upgradedProtocol/)",
  "legacy_ast": false,
  "exclude_dependencies": true
}
```

**Penjelasan konfigurasi:**

| Pengaturan | Fungsi |
|------------|--------|
| `detectors_to_exclude` | Mengecualikan detector tertentu (misal: penamaan konvensi, versi Solidity) untuk mengurangi noise. |
| `exclude_*` | Memastikan semua severity (Informational, Low, Medium, High) tetap dilaporkan. |
| `filter_paths` | Mengabaikan folder seperti `mocks/`, `test/`, `script/`, dan `upgradedProtocol/` agar fokus pada kode utama. |
| `exclude_dependencies` | Mengabaikan dependensi eksternal (seperti OpenZeppelin). |

### Menjalankan Slither

```bash
make slither
```

### Temuan dari Slither

#### 1. Missing Event pada State Change

**Lokasi:** `ThunderLoan.sol` line 269

**Kode bermasalah:**
```solidity
function updateFlashLoanFee(uint256 newFee) external onlyOwner {
    if (newFee > s_feePrecision) {
        revert ThunderLoan__BadNewFee();
    }
    s_flashLoanFee = newFee; // @Audit-Low: Events should be emitted on state change
}
```

**Masalah:** Perubahan state (`s_flashLoanFee`) tidak diikuti dengan event emission. Ini menyulitkan sistem off-chain untuk melacak perubahan.

**Tindakan:** Tandai sebagai temuan.

#### 2. Potensi Reentrancy

Slither mendeteksi beberapa situasi reentrancy yang berbeda dari yang biasa kita temui.

```solidity
// @Audit - Follow Up
```

Tambahkan tag `@Audit - Follow Up` pada setiap external call yang disebutkan Slither untuk ditinjau ulang nanti.

---

## Aderyn

### Menjalankan Aderyn

```bash
make aderyn
```

atau

```bash
aderyn .
```

### Temuan dari Aderyn

Aderyn melaporkan beberapa temuan tambahan yang akan dibahas lebih lanjut di pelajaran berikutnya.

---

## Ringkasan Temuan Awal

| Alat | Temuan | Lokasi | Severity |
|------|--------|--------|----------|
| Slither | Missing event on state change | `ThunderLoan.sol::updateFlashLoanFee` | Low |
| Slither | Potensi reentrancy | Beberapa external call | Perlu investigasi |
| Aderyn | (akan dibahas) | — | — |

---

## Kesimpulan

- **Slither dan Aderyn** membantu menemukan kerentanan awal secara otomatis.
- Konfigurasi yang tepat (seperti `filter_paths` dan `detectors_to_exclude`) membuat output lebih relevan.
- Temuan seperti **missing event** dan **potensi reentrancy** perlu dicatat untuk ditindaklanjuti dalam manual review.
- Static analysis adalah langkah pertama, tetapi **tidak menggantikan manual review** – temuan harus diverifikasi secara manual.