# Testing dan Coverage dalam Security Review

## Pendahuluan

Sebagai security researcher, tugas utama adalah melakukan apa pun yang diperlukan untuk membuat protokol lebih aman. Setelah memeriksa seluruh lingkup `PasswordStore`, masih ada nilai tambah dalam memperluas pengintaian (*recon*) ke **test suite** protokol.

Test suite seharusnya menjadi **ekspektasi** bagi protokol yang serius tentang keamanan. Menjamin cakupan tes (*test coverage*) yang memadai sangat berharga dalam *private audit*.

---

## Memeriksa Coverage dengan Foundry

Perintah dasar untuk membangun dan menjalankan tes:

```bash
forge build
forge test
```

Untuk memeriksa coverage (seberapa banyak kode yang dijalankan oleh tes):

```bash
forge coverage
```

Contoh Output Coverage

/security-section-3/12-protocol-tests/protocol-tests2.png

Coverage terlihat bagus... benarkah?

---

Peringatan: Coverage Adalah Vanity Metric

"Coverage may be a vanity metric and not truly representative of what's being tested for."

Coverage tinggi tidak menjamin bahwa:

· Semua skenario kerentanan telah diuji.
· Tes yang ada benar-benar menguji hal yang tepat.
· Kerentanan logika bisnis atau dokumentasi terdeteksi.

Contoh Kasus: PasswordStore

Jika kita lihat lebih dekat tes yang disertakan dalam repo:

```solidity
function test_owner_can_set_password() public {
    vm.startPrank(owner);
    string memory expectedPassword = "myNewPassword";
    passwordStore.setPassword(expectedPassword);
    string memory actualPassword = passwordStore.getPassword();
    assertEq(actualPassword, expectedPassword);
}

function test_non_owner_reading_password_reverts() public {
    vm.startPrank(address(1));
    vm.expectRevert(PasswordStore.PasswordStore__NotOwner.selector);
    passwordStore.getPassword();
}
```

Apa yang Dilewatkan oleh Tes Ini?

Kerentanan yang Ditemukan Apakah Diuji?
Access Control pada setPassword() (siapa pun bisa mengubah password) ❌ TIDAK
NatSpec tidak akurat pada getPassword() ❌ (bukan ranah tes otomatis)
Penyimpanan password di on-chain (publik) ❌ (kesalahan business logic)

Kesimpulan: Coverage tinggi bisa tercapai meskipun kerentanan kritis tidak diuji sama sekali.

---

Apa yang Tidak Bisa Ditangkap oleh Tes?

· Masalah dokumentasi (NatSpec tidak akurat, kurang jelas).
· Kesalahan business logic (misal: menyimpan rahasia di on-chain).
· Invariant yang tidak didefinisikan atau tidak diuji.
· Asumsi desain yang keliru (seperti menganggap private berarti rahasia).

"It's important not to assume things are fine because our framework tells us so."

---

Kesimpulan

· Coverage tinggi ≠ kode aman. Coverage hanya mengukur seberapa banyak baris kode dieksekusi, bukan apakah skenario serangan yang tepat telah diuji.
· Auditor harus memeriksa test suite untuk melihat apakah tes benar-benar mencakup skenario yang relevan (misal: akses kontrol, invariant, edge cases).
· Tes otomatis tidak bisa menggantikan manual review untuk menemukan masalah dokumentasi, logika bisnis, atau asumsi desain yang salah.
· Jangan terbuai oleh metrik – selalu verifikasi secara manual.

"We're really progressing through this process well and we're ready to write a report for each of our findings."