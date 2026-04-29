# Pelaporan Magic Numbers (Informational)

## Pendahuluan

Dalam proses audit Puppy Raffle, ditemukan penggunaan **magic numbers** (literal angka) pada fungsi `selectWinner`. Magic numbers adalah angka yang ditulis langsung di kode tanpa penjelasan maknanya. Hal ini mengurangi keterbacaan dan maintainability kode.

---

## Kode yang Bermasalah

```solidity
uint256 prizePool = (totalAmountCollected * 80) / 100;
uint256 fee = (totalAmountCollected * 20) / 100;
```

Masalah:
- `80`, `20`, `100` tidak memiliki nama yang menjelaskan tujuannya.
- Pembaca kode harus menebak atau mencari konteks untuk memahami arti angka-angka tersebut.

---

## Mengapa Magic Numbers Dihindari?

| Dampak | Penjelasan |
|--------|------------|
| **Keterbacaan menurun** | Angka literal tidak menjelaskan maksud di baliknya (misal: 80% untuk prize pool? 20% untuk fee?) |
| **Rentan kesalahan** | Jika angka yang sama digunakan di banyak tempat, perubahan nilai harus dilakukan di semua lokasi, rawan terlewat. |
| **Sulit dipelihara** | Developer baru atau reviewer harus mencari tahu arti angka tersebut di dokumentasi atau bertanya ke tim. |

---

## Rekomendasi Mitigasi

Gunakan **konstanta publik** dengan nama yang deskriptif untuk menggantikan magic numbers.

### Contoh Perbaikan

```solidity
uint256 public constant PRIZE_POOL_PERCENTAGE = 80;
uint256 public constant FEE_PERCENTAGE = 20;
uint256 public constant POOL_PRECISION = 100;

uint256 prizePool = (totalAmountCollected * PRIZE_POOL_PERCENTAGE) / POOL_PRECISION;
uint256 fee = (totalAmountCollected * FEE_PERCENTAGE) / POOL_PRECISION;
```

### Keuntungan

- Kode menjadi **self-documenting** – nama konstanta menjelaskan tujuannya.
- Perubahan nilai cukup dilakukan di satu tempat.
- Memudahkan audit dan maintenance.

---

## Severity

- **Tingkat:** **Informational** (tidak mempengaruhi keamanan atau fungsionalitas).
- **Laporan:** Cukup ditulis singkat, disertakan dalam laporan sebagai saran perbaikan kualitas kode.