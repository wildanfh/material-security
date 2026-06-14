# Ringkasan Materi: Payback or Revert – Mekanisme Flash Loan

## Pendahuluan

Flash loan memungkinkan pengguna biasa memanfaatkan peluang finansial berskala besar tanpa perlu modal sendiri. Ini dimungkinkan oleh sifat **programmable blockchain** di mana protokol dapat secara otomatis menjamin pinjaman akan dilunasi.

---

## Prinsip Dasar: Harus Kembali atau Batal (Revert)

- Dalam keuangan tradisional, pinjaman memerlukan **jaminan (collateral)**.
- Di Web3, protokol menggunakan **logika dalam smart contract** untuk memastikan pinjaman dilunasi di **transaksi yang sama**.
- Jika syarat tidak terpenuhi, seluruh transaksi akan **revert** (batal) – seolah-olah tidak pernah terjadi.

---

## Contoh Kode Minimal Flash Loan

```solidity
uint256 startingBalance = IERC20(token).balanceOf(address(this));
assetToken.transferUnderlyingTo(receiverAddress, amount);

// callback function di sini (pengguna bebas melakukan apa pun)

uint256 endingBalance = token.balanceOf(address(this));
if (endingBalance < startingBalance + fee) {
    revert();
}
```

### Penjelasan Alur:
1. Catat saldo awal token di protokol (`startingBalance`).
2. Transfer jumlah pinjaman ke penerima (`receiverAddress`).
3. **Callback** – di sinilah peminjam dapat melakukan aksi apapun (misal: arbitrase, swap, likuidasi).
4. Setelah callback selesai, periksa saldo akhir (`endingBalance`).
5. Jika `endingBalance` **kurang dari** `startingBalance + fee`, transaksi **dibatalkan (revert)**.
6. Jika cukup, transaksi berhasil – protokol menerima kembali pinjaman + biaya.

---

## Mengapa Ini Powerful?

- **Tanpa agunan** – karena pembayaran dipaksakan secara atomik.
- **Akses untuk semua** – tidak hanya whales, pengguna biasa juga bisa meminjam dana besar untuk arbitrase atau strategi lainnya.
- **Keamanan protokol** – tidak ada risiko kredit macet karena pinjaman harus lunas dalam satu transaksi.

![pay-back-or-revert1](/security-section-6/5-pay-back-or-revert/pay-back-or-revert1.png)

> *"If these checks don't pass, the transaction will revert, back to its initial state – as though the loan never took place."*

---

## Kesimpulan

- Mekanisme **payback or revert** adalah inti dari flash loan.
- Peminjam bebas menggunakan dana di antara dua titik pemeriksaan saldo.
- Jika gagal melunasi, semua perubahan yang dilakukan akan dibatalkan.
- Ini membuka peluang DeFi yang lebih demokratis dan efisien.