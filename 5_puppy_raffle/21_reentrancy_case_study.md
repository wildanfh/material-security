# Studi Kasus Re-entrancy: The DAO & Ancaman Industri

## Pendahuluan

Re-entrancy bukan sekadar bug biasa; ini adalah "hantu" yang terus menghantui ekosistem Web3 sejak awal berdirinya. Meskipun sudah dikenal luas dan mudah dideteksi oleh tool seperti Slither, kerentanan ini masih menyebabkan kerugian jutaan dolar setiap tahunnya.

---

## 1. Mengapa Re-entrancy Begitu Berbahaya?

-   **Sudah Diketahui Sejak 2016**: Kita sudah tahu cara kerjanya selama bertahun-tahun, namun pengembang masih sering melakukan kesalahan yang sama.
-   **Mudah Dideteksi**: Tool statis seperti Slither hampir selalu bisa menemukannya.
-   **Dampak Finansial Masif**: Masih banyak protokol yang kehilangan jutaan dolar per tahun akibat variasi serangan ini (termasuk jenis baru seperti *Read-only Re-entrancy*).

---

## 2. Kasus Terbesar: The DAO (2016)

**The DAO** adalah salah satu protokol paling ambisius dalam sejarah Web3. Pada Mei 2016, protokol ini memegang sekitar **14% dari total pasokan ETH** yang beredar pada saat itu.

### Masalah pada Kode `splitDAO`
Fungsi `splitDAO` memiliki kesalahan fatal dalam urutan operasinya:

```javascript
// Kode yang disederhanakan dari The DAO
function splitDAO(uint _proposalID, address _newCurator) {
    // ... pengecekan awal ...

    // ❌ EXTERNAL CALL: Mengirim dana ke DAO baru
    if (p.splitData[0].newDAO.createTokenProxy.value(fundsToBeMoved)(msg.sender) == false)
        throw;

    // ❌ STATE CHANGE: Baru dilakukan di akhir setelah pengiriman dana
    Transfer(msg.sender, 0, balances[msg.sender]);
    withdrawRewardFor(msg.sender);
    totalSupply -= balances[msg.sender];
    balances[msg.sender] = 0; // Terlambat!
    return true;
}
```

### Masalah pada Kode `withdrawRewardFor`
Kesalahan yang sama terjadi pada fungsi reward:

```javascript
function withdrawRewardFor(address _account) internal {
    uint reward = ...;
    
    // ❌ EXTERNAL CALL: Membayar reward
    if (!rewardAccount.payOut(_account, reward))
        throw;

    // ❌ STATE CHANGE: Mengupdate status pembayaran
    paidOut[_account] += reward; // Terlambat!
}
```

---

## 3. Dampak Terhadap Ekosistem

Serangan terhadap The DAO pada Juni 2016 mengakibatkan:
1.  **3.8 Juta ETH Tercuri**: Nilai yang sangat fantastis baik dulu maupun sekarang.
2.  **Ethereum Hard Fork**: Karena besarnya dana yang hilang (14% dari total suplai), komunitas memutuskan untuk melakukan *hard fork* guna mengembalikan dana tersebut.
3.  **Lahirnya Ethereum Classic (ETC)**: Sebagian komunitas menolak *hard fork* dengan prinsip "Code is Law", sehingga jaringan terpecah menjadi Ethereum (ETH) yang kita kenal sekarang dan Ethereum Classic (ETC).

---

## 4. Kesimpulan

Kasus The DAO mengajarkan kita bahwa **urutan kode sangatlah penting**. Satu baris kode yang diletakkan di tempat yang salah (sebelum update state) dapat menghancurkan seluruh protokol dan mengubah sejarah blockchain.

**Pesan untuk Auditor:**
Jangan pernah mengabaikan peringatan Re-entrancy dari Slither, sekecil apa pun itu. Selalu pastikan protokol menerapkan pola **CEI (Checks-Effects-Interactions)** tanpa pengecualian.

---
*Referensi tambahan: [Katalog Serangan Re-entrancy oleh Pascal Caversaccio](https://github.com/pcaversaccio/reentrancy-attacks)*
