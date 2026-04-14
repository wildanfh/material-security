# Rekap Temuan Audit PasswordStore

## Pendahuluan

Setelah melakukan proses *security review* pada kontrak `PasswordStore`, ditemukan tiga kerentanan utama yang meliputi **Access Control**, **dokumentasi tidak akurat**, dan **kesalahan fundamental dalam penyimpanan data sensitif di blockchain**.

---

## Vulnerability 1: Access Control pada `setPassword()`

### Kode yang Rentan
```solidity
function setPassword(string memory newPassword) external {
    s_password = newPassword;
    emit SetNetPassword();
}
```

Masalah

· Fungsi setPassword() dimaksudkan hanya dapat dipanggil oleh owner (berdasarkan NatSpec).
· Namun, implementasi tidak memiliki pemeriksaan akses sama sekali.
· Akibatnya, siapa pun dapat mengubah password kapan saja.

Perbaikan yang Direkomendasikan

```solidity
function setPassword(string memory newPassword) external {
    if (msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
    s_password = newPassword;
    emit SetNetPassword();
}
```

Dengan tambahan pemeriksaan ini, fungsi akan revert jika pemanggil bukan owner.

---

Vulnerability 2: NatSpec Tidak Akurat pada getPassword()

Kode yang Bermasalah

```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {
    // ...
}
```

Masalah

· NatSpec mencantumkan @param newPassword yang menyatakan fungsi menerima parameter newPassword.
· Namun, fungsi getPassword() tidak memiliki parameter.
· Ini adalah temuan tingkat informational – tidak mempengaruhi keamanan, tetapi mengurangi kualitas dokumentasi.

Rekomendasi

Hapus baris @param newPassword dari komentar NatSpec.

---

Vulnerability 3: Penyimpanan Password di On-Chain (Kritis)

Kode yang Rentan

```solidity
string private s_password; // Ini TIDAK aman!
```

Masalah Fundamental

· private di Solidity bukan berarti data tersembunyi.
· Semua data yang disimpan di blockchain bersifat publik – siapa pun dapat membaca slot storage menggunakan eth_getStorageAt.
· Password disimpan dalam bentuk plaintext (atau encoded) tanpa enkripsi.
· Ini melanggar invariant utama protokol: "Others should not be able to access the password".

Dampak

· Kerahasiaan password rusak total.
· Tidak ada perbaikan parsial – arsitektur harus diubah.

Rekomendasi

Jangan pernah menyimpan data sensitif (password, kunci API, dll) di on-chain. Blockchain tidak dirancang untuk kerahasiaan data. Gunakan penyimpanan off-chain (database terenkripsi, Vault) atau simpan hanya hash-nya.

---

Ringkasan Temuan

# Temuan Jenis Severity
1 setPassword() dapat dipanggil siapa saja Access Control High
2 NatSpec getPassword() mencantumkan parameter yang tidak ada Dokumentasi Informational
3 Password disimpan di on-chain (public storage) Business Logic Critical

---

Kesimpulan

· Protokol PasswordStore memiliki cacat desain fundamental yang membuatnya tidak aman untuk menyimpan rahasia.
· Dua kerentanan dapat diperbaiki dengan menambahkan access control dan memperbaiki dokumentasi.
· Kerentanan ketiga (storage publik) tidak dapat diperbaiki tanpa redesign total – karena blockchain bersifat transparan.
· Auditor harus memahami bahwa private di Solidity bukan fitur keamanan untuk kerahasiaan data.

"Private variables aren't hidden – all data is publicly accessible, breaking the protocol logic."