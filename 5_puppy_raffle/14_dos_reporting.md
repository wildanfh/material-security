# Menulis Laporan Denial of Service (DoS) untuk Puppy Raffle

## Pendahuluan

Setelah menemukan kerentanan **Denial of Service (DoS)** pada fungsi `PuppyRaffle::enterRaffle`, langkah selanjutnya adalah menulis laporan temuan yang jelas dan profesional. Materi ini memandu pembuatan laporan untuk kerentanan DoS akibat *nested loop* yang tidak efisien.

---

## Kerentanan: DoS via Gas Cost Increment

### Root Cause
Fungsi `enterRaffle` menggunakan **dua perulangan bersarang (nested loop)** untuk memeriksa duplikat pemain. Perulangan ini memiliki kompleksitas **O(n²)** – semakin banyak pemain, semakin banyak iterasi yang harus dilakukan.

### Impact
- Biaya gas untuk pemain baru meningkat secara signifikan seiring bertambahnya ukuran array `players`.
- Pemain yang masuk di awal akan membayar gas murah, sementara pemain di akhir membayar sangat mahal (bisa 3x lipat atau lebih).
- Penyerang dapat mengisi array dengan banyak alamat sehingga tidak ada yang mau masuk lagi, menjamin kemenangan mereka.

---

## Menentukan Severity

| Faktor | Penilaian | Alasan |
|--------|-----------|--------|
| **Impact** | Medium | Protokol tidak sepenuhnya rusak, hanya menjadi lebih mahal untuk berpartisipasi. |
| **Likelihood** | Medium | Penyerang membutuhkan biaya besar untuk mengisi array, tetapi jika termotivasi (misal: NFT langka), serangan mungkin terjadi. |

**Matriks Severity: Medium Impact + Medium Likelihood = Medium (M)**

Label: `[M-#]`

---

## Judul Laporan (Title)

Aturan praktis: **Root Cause + Impact**

Judul yang diusulkan:

```
[M-#] Looping through players array to check for duplicates in `PuppyRaffle::enterRaffle` is a potential denial of service (DoS) attack, incrementing gas costs for future entrants
```

> Meskipun panjang, judul ini jelas dan informatif.

---

## Deskripsi (Description)

Penjelasan singkat tentang bagaimana kerentanan bekerja:

```markdown
**Description:** The `PuppyRaffle::enterRaffle` function loops through the `players` array to check for duplicates. However, the longer the `PuppyRaffle::players` array is, the more checks a new player will have to make. This means the gas costs for players who enter right when the raffle starts will be dramatically lower than those who enter later. Every additional address in the `players` array is an additional check the loop will have to make.
```

Sertakan potongan kode yang rentan dengan komentar `// @audit`:

```solidity
// @audit DoS Attack
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate Player");
    }
}
```

---

## Dampak (Impact)

Jelaskan konsekuensi bagi pengguna dan protokol:

```markdown
**Impact:** The gas costs for raffle entrants will greatly increase as more players enter the raffle, discouraging later users from entering and causing a rush at the start of a raffle to be one of the first entrants in queue.

An attacker might make the `PuppyRaffle::players` array so big that no one else enters, guaranteeing themselves the win.
```

---

## Proof of Concept (PoC)

Demonstrasi kuantitatif perbedaan biaya gas:

- **100 pemain pertama:** ~6.252.048 gas
- **100 pemain kedua:** ~18.068.138 gas (lebih dari 3x lipat)

### Kode Test (Foundry)

```solidity
function testDenialOfService() public {
    vm.txGasPrice(1);

    uint256 playersNum = 100;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < players.length; i++) {
        players[i] = address(i);
    }

    uint256 gasStart = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * players.length}(players);
    uint256 gasEnd = gasleft();
    uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
    console.log("Gas cost of the first 100 players: ", gasUsedFirst);

    address[] memory playersTwo = new address[](playersNum);
    for (uint256 i = 0; i < playersTwo.length; i++) {
        playersTwo[i] = address(i + playersNum);
    }

    uint256 gasStartTwo = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * players.length}(playersTwo);
    uint256 gasEndTwo = gasleft();
    uint256 gasUsedSecond = (gasStartTwo - gasEndTwo) * tx.gasprice;
    console.log("Gas cost of the second 100 players: ", gasUsedSecond);

    assert(gasUsedSecond > gasUsedFirst);
}
```

> Gunakan dropdown `<details>` untuk menyembunyikan blok kode yang panjang.

---

## Laporan Sementara (Sebelum Mitigasi)

<details>
<summary>Hasil Laporan DoS (sementara)</summary>

```markdown
### [M-#] Looping through players array to check for duplicates in `PuppyRaffle::enterRaffle` is a potential denial of service (DoS) attack, incrementing gas costs for future entrants

**Description:** The `PuppyRaffle::enterRaffle` function loops through the `players` array to check for duplicates. However, the longer the `PuppyRaffle::players` array is, the more checks a new player will have to make. This means the gas costs for players who enter right when the raffle starts will be dramatically lower than those who enter later. Every additional address in the `players` array is an additional check the loop will have to make.

```solidity
// @audit DoS Attack
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate Player");
    }
}
```

**Impact:** The gas costs for raffle entrants will greatly increase as more players enter the raffle, discouraging later users from entering and causing a rush at the start of a raffle to be one of the first entrants in queue.

An attacker might make the `PuppyRaffle::players` array so big that no one else enters, guaranteeing themselves the win.

**Proof of Concept:**  
If we have 2 sets of 100 players enter, the gas costs will be as such:  
- 1st 100 players: ~6252048 gas  
- 2nd 100 players: ~18068138 gas  

<details>
<summary>Proof of Code</summary>

```solidity
// test code di atas
```

</details>

**Recommended Mitigation:** (akan diisi di pelajaran berikutnya)
```

</details>

---

## Catatan Proses: Kapan Menulis Laporan?

- **Jangan terburu-buru menulis laporan** saat baru menemukan indikasi kerentanan – mungkin informasi lebih lanjut akan membatalkan temuan.
- Simpan **catatan inline** (komentar `// @audit`) dan kembali ke akhir setelah eksplorasi selesai.
- Tulis laporan ketika yakin kerentanan valid.

---

## Kesimpulan

- Kerentanan DoS akibat *nested loop* dengan kompleksitas O(n²) dapat membuat partisipasi di undian menjadi sangat mahal.
- Severity **Medium** (Impact Medium, Likelihood Medium) – tidak mematikan protokol tetapi dapat merusak pengalaman pengguna.
- PoC kuantitatif (perbandingan gas) sangat kuat untuk meyakinkan pengembang.
- Laporan yang rapi dengan judul jelas, deskripsi, impact, dan PoC akan membantu mitigasi.