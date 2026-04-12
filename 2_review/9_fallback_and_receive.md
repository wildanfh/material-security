## Latar Belakang

Secara default, smart contract Solidity **menolak** setiap pengiriman Ether (ETH) yang masuk. Agar kontrak dapat menerima ETH, kita perlu mengimplementasikan fungsi `fallback` dan/atau `receive`.

---

## Fungsi Receive dan Fallback

Kedua fungsi ini bersifat `external payable` – artinya dapat menerima ETH.

### Receive Function

- Dipanggil ketika **calldata kosong** (tidak ada data yang dikirim bersama ETH).
- Sederhananya: saat Anda mengirim ETH ke kontrak hanya dengan transfer tanpa pesan tambahan.

```solidity
receive() external payable {
    // kode untuk menangani penerimaan ETH
}
```

### Fallback Function

- Dipanggil ketika **calldata tidak kosong** (ada data yang dikirim) **ATAU** ketika fungsi `receive` tidak ada.
- Juga dapat dipanggil jika `receive` ada tetapi calldata tidak kosong.

```solidity
fallback() external payable {
    // kode untuk menangani data atau fallback
}
```

---

## Alur Logika Penerimaan ETH

Berikut alur yang terjadi ketika ETH dikirim ke kontrak:

1. **Apakah calldata kosong?**
   - Jika **YA** → cek apakah ada `receive()`?
     - Ada → jalankan `receive()`
     - Tidak ada → cek apakah `fallback()` payable?
       - Ya → jalankan `fallback()`
       - Tidak → transaksi ditolak
   - Jika **TIDAK** (ada data) → langsung ke `fallback()` jika ada dan payable, jika tidak → ditolak.

> Diagram dari video memperjelas alur ini.

---

## Aturan Penting

- `fallback` **boleh** tidak `payable`, tapi jika tidak `payable` dan `receive` juga tidak ada, maka kontrak **tidak bisa menerima ETH**.
- Jika kontrak **tidak memiliki** `receive` maupun `fallback` yang `payable`, Solidity secara otomatis akan menolak semua pengiriman ETH ke kontrak tersebut.
- Pengecualian: `selfdestruct` (akan dibahas lain waktu) dapat memaksa pengiriman ETH meskipun tanpa fungsi-fungsi ini.

---

## Contoh Minimal agar Kontrak Bisa Menerima ETH

```solidity
contract MyContract {
    receive() external payable {}
    // atau fallback() external payable {}
}
```

Cukup salah satu yang `payable`, kontrak sudah bisa menerima ETH.

---

## Ringkasan Perbedaan

| Kondisi | Fungsi yang Dipanggil |
|---------|----------------------|
| Kirim ETH + calldata kosong, ada `receive` | `receive` |
| Kirim ETH + calldata kosong, tidak ada `receive` | `fallback` (jika payable) |
| Kirim ETH + calldata tidak kosong | `fallback` (jika payable) |
