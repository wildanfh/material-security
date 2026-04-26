# Recap: Re-entrancy – Dari Teori ke Pengujian

## Pendahuluan

Re-entrancy adalah salah satu vektor serangan paling umum dan merusak di dunia Smart Contract. Materi ini merangkum semua yang telah kita pelajari, mulai dari konsep dasar hingga cara pengetesan otomatis.

---

## 1. Ringkasan Konsep

**Apa itu Re-entrancy?**
Serangan yang terjadi ketika kontrak penyerang memanfaatkan kemampuan pemanggilan rekursif untuk memanggil kembali fungsi kontrak korban sebelum eksekusi sebelumnya selesai.

**Dampaknya:**
- Penarikan dana berulang kali (drain funds).
- Manipulasi state kontrak.
- Kerugian finansial yang masif (seperti Kasus The DAO).

---

## 2. Cara Deteksi & Identifikasi

1.  **Analisis Statis**: Tool seperti `Slither` seringkali dapat mendeteksi urutan operasi yang tidak aman secara otomatis.
2.  **Manual Review**: Mencari fungsi yang melakukan `.call` (eksternal interaction) sebelum memperbarui state internal (mapping saldo, status, dll).
3.  **Remix & Foundry**: Membuat simulasi serangan untuk membuktikan kerentanan secara empiris.

---

## 3. Strategi Mitigasi (Pencegahan)

Ada tiga cara utama yang telah kita bahas:

| Metode | Deskripsi |
| :--- | :--- |
| **Pola CEI** | **Checks-Effects-Interactions**. Selalu update state sebelum interaksi eksternal. |
| **Mutex Lock** | Mekanisme penguncian manual menggunakan variabel boolean (`locked`). |
| **ReentrancyGuard** | Menggunakan library standar industri dari **OpenZeppelin** (`nonReentrant`). |

---

## 4. Pengujian Otomatis (Foundry Test)

Sebagai auditor, kita harus bisa menulis test case untuk mereproduksi bug. Berikut adalah struktur dasar pengujian Re-entrancy:

```solidity
function test_reenter() public {
    // 1. Setup: User (Sucker) deposit dana ke korban
    vm.prank(victimUser);
    victimContract.deposit{value: 5 ether}();

    // 2. Aksi: Penyerang meluncurkan serangan
    vm.prank(attackerUser);
    attackerContract.attack{value: 1 ether}();

    // 3. Verifikasi: Saldo kontrak korban harusnya terkuras habis (0)
    assertEq(address(victimContract).balance, 0);
    
    // 4. Verifikasi Lanjutan: User asli tidak bisa menarik dananya lagi
    vm.prank(victimUser);
    vm.expectRevert();
    victimContract.withdrawBalance();
}
```

---

## 5. Pesan Penting

-   **Sejarah Berulang**: Kerentanan ini sudah ada sejak 2016, namun tetap memakan korban jutaan dolar setiap tahun.
-   **Kritis bagi Auditor**: Jangan pernah melewatkan fungsi yang melakukan pengiriman dana. Selalu asumsikan setiap interaksi eksternal bisa menjadi titik masuk serangan.
-   **Tooling**: Gunakan kombinasi Slither dan pengujian manual (Foundry/Remix) untuk hasil terbaik.
