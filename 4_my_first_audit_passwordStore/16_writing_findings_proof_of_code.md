# Proof of Concept (PoC) dalam Laporan Temuan

## Pendahuluan

Setelah mengisi bagian **Description** dan **Impact**, langkah kritis berikutnya dalam laporan temuan adalah **Proof of Concept (PoC)** atau **Proof of Code**. Bagian ini adalah **bukti tak terbantahkan** bahwa kerentanan yang diklaim benar-benar ada dan dapat dieksploitasi. PoC mencegah protokol mengabaikan temuan yang sah.

---

## Template Laporan Saat Ini

```markdown
### [S-#] Storing the password on-chain makes it visible to anyone and no longer private

**Description:** ... (sudah diisi)

**Impact:** ... (sudah diisi)

**Proof of Concept:**

**Recommended Mitigation:**
```

---

Apa Itu Proof of Concept (PoC)?

PoC adalah demonstrasi konkret yang menunjukkan bagaimana kerentanan dapat dieksploitasi. Dalam konteks smart contract, PoC biasanya berupa:

· Perintah terminal (cast, forge, dll)
· Kode Solidity (test case)
· Skrip atau langkah-langkah yang dapat dijalankan ulang

PoC harus reproduksibel – siapa pun yang mengikuti langkah-langkahnya akan mendapatkan hasil yang sama.

---

Contoh PoC untuk PasswordStore

Untuk kerentanan "password disimpan di on-chain (publik)", berikut langkah-langkah yang ditunjukkan:

Langkah 1: Menjalankan Local Chain

```bash
anvil
```

Catatan: Sebagian besar PoC tidak memerlukan blockchain lokal, tetapi ini membantu demonstrasi.

Langkah 2: Deploy Kontrak

```bash
make deploy
```

Proses deploy ini akan mengatur password awal (misal: myPassword).

Langkah 3: Membaca Storage Langsung dengan cast

Kita perlu mengetahui slot storage tempat s_password disimpan. Dalam kontrak ini, s_password berada di slot 1.

```bash
cast storage <ADDRESS_CONTRACT> 1 --rpc-url http://127.0.0.1:8545
```

Contoh output (hex):

```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```

Langkah 4: Mendekode Hex ke String

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

Output:

```
myPassword
```

Hasil: Password yang seharusnya hanya diketahui owner berhasil dibaca oleh siapa pun.

---

Menulis PoC dalam Laporan

Berikut adalah contoh penulisan PoC yang baik (diringkas):

```markdown
**Proof of Concept:**

The below test case shows how anyone could read the password directly from the blockchain. We use Foundry's cast tool to read directly from the storage of the contract, without being the owner.

1. Create a locally running chain:
```

make anvil

```

2. Deploy the contract to the chain:
```

make deploy

```

3. Run the storage tool (slot 1 because that's the storage slot of `s_password`):
```

cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545

```
   Output: `0x6d7950617373776f726400000000000000000000000000000000000000000014`

4. Parse the hex to a string:
```

cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014

```
   Output: `myPassword`
```

---

Karakteristik PoC yang Efektif

Karakteristik Penjelasan
Reproduksibel Langkah-langkah jelas dan dapat dijalankan ulang oleh siapa pun.
Minimal Hanya langkah yang diperlukan – tidak perlu kode berlebihan.
Menggunakan alat standar cast, forge, anvil – mudah diakses.
Menunjukkan dampak langsung Output akhir (misal password terbaca) terbukti.
Tidak bergantung pada asumsi Bukti nyata, bukan teori.

---

Mengapa PoC Sangat Penting?

· Meyakinkan protokol – tidak bisa berdebat dengan bukti yang berhasil dieksekusi.
· Mencegah pengabaian – protokol cenderung mengabaikan temuan tanpa bukti konkret.
· Mempercepat mitigasi – pengembang langsung melihat celah dan cara memperbaikinya.
· Meningkatkan kredibilitas auditor – laporan dengan PoC lebih dipercaya.

"This is the section that prevents protocols from disregarding legitimate concerns."

---

Kesimpulan

· Proof of Concept adalah bukti tak terbantahkan bahwa kerentanan ada dan dapat dieksploitasi.
· Gunakan alat standar seperti Foundry (cast, forge, anvil) untuk demonstrasi.
· PoC harus reproduksibel dan minimal.
· Setelah PoC selesai, laporan tinggal melengkapi bagian Recommended Mitigation (rekomendasi perbaikan).

"In a few quick commands we've shown that the data our client is expecting to keep hidden on chain is accessible to anyone."