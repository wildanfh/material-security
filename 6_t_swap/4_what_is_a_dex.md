# 🏦 DEX dan AMM: Panduan Pemula untuk Pertukaran Kripto Terdesentralisasi

Prinsip kerja bursa terdesentralisasi (DEX) seperti Uniswap sangat berbeda dengan bursa tradisional. Ringkasan ini akan mengupas tuntas konsep DEX dan mekanisme intinya, yaitu *Automated Market Maker* (AMM), termasuk contoh penerapannya di dunia nyata, kelebihan, serta potensi risikonya seperti *Impermanent Loss*.

---

## 📌 Bagian 1: Decentralized Exchange (DEX)

**DEX adalah bursa yang beroperasi di atas blockchain (seperti Ethereum atau Solana) tanpa otoritas pusat.** Ia menggunakan smart contract untuk mengotomatisasi perdagangan, memungkinkan pengguna untuk tetap memegang kendali penuh atas aset mereka sendiri (non-custodial). Ini berbeda dengan bursa tradisional (CEX) seperti Binance atau Kraken yang menyimpan dana pengguna. DEX menawarkan likuiditas instan untuk ribuan jenis token dan dapat diakses siapa saja tanpa perlu registrasi akun.

Ada banyak DEX di luar sana, dari yang paling populer hingga yang sangat spesifik. Untuk melihat dan membandingkannya, kalian bisa mengunjungi dashboard **DeFiLlama** yang melacak volume dari berbagai DEX (hanya spot/swap). Beberapa contoh nama DEX yang muncul antara lain 1Beets DEX, BEX, PHUX, Gaming DEX, KLEX Finance, dan Value Liquid.

---

## 🤖 Bagian 2: Automated Market Maker (AMM)

**AMM adalah jantung dari kebanyakan DEX modern.** Ia adalah protokol algoritmik yang bertindak sebagai "money robot" untuk menyediakan likuiditas secara otomatis. Alih-alih menunggu order beli dan jual bertemu (seperti di order book bursa tradisional), pengguna bertransaksi langsung dengan smart contract AMM.

#### 🆚 Perbandingan Cepat: DEX vs. CEX (Bursa Tradisional)

| Fitur Utama | Bursa Tradisional (CEX) | Bursa Terdesentralisasi (DEX) |
| :--- | :--- | :--- |
| **Model Utama** | Order Book (mencocokkan order beli & jual) | Automated Market Maker (AMM) |
| **Penyimpanan Dana** | Bursa memegang kendali atas dana pengguna (custodial). | Pengguna memegang kendali atas dana mereka sendiri (non-custodial). |
| **Proses Transaksi** | Kamu menunggu orang lain yang ingin membeli/jual di harga yang sama. | Kamu bertransaksi langsung dengan **Liquidity Pool** dari AMM. |

#### 🧠 Bagaimana Cara Kerja AMM?

- **Liquidity Pool (Pool Likuiditas):** Sebuah kontrak pintar yang berisi cadangan dua atau lebih aset kripto (misalnya, cadangan ETH dan USDC). Orang yang menyetor aset ke pool ini disebut **Liquidity Provider (LP)**.
- **Insentif untuk LP:** Untuk mendorong orang menyetor aset, AMM memberi mereka imbalan berupa sebagian dari biaya transaksi yang dihasilkan di pool tersebut, biasanya dalam bentuk **token LP**. Aktivitas menyetor aset untuk mendapat imbalan ini disebut **Yield Farming**.
- **Penentuan Harga Otomatis:** Harga aset di AMM ditentukan oleh rumus matematika berdasarkan rasio pasokan di dalam pool. Contohnya, jika banyak orang membeli ETH dari pool ETH/USDC, pasokan ETH berkurang sehingga harganya naik secara otomatis.

---

## 💡 Bagian 3: Rumus Kunci: `x * y = k`

Ini adalah formula fundamental yang digunakan AMM generasi pertama (Constant Product Market Maker), dipopulerkan oleh protokol seperti Bancor dan Uniswap. Intinya, hasil kali jumlah dua token dalam sebuah pool (X dan Y) harus selalu konstan (K).

- **Cara Kerja:** Jika rasio X berubah karena satu jenis token dibeli, maka jumlah token lainnya harus menyesuaikan agar `x * y` tetap sama. Hasilnya adalah kurva hiperbola di mana likuiditas selalu tersedia, tetapi dengan harga yang semakin tinggi ketika stok menipis.

> Ini adalah alasan mengapa di DEX, membeli dalam jumlah besar bisa menyebabkan harga melonjak drastis.

---

## ⚠️ Bagian 4: Istilah Penting yang Perlu Kamu Tahu

- **Liquidity Provider (LP):** Seseorang yang menyetorkan asetnya ke liquidity pool untuk mendapatkan imbalan.
- **Yield Farming:** Praktik menyetor aset (biasanya token LP) ke berbagai protokol DeFi untuk memaksimalkan imbalan.
- **Token LP:** Token yang diberikan kepada LP sebagai bukti kepemilikan saham mereka di sebuah liquidity pool. Token ini bisa disetor kembali ke protokol lain untuk yield farming.
- **Impermanent Loss (IL):** Kerugian sementara yang dialami LP ketika harga aset di pool berubah drastis dibandingkan jika mereka hanya menyimpan aset tersebut di dompet.

---

## ✨ Ringkasan

- **DEX** adalah bursa kripto yang terdesentralisasi, tanpa perantara, dan pengguna memegang kendali penuh atas asetnya.
- **AMM** adalah mekanisme di balik kebanyakan DEX, menggunakan "money robot" algoritmik dan **liquidity pool** untuk memfasilitasi perdagangan.
- **Liquidity Providers (LP)** dan **Yield Farmers** adalah peserta kunci yang menyediakan likuiditas dan mendapatkan imbalan sebagai gantinya.
- Rumus **`x * y = k`** adalah fondasi matematika untuk model AMM Constant Product yang paling umum, yang menentukan harga berdasarkan rasio pasokan di dalam pool.
- AMM bukan tanpa risiko. **Impermanent Loss** adalah risiko utama yang perlu dipahami oleh setiap LP.