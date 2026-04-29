# Latihan Setelah Audit Puppy Raffle

## Sumber Belajar Tambahan

Setelah menyelesaikan audit Puppy Raffle, ada beberapa sumber daya yang dapat digunakan untuk mengasah keterampilan keamanan smart contract lebih lanjut.

Repositori utama: [**sc-exploits-minimized**](https://github.com/Cyfrin/sc-exploits-minimized)

Di dalamnya terdapat tiga bagian penting:
- `Ethernaut`
- `Damn Vulnerable DeFi`
- `Case Studies`

---

## Ethernaut

**Platform:** [Ethernaut](https://ethernaut.openzeppelin.com/) oleh OpenZeppelin

**Deskripsi:** Kumpulan CTF (Capture The Flag) di mana Anda belajar mengeksploitasi berbagai kerentanan dalam lingkungan semi-live.

**Tingkat kesulitan:** Pemula hingga menengah

**Rekomendasi:** Mulai dengan tantangan `Hello Ethernaut` untuk memahami dasar-dasar cara kerja Ethernaut.

**Catatan:** Diperlukan sedikit pengetahuan JavaScript untuk beberapa fungsi. Alternatifnya, Anda bisa menggunakan Foundry atau Etherscan untuk berinteraksi dengan kontrak yang di-deploy.

**Tugas yang direkomendasikan:** Selesaikan tantangan 1, 9, dan 10.

---

## Damn Vulnerable DeFi

**Platform:** [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)

**Deskripsi:** Kumpulan tantangan serupa dengan Ethernaut, tetapi **lebih menantang**.

**Tingkat kesulitan:** Menengah hingga lanjut

**Catatan penting:** 
- DVD awalnya ditulis dalam Hardhat (JavaScript).
- **Update:** Versi 4.0.0 ke atas sudah ditulis ulang dalam Foundry oleh The Red Guild. [Repo baru di sini](https://github.com/theredguild/damn-vulnerable-defi/tree/master)

**Alternatif jika tidak nyaman dengan Hardhat:** Salin kontrak DVD ke proyek Foundry dan coba eksploitasi secara lokal. Setiap tantangan menyertakan deskripsi tujuan yang jelas.

---

## Case Studies

Bagian ini berisi contoh studi kasus nyata dari kerentanan yang telah dibahas. Berguna untuk memahami dampak eksploitasi di dunia nyata, melampaui teori.

---

## Tugas Tambahan untuk Melatih Skill

1. **Selesaikan Ethernaut Challenges** (minimal 1, 9, dan 10)
2. **Daftar ke [Solodit](https://solodit.xyz/)** – platform untuk belajar dari laporan audit orang lain.
3. **Tweet pencapaian** menyelesaikan audit Puppy Raffle.
4. **Daftar ke [Farcaster](https://www.farcaster.xyz/)** – jaringan sosial terdesentralisasi untuk komunitas kripto.
5. **Ikuti CodeHawks First Flight** – kompetisi audit pemula di [CodeHawks](https://www.codehawks.com/first-flights)

---

## NFT Challenges (Bagian 4)

Sebagai tantangan tambahan, coba analisis kontrak berikut:

- **Arbitrum:** `0xef72ba6575b86beaa9b9e4a78bca4a58f3cce276`
- **Sepolia:** `0xf988ebf9d801f4d3595592490d7ff029e438deca`

Keduanya merupakan *combination hack* – menggabungkan beberapa kerentanan dalam satu kontrak.