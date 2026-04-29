# Smart Contract Wallet Raffle Winners tanpa `receive` atau `fallback` dapat menghalangi dimulainya kontes baru

## Kode Rentan

```solidity
//@Audit: winner wouldn't get their money if their fallback was messed up!
```

## Root Cause

Fungsi `PuppyRaffle::selectWinner` bertanggung jawab untuk mengatur ulang lotre setelah pemenang dipilih. Jika pemenang adalah smart contract wallet yang **menolak pembayaran** (tidak memiliki fungsi `receive` atau `fallback`), maka transaksi `selectWinner` akan revert. Akibatnya, lotre tidak bisa diatur ulang dan kontes baru tidak dapat dimulai.

## Severity

| Faktor | Penilaian | Alasan |
|--------|-----------|--------|
| **Impact** | Medium | Membuang gas, mengganggu fungsionalitas protokol (selectWinner terus revert). |
| **Likelihood** | Low | Dampak parah hanya terjadi jika banyak pengguna dan pemenang adalah kontrak tanpa fallback. |

**Severity:** **Medium** (`[M-4]`)

## Dampak

- Fungsi `selectWinner` bisa revert berkali-kali, membuat sangat sulit untuk mereset lotre dan memulai kontes baru.
- Pemenang sebenarnya tidak bisa mendapatkan hadiah mereka.
- Orang lain (atau tidak ada) yang akhirnya memenangkan dana tersebut.

## Proof of Concept

1. 10 smart contract wallet masuk ke lotre, masing-masing **tanpa** fungsi `receive` atau `fallback`.
2. Lotre berakhir.
3. Fungsi `selectWinner` akan gagal (revert) karena mencoba mengirim ETH ke kontrak yang menolak pembayaran, meskipun lotre sudah selesai.

## Rekomendasi Mitigasi

Dua opsi utama:

1. **Larang smart contract wallet sebagai peserta** (tidak direkomendasikan) – karena akan mengunci multisignature wallet dan pengguna legitimate lainnya.
2. **Gunakan pola *Pull over Push*** – buat mapping `address => uint256 payout` sehingga pemenang dapat menarik dana mereka sendiri (claim). Ini adalah rekomendasi yang lebih baik.

Contoh pendekatan *pull*:

```solidity
mapping(address => uint256) public winnings;

function selectWinner() external {
    // ... hitung winner dan winnings
    winnings[winner] = winnings;
    // reset state lotre
}

function claimPrize() external {
    uint256 amount = winnings[msg.sender];
    require(amount > 0, "No winnings");
    winnings[msg.sender] = 0;
    payable(msg.sender).transfer(amount);
}
```

Dengan pola ini, protokol tidak lagi *push* dana ke pemenang, tetapi pemenang yang *pull* dana mereka sendiri. Ini menghindari masalah kontrak yang menolak pembayaran.