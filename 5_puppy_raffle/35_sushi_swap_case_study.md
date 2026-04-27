# Kasus Eksploitasi: Sushi Swap (Batch Delegatecall)

Studi kasus ini membahas salah satu bug paling terkenal dalam kategori **Mishandling of ETH**, yang ditemukan oleh peneliti keamanan **samczsun** pada protokol Sushi Swap.

### 1. Masalah pada Fungsi `batch`
Sushi Swap memiliki fungsi `batch` yang memungkinkan pengguna untuk menjalankan banyak panggilan fungsi (multicall) dalam satu transaksi tunggal untuk menghemat gas.

```javascript
function batch(bytes[] calldata calls, bool revertOnFail) external payable returns (bool[] memory successes, bytes[] memory results) {
    successes = new bool[](calls.length);
    results = new bytes[](calls.length);
    for (uint256 i = 0; i < calls.length; i++) {
        // VULNERABILITY: delegatecall inside a loop
        (bool success, bytes memory result) = address(this).delegatecall(calls[i]);
        require(success || !revertOnFail, _getRevertMsg(result));
        successes[i] = success;
        results[i] = result;
    }
}
```

### 2. Akar Masalah: Persistensi `msg.value`
Masalah utama terjadi karena karakteristik `delegatecall` dalam sebuah loop:
*   `delegatecall` mempertahankan konteks pemanggil asli, termasuk `msg.sender` dan **`msg.value`**.
*   Jika fungsi `batch` dipanggil dengan `msg.value` sebesar 1 ETH, maka setiap fungsi yang dipanggil melalui `delegatecall` di dalam loop tersebut akan melihat `msg.value` yang sama (1 ETH).

### 3. Skenario Eksploitasi (The Exploit)
Bayangkan sebuah fungsi `deposit()` yang menambahkan saldo pengguna berdasarkan `msg.value`.
1.  Penyerang memanggil fungsi `batch`.
2.  Di dalam `calls`, penyerang memasukkan 100 instruksi untuk memanggil `deposit()`.
3.  Penyerang mengirimkan transaksi dengan hanya **1 ETH**.
4.  Loop akan berjalan 100 kali. Setiap kali `delegatecall` memanggil `deposit()`, kontrak akan memeriksa `msg.value` dan melihat angka **1 ETH**.
5.  **Hasilnya:** Penyerang mendapatkan saldo **100 ETH** di dalam protokol, padahal mereka hanya mengirimkan **1 ETH**.

### 4. Mengapa Ini Disebut Mishandling of ETH?
Ini adalah kesalahan logika dalam menangani dana (ETH). Pengembang lupa bahwa `msg.value` adalah nilai global untuk seluruh transaksi, bukan nilai yang "berkurang" setiap kali digunakan dalam `delegatecall`.

### 5. Pelajaran Penting (Key Takeaways)
*   **Be Careful with `delegatecall`:** `delegatecall` sangat kuat tetapi berbahaya karena membawa seluruh status pemanggil.
*   **Avoid `msg.value` in Loops:** Jangan pernah menggunakan `msg.value` di dalam loop jika fungsi yang dipanggil menggunakan nilai tersebut untuk akuntansi internal (seperti menambah saldo).
*   **The "Two Rights Might Make a Wrong":** Fungsi `batch` tampak benar, fungsi `deposit` tampak benar, tetapi ketika digabungkan dengan cara yang salah, mereka menciptakan celah keamanan yang fatal.

### 6. Rekomendasi Perbaikan
Untuk menghindari masalah ini, protokol biasanya menggunakan pengecekan tambahan atau menggunakan kontrak pembantu (helper) yang melacak berapa banyak ETH yang telah digunakan selama eksekusi batch, untuk memastikan tidak melebihi `msg.value` yang dikirimkan.
