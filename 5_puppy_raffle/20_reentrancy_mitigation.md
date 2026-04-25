# Mitigasi Re-entrancy: 3 Cara Melindungi Kontrak

## Pendahuluan

Re-entrancy adalah ancaman serius, namun cara memperbaikinya sebenarnya cukup sederhana jika kita memahami urutan operasi. Ada tiga metode utama untuk melindungi Smart Contract dari serangan ini.

---

## 1. Pola CEI (Checks, Effects, Interactions)

Ini adalah *best practice* standar dalam penulisan fungsi Smart Contract. Pola ini mengatur urutan operasi agar interaksi eksternal selalu menjadi langkah terakhir.

### Urutan Operasi CEI:
1.  **Checks**: Melakukan validasi input, kondisi, dan persyaratan (`require`, `if`).
2.  **Effects**: Melakukan pembaruan state internal kontrak (update saldo, status, dll).
3.  **Interactions**: Berinteraksi dengan kontrak eksternal atau mengirim ETH.

### Contoh Implementasi:
```javascript
function withdrawBalance() public {
    // 1. Checks
    uint256 balance = userBalance[msg.sender];
    require(balance > 0, "Saldo nol");

    // 2. Effects
    userBalance[msg.sender] = 0;

    // 3. Interactions
    (bool success,) = msg.sender.call{value: balance}("");
    if (!success) revert();
}
```
**Mengapa ini berhasil?** Karena saldo sudah di-nol-kan di tahap **Effects**, jika penyerang mencoba masuk kembali di tahap **Interactions**, pengecekan saldo mereka akan gagal atau bernilai nol.

---

## 2. Mekanisme Mutex Lock (Manual)

Dalam ilmu komputer, *mutex* (mutual exclusion) adalah mekanisme penguncian agar suatu fungsi tidak bisa dijalankan oleh lebih dari satu eksekusi pada saat yang bersamaan.

### Contoh Implementasi:
```javascript
bool private locked = false;

function withdrawBalance() public {
    require(!locked, "Fungsi sedang dikunci");
    locked = true; // Kunci dipasang

    // --- Logika Fungsi ---
    uint256 balance = userBalance[msg.sender];
    userBalance[msg.sender] = 0;
    (bool success,) = msg.sender.call{value: balance}("");
    if (!success) revert();
    // ---------------------

    locked = false; // Kunci dilepas setelah selesai
}
```
Jika penyerang mencoba memanggil fungsi ini lagi sebelum eksekusi pertama selesai, `require(!locked)` akan memicu revert.

---

## 3. OpenZeppelin ReentrancyGuard

Ini adalah cara yang paling bersih dan profesional karena menggunakan library yang sudah teruji secara industri.

### Cara Penggunaan:
1.  Import library `ReentrancyGuard`.
2.  Inherit kontrak Anda dari `ReentrancyGuard`.
3.  Gunakan modifier `nonReentrant` pada fungsi yang ingin dilindungi.

```javascript
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract MyContract is ReentrancyGuard {
    function withdrawBalance() public nonReentrant {
        uint256 balance = userBalance[msg.sender];
        userBalance[msg.sender] = 0;
        (bool success,) = msg.sender.call{value: balance}("");
        if (!success) revert();
    }
}
```
Di balik layar, `nonReentrant` bekerja mirip dengan mekanisme Mutex Lock, namun lebih hemat gas dan kodenya lebih rapi.

---

## Ringkasan Mitigasi

| Metode | Kelebihan | Rekomendasi |
| :--- | :--- | :--- |
| **Pola CEI** | Hemat gas, tidak butuh state tambahan. | **Wajib** dilakukan di setiap fungsi. |
| **Mutex Lock** | Memberikan perlindungan ekstra. | Gunakan jika CEI sulit diterapkan karena logika yang kompleks. |
| **ReentrancyGuard**| Standar industri, sangat aman, kode bersih. | **Sangat Disarankan** untuk fungsi sensitif yang mengirim dana. |
