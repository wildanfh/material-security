# Recommended Mitigation dalam Laporan Temuan

## Pendahuluan

Setelah menyelesaikan bagian **Description**, **Impact**, dan **Proof of Concept**, langkah terakhir dalam laporan temuan adalah **Recommended Mitigation** (Rekomendasi Mitigasi). Bagian ini adalah kesempatan bagi auditor untuk menunjukkan keahliannya dengan memberikan saran konkret tentang bagaimana memperbaiki kerentanan yang ditemukan.

> Rekomendasi mitigasi harus **praktis**, **dapat diimplementasikan**, dan **sesuai dengan arsitektur protokol**.

---

## Template Laporan Saat Ini (Sebelum Mitigasi)

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** ... (sudah diisi)

**Impact:** ... (sudah diisi)

**Proof of Concept:** ... (sudah diisi)

**Recommended Mitigation:**
```

---

Tantangan dalam Memberikan Mitigasi

Tidak semua kerentanan dapat diperbaiki dengan penyesuaian kode sederhana. Beberapa temuan, seperti penyimpanan password di on-chain, bersifat fundamental terhadap arsitektur protokol.

Dalam kasus PasswordStore:

· Masalahnya bukan pada bug kode, tetapi pada asumsi desain bahwa private memberikan kerahasiaan.
· Blockchain bersifat transparan – tidak ada data yang benar-benar tersembunyi.
· Perbaikan sederhana (seperti menambahkan access control) tidak akan menyelesaikan masalah.

Contoh Rekomendasi yang Tidak Tepat

❌ "Ganti private menjadi public" – tidak menyelesaikan masalah.
❌ "Tambahkan modifier onlyOwner pada getter" – data tetap bisa dibaca via storage.
❌ "Hapus fungsi getPassword()" – data masih bisa diakses langsung.

---

Contoh Rekomendasi Mitigasi yang Baik

Untuk kerentanan ini, auditor harus menyadari bahwa arsitektur perlu diubah total. Berikut contoh rekomendasi yang diberikan dalam materi:

```markdown
**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you would also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.
```

Penjelasan Rekomendasi Tersebut

Elemen Rekomendasi Alasan
Rethink overall architecture Masalah fundamental tidak bisa diperbaiki dengan tambalan kode.
Enkripsi off-chain, simpan ciphertext on-chain Data sensitif tidak pernah dalam bentuk plaintext di blockchain.
User mengingat password dekripsi off-chain Kunci dekripsi tidak pernah tersimpan di on-chain.
Hapus view function Mencegah kebocoran ciphertext melalui fungsi yang tidak sengaja dipanggil.

---

Prinsip Menulis Rekomendasi Mitigasi yang Efektif

Prinsip Penjelasan
Spesifik Jangan hanya bilang "perbaiki arsitektur" – jelaskan bagaimana.
Dapat ditindaklanjuti Tim protokol harus bisa mengimplementasikannya.
Mempertimbangkan trade-off Akui bahwa solusi mungkin memiliki konsekuensi (misal: user harus mengingat password lain).
Menunjukkan keahlian Auditor yang baik memberikan solusi, bukan hanya mengkritik.
Jujur tentang keterbatasan Jika tidak ada perbaikan sempurna, jelaskan opsi terbaik yang tersedia.

---

Hasil Akhir Laporan (Setelah Mitigasi)

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** All data stored on chain is public and visible to anyone. The `PasswordStore::s_password` variable is intended to be hidden and only accessible by the owner through the `PasswordStore::getPassword` function.

I show one such method of reading any data off chain below.

**Impact:** Anyone is able to read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:** [langkah-langkah dengan cast dan anvil]

**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you would also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.
```

---

Kesimpulan

· Recommended Mitigation adalah bagian akhir dari laporan temuan yang sangat penting untuk menunjukkan nilai tambah auditor.
· Untuk kerentanan fundamental, rekomendasi mungkin memerlukan redesain arsitektur, bukan sekadar tambalan kode.
· Auditor harus jujur jika tidak ada perbaikan sederhana, dan menawarkan solusi alternatif yang realistis.
· Rekomendasi yang baik membantu protokol belajar dan tumbuh – tidak hanya memperbaiki satu bug, tetapi menghindari pola serupa di masa depan.

"I challenge you to write something you'd think is more appropriate, or better expresses what the protocol could do in a situation like this!"