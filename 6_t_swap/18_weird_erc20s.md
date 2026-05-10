# Weird ERC20s – Token dengan Perilaku Tidak Standar

## Pendahuluan

**Weird ERC20** adalah token yang mengimplementasikan standar ERC20 tetapi dengan perilaku **tidak biasa** atau **menyimpang** dari spesifikasi standar. Perilaku ini dapat menyebabkan kerentanan serius pada protokol DeFi yang mengintegrasikan token tersebut tanpa antisipasi yang tepat.

> Sumber daya penting: [**Weird ERC20 Repo by d-xo**](https://github.com/d-xo/weird-erc20) – koleksi contoh token dengan perilaku aneh.

---

## Contoh Weird ERC20 yang Ditemukan dalam Praktik

### 1. Fee‑on‑Transfer (Token dengan Biaya Transfer)

Token memotong sebagian kecil nilai setiap kali transfer dilakukan. Contoh: `YieldERC20` dalam latihan kita memotong 10% setiap 10 transaksi.

**Risiko:** Protokol yang mengharapkan jumlah yang dikirim sama dengan yang diterima akan mengalami ketidakcocokan saldo, gagal withdraw, atau kekurangan dana.

### 2. Reentrancy pada Token

Beberapa token mengizinkan panggilan ulang (reentrancy) saat melakukan transfer. Ini dapat membuka celah serangan *reentrancy* pada protokol yang tidak menggunakan `nonReentrant` modifier.

### 3. Missing Return Values (Tidak Mengembalikan Nilai Boolean)

Beberapa token (misal USDT versi lama) tidak mengembalikan `bool` pada fungsi `transfer`/`transferFrom`. Jika protokol mengasumsikan selalu ada return value, transaksi dapat gagal atau salah diinterpretasi.

### 4. Upgradeable ERC20 (Token yang Dapat Ditingkatkan)

Token yang menggunakan pola *proxy* (contoh: USDC) dapat diubah logikanya di masa depan. Perubahan mendadak dapat merusak protokol yang bergantung pada perilaku lama.

### 5. Rebasing Token (Token dengan Suplai Berubah)

Saldo pengguna berubah secara otomatis (biasanya setiap periode) tanpa adanya transfer. Contoh: stETH, AMPL. Protokol yang menyimpan jumlah token sebagai representasi nilai dapat kehilangan akurasi.

### 6. Block Lists (Daftar Blokir)

Token tertentu (misal USDC, USDT) memiliki kemampuan untuk memblokir alamat tertentu. Jika protokol menerima deposit dari alamat yang kemudian diblokir, dana dapat terkunci.

### 7. Allowance Race Conditions

Beberapa token tidak mengikuti standar EIP-20 untuk perubahan allowance, menyebabkan *race condition* saat dua transaksi membatalkan approval secara bersamaan.

---

## Mengapa Weird ERC20 Berbahaya?

- **Asumsi yang salah** – Protokol sering mengasumsikan token berperilaku seperti ERC20 standar (misal tidak ada fee, selalu return true, saldo tidak berubah sendiri).
- **Eksploitasi** – Penyerang dapat memanfaatkan token dengan fee atau rebasing untuk menguras dana, memblokir protokol, atau merusak invariant.
- **Integrasi yang tidak kompatibel** – Token yang tidak mengembalikan boolean dapat menyebabkan transaksi revert di protokol yang menggunakan `require(token.transfer(...))`.

---

## Rekomendasi Perlindungan

1. **Integrasi token secara hati-hati** – Gunakan [**Token Integration Checklist by Trail of Bits**](https://secure-contracts.com/development-guidelines/token_integration.html) sebagai panduan.
2. **Gunakan library yang aman** – Misal: OpenZeppelin's `SafeERC20` yang menangani token dengan return value tidak ada.
3. **Fuzzing invariant dengan handler** – Seperti yang kita praktikkan, handler dapat menyimulasikan berbagai perilaku token aneh.
4. **Batasi token yang didukung** – Jika memungkinkan, buat whitelist token yang perilakunya sudah terverifikasi.
5. **Uji dengan token mock yang aneh** – Buat mock token yang meniru fee‑on‑transfer, rebasing, dll. untuk menguji ketahanan protokol.

> Sumber daya tambahan: [**secure-contracts.com**](https://secure-contracts.com/index.html) – panduan keamanan smart contract yang sangat berharga.

---

## Kesimpulan

- **Weird ERC20** adalah kelas token yang tidak sepenuhnya mengikuti standar ERC20.
- Contoh umum: fee‑on‑transfer, rebasing, missing return, upgradeable, blocklist.
- Protokol DeFi harus secara eksplisit **memperlakukan setiap token eksternal sebagai potensi ancaman**.
- Pengujian dengan handler‑based fuzzing adalah cara efektif untuk menemukan kerentanan yang berasal dari token aneh.