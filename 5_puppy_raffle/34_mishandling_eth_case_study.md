# Kasus Eksploitasi: Force Feeding ETH via selfdestruct

Materi ini memberikan demonstrasi praktis bagaimana penyerang dapat merusak logika akuntansi Smart Contract dengan mengirimkan ETH secara paksa.

### 1. Kontrak yang Rentan (`SelfDestructMe.sol`)
Kontrak ini mencoba mengelola deposit pengguna dan memastikan integritas saldo dengan menggunakan `assert`.

```javascript
contract SelfDestructMe {
    uint256 public totalDeposits;
    mapping(address => uint256) public deposits;

    function deposit() external payable {
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
    }

    function withdraw() external {
        // VULNERABILITY: Strict equality check
        assert(address(this).balance == totalDeposits); 

        uint256 amount = deposits[msg.sender];
        totalDeposits -= amount;
        deposits[msg.sender] = 0;

        payable(msg.sender).transfer(amount);
    }
}
```

### 2. Mekanisme Serangan (The Attack)
Penyerang membuat kontrak penyerang yang memiliki fungsi `selfdestruct`. Fungsi ini akan menghapus kode kontrak dari blockchain dan mengirimkan seluruh saldonya ke alamat target secara paksa.

```javascript
contract AttackSelfDestructMe {
    constructor(address target) payable {
        selfdestruct(payable(target));
    }
}
```

**Langkah-langkah Serangan:**
1. Pengguna jujur melakukan deposit (misal: 1 ETH). `totalDeposits` = 1, `address(this).balance` = 1.
2. Penyerang men-deploy `AttackSelfDestructMe` dengan nilai 0.1 ETH dan menargetkan alamat `SelfDestructMe`.
3. Setelah serangan, `address(this).balance` milik `SelfDestructMe` menjadi **1.1 ETH**, namun `totalDeposits` tetap **1 ETH**.

### 3. Dampak: Denial of Service (DoS)
Ketika pengguna mencoba memanggil `withdraw()`:
*   Fungsi menjalankan `assert(address(this).balance == totalDeposits)`.
*   Karena `1.1 ETH != 1 ETH`, maka `assert` akan gagal (revert).
*   **Hasilnya:** Dana seluruh pengguna terjebak di dalam kontrak selamanya karena fungsi withdraw tidak akan pernah bisa berhasil lagi.

### 4. Pelajaran Penting (Key Takeaways)
*   **Don't Trust the Balance:** Jangan pernah berasumsi bahwa Anda memiliki kendali penuh atas siapa yang bisa mengirim ETH ke kontrak Anda.
*   **Strict Equality is Dangerous:** Menggunakan `==` untuk membandingkan `address(this).balance` dengan variabel internal hampir selalu merupakan celah keamanan.
*   **Invariants:** Jika Anda harus melakukan pengecekan, gunakan operator `>=` agar dana tambahan (yang dikirim secara paksa) tidak merusak fungsionalitas utama.
*   **Internal Accounting:** Selalu andalkan variabel state internal (seperti `totalDeposits`) untuk logika penarikan dana, bukan saldo global kontrak.

### 5. Rekomendasi Perbaikan
Ubah pengecekan saldo menjadi lebih fleksibel:
```javascript
// Lebih aman, meskipun masih ada risiko jika logika bergantung pada saldo yang tepat
require(address(this).balance >= totalDeposits, "Inconsistent balance");
```
Atau lebih baik lagi, hapus ketergantungan logika pada `address(this).balance` sama sekali untuk fungsi penarikan.
