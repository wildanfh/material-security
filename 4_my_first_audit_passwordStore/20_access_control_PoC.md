
# Proof of Concept dan Mitigasi untuk Access Control

## Pendahuluan

Setelah menyusun **Title**, **Description**, dan **Impact** untuk temuan Access Control pada `setPassword()`, langkah selanjutnya adalah mengisi **Proof of Concept (PoC)** dan **Recommended Mitigation**. Bagian ini penting untuk **membuktikan** bahwa kerentanan benar-benar ada dan memberikan solusi konkret.

---

## Laporan Sejauh Ini (Sebelum PoC)

```markdown
### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}

**Impact:** Anyone can set/change the stored password, severely breaking the contract's intended functionality

**Proof of Concept:** (kosong)

**Recommended Mitigation:** (kosong)
```

---

## Proof of Concept (PoC) dengan Fuzz Test

Untuk membuktikan bahwa **siapa pun** (bukan hanya owner) dapat memanggil `setPassword()`, kita dapat menulis **fuzz test** di Foundry. Fuzz test akan menguji fungsi dengan berbagai alamat acak.

### Kode PoC

```solidity
function test_anyone_can_set_password(address randomAddress) public {
    vm.assume(randomAddress != owner);
    vm.startPrank(randomAddress);
    string memory expectedPassword = "myNewPassword";
    passwordStore.setPassword(expectedPassword);

    vm.startPrank(owner);
    string memory actualPassword = passwordStore.getPassword();
    assertEq(actualPassword, expectedPassword);
}
```

### Penjelasan Kode

| Baris | Fungsi |
|-------|--------|
| `vm.assume(randomAddress != owner)` | Abaikan jika alamat acak kebetulan sama dengan owner. |
| `vm.startPrank(randomAddress)` | Berpura-pura bahwa pemanggil adalah `randomAddress`. |
| `passwordStore.setPassword(...)` | Mencoba mengubah password sebagai `randomAddress`. |
| `vm.startPrank(owner)` | Kembali menjadi owner untuk membaca password. |
| `assertEq(...)` | Verifikasi bahwa password berhasil diubah oleh non-owner. |

### Hasil Eksekusi

Foundry akan menjalankan test ini sebanyak `runs` yang dikonfigurasi (misal 256 kali) dengan alamat acak berbeda. Jika semua lulus, terbukti bahwa **siapa pun dapat mengubah password**.

> *"We can see that through 256 runs, our fuzz test passed! So indeed any address was able to call our `setPassword` function!"*

---

## Recommended Mitigation

Mitigasi untuk kerentanan ini cukup jelas: **tambahkan access control** pada fungsi `setPassword()`.

### Kode Mitigasi

```solidity
if (msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```

### Penerapan dalam Fungsi

```solidity
function setPassword(string memory newPassword) external {
    if (msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
    s_password = newPassword;
    emit SetNetPassword();
}
```

---

## Laporan Akhir (Setelah PoC dan Mitigasi)

<details closed>
<summary>Access Control Report (Lengkap)</summary>

```markdown
### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

```solidity
function setPassword(string memory newPassword) external {
    // @Audit - There are no Access Controls.
    s_password = newPassword;
    emit SetNewPassword();
}
```

**Impact:** Anyone can set/change the stored password, severely breaking the contract's intended functionality

**Proof of Concept:** Add the following to the `PasswordStore.t.sol` test file:

```solidity
function test_anyone_can_set_password(address randomAddress) public {
    vm.assume(randomAddress != owner);
    vm.startPrank(randomAddress);
    string memory expectedPassword = "myNewPassword";
    passwordStore.setPassword(expectedPassword);

    vm.startPrank(owner);
    string memory actualPassword = passwordStore.getPassword();
    assertEq(actualPassword, expectedPassword);
}
```

**Recommended Mitigation:** Add an access control conditional to `PasswordStore::setPassword`.

```solidity
if (msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```
```

</details>

---

## Pro-Tip: Menggunakan Dropdown untuk Kode Panjang

Dalam laporan profesional, Anda dapat menyembunyikan blok kode yang panjang menggunakan sintaks HTML `<details>`.

### Contoh Sintaks

```html
<details>
<summary>Code</summary>

```solidity
// kode di sini
```

</details>
```

Ini membuat laporan lebih rapi dan mudah dibaca.

---

## Kesimpulan

- **Proof of Concept** untuk Access Control dapat berupa **fuzz test** yang membuktikan non-owner dapat memanggil fungsi.
- **Recommended Mitigation** adalah menambahkan pemeriksaan `msg.sender != s_owner` dengan `revert`.
- PoC yang kuat **meyakinkan protokol** bahwa kerentanan valid dan serius.
- Gunakan **dropdown** untuk menyembunyikan blok kode yang panjang agar laporan tetap bersih.

> *"Repetition is what will strengthen these skills and make writing these reports second nature."*