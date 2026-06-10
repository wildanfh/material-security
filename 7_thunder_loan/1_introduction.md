# Ringkasan Materi: Introduction to Thunderloan (Section 6)

## Pendahuluan

Section 6 – Thunder Loan merupakan kelanjutan dari materi sebelumnya dengan **fokus besar pada pengujian (testing)**. Di sini kita akan memperdalam pengetahuan tentang sistem pinjam-meminjam DeFi, oracle, kontrak upgradeable, dan flash loan.

---

## Langkah Awal: Clone Repositori

```bash
git clone https://github.com/Cyfrin/6-thunder-loan-audit.git
cd 6-thunder-loan-audit/
code .
```

> **Catatan:** Branch `audit-data` dalam repositori ini berfungsi sebagai "kunci jawaban". Jika mengalami kesulitan, Anda dapat melihatnya untuk petunjuk.

---

## Topik Utama yang Akan Dipelajari

### 1. Sistem Borrowing & Lending DeFi

Thunder Loan didasarkan pada **Aave** dan **Compound** – dua protokol paling signifikan di DeFi. Kita akan mempelajari mekanisme pinjam-meminjam, bagaimana suku bunga ditentukan, serta risiko yang terkait.

### 2. Oracle & Chainlink

Memahami bagaimana protokol mengintegrasikan **oracle** untuk mendapatkan data harga yang andal dan terdesentralisasi. Chainlink akan menjadi contoh utama.

### 3. Kontrak Upgradeable (Proxy)

Ini akan menjadi **mock audit pertama** untuk kontrak upgradeable. Topik yang dicakup:

- **UUPS & Transparent Proxy** (dari OpenZeppelin)
- **Multi‑facet Proxy (Diamond / EIP‑2535)**
- **Foundry Proxies & Upgrades** – [repo terpisah](https://github.com/Cyfrin/foundry-upgrades-f23)
- Video: [Apa itu upgradeable smart contracts?](https://www.youtube.com/watch?v=bdXJmWajZRY)

### 4. Flash Loan

Kita akan mendalami **flash loan** – topik panas di DeFi. Materi mencakup cara kerja flash loan, apa yang mendasarinya, serta jenis eksploitasi yang dapat dilakukan dengan flash loan (misal: manipulasi harga, arbitrase, dll).

---

## Tooling Baru

| Alat | Fungsi |
|------|--------|
| **[Upgradehub](https://upgradehub.xyz/)** | Melihat riwayat perubahan selama upgrade kontrak dengan memasukkan alamat kontrak. Sangat berguna untuk audit upgradeable contract. |

---

## Pentingnya Section Ini

Dari perspektif DeFi, **section ini adalah salah satu yang terpenting dalam keseluruhan kursus**. Pengetahuan yang diperoleh akan langsung dapat diterapkan dalam security review dunia nyata, terutama untuk codebase yang lebih besar dan kompleks.

> *"The knowledge you take away from here will be directly applicable to real world security reviews."*

---

## Catatan Tambahan

- Selalu ada kemungkinan bug dalam codebase ini yang tidak tercakup dalam materi. Anda bebas menulis temuan sendiri untuk latihan.
- Jangan lupa untuk memanfaatkan branch `audit-data` sebagai petunjuk jika menemui jalan buntu.

---

## Kesimpulan

Section 6 akan menguji kemampuan pengujian (testing) secara ekstensif, memperkenalkan konsep lanjutan DeFi, serta memberikan pengalaman langsung dengan kontrak upgradeable dan flash loan. Siapkan diri untuk tantangan yang lebih besar!