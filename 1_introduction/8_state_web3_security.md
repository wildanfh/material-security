# State of Web3 Security: Ringkasan Materi

## 1. Realita Saat Ini: Skala Kerugian

- **$3.1 miliar** dicuri melalui crypto hacks sepanjang tahun 2022
- Sekitar **7% dari total nilai DeFi** berhasil dicuri oleh hacker
- Ancaman ini menjadi hambatan utama adopsi mainstream Web3

---

## 2. Pola Serangan Umum (Attack Vectors)

Beberapa vektor serangan yang paling sering ditemukan:

| Attack Vector | Keterangan |
|---|---|
| **Price Oracle Manipulation** | Memanipulasi sumber harga on-chain untuk keuntungan sendiri |
| **Reward Manipulation** | Mengeksploitasi mekanisme distribusi reward |
| **Stolen Private Keys** | Kompromi kunci privat / centralized key management |
| **Reentrancy** | Vulnerability klasik yang masih terus berulang hingga kini |

> ⚠️ Banyak vulnerability sudah dikenal bertahun-tahun, tapi developer masih terus mengulangi kesalahan yang sama karena kurangnya penerapan *best practices*.

---

## 3. Tantangan Besar yang Harus Diatasi

### a. Teknologi Terpusat (Centralized Technology)
- Private key yang dikompromikan
- Komponen off-chain yang menjadi single point of failure
- Bertentangan dengan prinsip desentralisasi Web3

### b. Lemahnya Post-Deployment Practices
- Minimnya monitoring aktif setelah kontrak di-deploy
- Sedikit protokol yang memanfaatkan **bug bounty** untuk keamanan berkelanjutan
- Kurangnya prosedur *emergency triage* yang terstruktur

### c. Tidak Mengikuti Best Practices
- Developer sering melewatkan standar keamanan yang sudah mapan
- Audit ≠ 100% aman — laporan audit yang tidak dibaca tidak memberikan manfaat apapun

### d. Disconnect antara Industri dan Security Professionals
- Protokol sering menganggap audit sebagai formalitas, bukan sebagai bagian dari budaya keamanan
- Rekomendasi dari security researcher sering diabaikan

---

## 4. Tren Positif: Web3 Security Sedang Membaik

### Pertumbuhan Ekosistem Keamanan
- Semakin banyak **auditor** dan **security researcher** yang masuk ke industri
- Meningkatnya investasi protokol dalam keamanan (audit pra-deploy + bug bounty pasca-deploy)
- Filosofi: *"Spending $1M on security now is worth saving $100M from being hacked"*

### Edukasi yang Berkembang
- Materi edukasi keamanan Web3 semakin berkualitas dan mudah diakses
- Developer lebih sadar terhadap best practices dan common pitfalls

### Peningkatan Tooling
- AI semakin dimanfaatkan untuk analisis keamanan
- Static analysis tools terus berkembang
- **[Solodit](https://solodit.xyz/)** — agregator temuan audit dari berbagai kompetisi dan firm

### Pemain Kunci di Ekosistem

**Pre-deployment (Audit Firms):**
- Cyfrin
- Trail of Bits
- OpenZeppelin

**Competitive Audit Platforms:**
- [CodeHawks](https://www.codehawks.com/)

**Independent Security Researchers:**
- [Pashov](https://twitter.com/pashovkrum) (contoh ISR terkemuka)

---

## 5. Sumber Daya Penting

| Resource | Deskripsi |
|---|---|
| [Block Threat Intelligence](https://newsletter.blockthreat.io/) by Peter Kacherginsky | Newsletter wajib bagi security researcher — merangkum hack terbaru dan analisisnya |
| [Solodit](https://solodit.xyz/) | Agregator temuan audit dari berbagai platform dan firm |
| [CodeHawks](https://www.codehawks.com/) | Platform kompetisi audit smart contract |

---

## 6. Pesan Kunci

```
Audit (Security Review) ≠ 100% Safe

Keamanan adalah proses berkelanjutan:
  Pre-deployment  →  Audit + Review
  Post-deployment →  Bug Bounty + Monitoring + Emergency Response
```

Keamanan Web3 adalah **tanggung jawab kolektif** — antara developer, auditor, researcher, dan komunitas. Semakin sedikit hack, semakin besar kepercayaan, semakin luas adopsi mainstream.

---

*Materi dari: "The Current State of Web3 Security: A Crucial Call to Action" — Cyfrin Updraft / Patrick Collins*
