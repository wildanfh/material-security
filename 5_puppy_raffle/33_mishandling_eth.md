# Mishandling ETH & Strict Balance Equality

Materi ini membahas jebakan umum dalam manajemen saldo ETH pada Smart Contract, khususnya terkait penggunaan `address(this).balance` untuk logika akuntansi (accounting).

### 1. Masalah pada Kode `PuppyRaffle`
Dalam fungsi `withdrawFees`, terdapat pengecekan berikut:
```javascript
require(address(this).balance == uint256(totalFees), "PuppyRaffle: there are currently players active!");
```
**Tujuan Pengembang:** Memastikan bahwa kontrak hanya mengizinkan penarikan biaya (fees) jika saldo kontrak tepat sama dengan total biaya yang terkumpul. Mereka berasumsi saldo hanya akan berisi dana dari pendaftar aktif atau biaya yang belum ditarik.

### 2. Asumsi yang Salah (The False Assumption)
Pengembang sering berasumsi bahwa jika sebuah kontrak **tidak memiliki** fungsi `receive()` atau `fallback()`, maka tidak ada cara bagi ETH untuk masuk ke dalam kontrak tersebut selain melalui fungsi-fungsi yang telah ditentukan (seperti fungsi `enterRaffle`).

### 3. Hasil Pengujian Awal (False Sense of Security)
Ketika dilakukan pengujian untuk mengirim ETH secara paksa melalui transaksi biasa:
```javascript
function testCantSendMoneyToRaffle() public {
    address sendAddy = makeAddr("sender");
    vm.deal(sendAddy, 1 ether);
    vm.expectRevert();
    vm.prank(sendAddy);
    (bool success, ) = payable(address(puppyRaffle)).call{value: 1 ether}("");
    require(success);
}
```
Tes ini **berhasil (PASS)**, yang berarti pengiriman ETH biasa memang ditolak oleh kontrak. Hal ini sering membuat auditor atau pengembang pemula merasa bahwa kode tersebut sudah aman.

### 4. Mengapa Ini Berbahaya?
Meskipun pengiriman ETH biasa ditolak, ada beberapa cara "paksa" untuk mengirim ETH ke kontrak yang tidak bisa dicegah oleh kode Solidity:
*   **`selfdestruct`**: Sebuah kontrak lain dapat memanggil `selfdestruct(targetAddress)`, yang akan mengirimkan seluruh sisa saldonya ke `targetAddress` secara paksa tanpa memicu fungsi apa pun di target.
*   **Pre-funding**: Kontrak dapat dikirimi ETH bahkan sebelum ia di-deploy (menggunakan perhitungan alamat kontrak di masa depan).
*   **Block Rewards/Coinbase**: Penambang (miners/validators) bisa mengarahkan reward blok ke alamat kontrak.

### 5. Kesimpulan
Mengandalkan `address(this).balance` untuk pengecekan saldo yang ketat (`==`) adalah anti-pattern yang berbahaya. Jika seseorang mengirimkan 1 wei saja secara paksa melalui `selfdestruct`, maka kondisi `address(this).balance == totalFees` akan menjadi **false** selamanya, yang mengakibatkan fungsi penarikan biaya (atau fungsi krusial lainnya) terkunci secara permanen (Denial of Service).
