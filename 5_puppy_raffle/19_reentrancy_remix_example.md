# Praktik Re-entrancy: Remix & Solusi Sederhana

## Pendahuluan

Materi ini memberikan demonstrasi praktis tentang bagaimana serangan Re-entrancy bekerja di lingkungan Remix dan bagaimana cara memperbaikinya dengan sangat sederhana namun efektif.

---

## 1. Inti Masalah: Urutan State Update

Kesalahan fatal dalam kerentanan ini adalah melakukan pembaruan saldo (state change) di **akhir** fungsi, setelah pengiriman dana dilakukan.

### Kode Rentan (Vulnerable)
```javascript
function withdrawBalance() public {
    uint256 balance = userBalance[msg.sender];
    
    // ❌ EXTERNAL CALL DILAKUKAN SEBELUM STATE UPDATE
    (bool success,) = msg.sender.call{value: balance}("");
    if (!success) revert();

    // ❌ STATE UPDATE DI AKHIR
    userBalance[msg.sender] = 0;
}
```

---

## 2. Solusi: Pola CEI (Checks-Effects-Interactions)

Pencegahan Re-entrancy sebenarnya sangat mudah: **Update state Anda sebelum melakukan external call.**

### Kode Aman (Fixed)
```javascript
function withdrawBalance() public {
    uint256 balance = userBalance[msg.sender];

    // ✅ STATE UPDATE DILAKUKAN DULU (EFFECTS)
    userBalance[msg.sender] = 0;

    // ✅ EXTERNAL CALL DILAKUKAN TERAKHIR (INTERACTIONS)
    (bool success,) = msg.sender.call{value: balance}("");
    if (!success) revert();
}
```
**Mengapa ini aman?**
Ketika penyerang mencoba memanggil `withdrawBalance()` lagi melalui fungsi `receive()`, saldo mereka (`userBalance[msg.sender]`) sudah bernilai **0**. Maka, pengiriman dana berikutnya akan bernilai 0 dan serangan berhenti.

---

## 3. Simulasi Serangan di Remix (Langkah-langkah)

Untuk melihat dampak nyata dari serangan ini, kita menggunakan dua kontrak: `ReentrancyVictim` dan `ReentrancyAttacker`.

### Tahap Persiapan:
1.  **Deploy Korban**: Deploy `ReentrancyVictim`.
2.  **Deploy Penyerang**: Deploy `ReentrancyAttacker` dengan memasukkan alamat kontrak korban di constructor.
3.  **Deposit Dana (Korban)**: Gunakan akun lain (akun "Sucker") untuk melakukan `deposit` sebesar 5 ETH ke kontrak korban. (Saldo Korban = 5 ETH).

### Tahap Serangan:
1.  **Eksekusi**: Pindah ke akun penyerang. Panggil fungsi `attack()` dengan mengirimkan minimal 1 ETH.
2.  **Mekanisme**:
    - Kontrak penyerang deposit 1 ETH ke korban.
    - Kontrak penyerang memanggil `withdrawBalance()`.
    - Korban mengirim 1 ETH balik ke penyerang -> memicu fungsi `receive()` penyerang.
    - Fungsi `receive()` memanggil kembali `withdrawBalance()` milik korban.
    - **Loop terjadi** hingga seluruh saldo 5 ETH milik korban terkuras habis.

### Hasil Akhir:
-   **Saldo Korban**: 0 ETH.
-   **Saldo Penyerang**: Meningkat drastis (Dana awal + Seluruh dana korban).

---

## 4. Kesimpulan

Re-entrancy bukan sekadar teori, melainkan ancaman nyata yang bisa menguras seluruh isi protokol dalam sekejap.

**Key Takeaways:**
-   Selalu gunakan pola **Checks-Effects-Interactions (CEI)**.
-   Pikirkan setiap `.call` sebagai potensi titik masuk bagi kode eksternal yang tidak dipercaya.
-   Gunakan tool statis seperti Slither untuk mendeteksi urutan operasi yang mencurigakan.
