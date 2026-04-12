## Apa Itu NFT?

**NFT** (Non-Fungible Token) atau **ERC721** adalah jenis token yang bersifat **unik** dan **tidak dapat dipertukarkan** satu sama lain. Berbeda dengan ERC20 yang fungible (setiap token memiliki nilai dan sifat yang sama), setiap NFT memiliki identitas dan nilai yang berbeda.

### Analogi Sederhana

| Fungible (ERC20) | Non-Fungible (ERC721) |
|------------------|----------------------|
| Uang kertas $1 – bisa ditukar dengan $1 lainnya, nilainya sama. | Pokémon unik – setiap Pikachu memiliki karakteristik berbeda. |
| Token LINK, DAI, USDT | Karya seni digital, kartu trading langka, item game. |

> NFT dapat dianggap sebagai **karya seni digital yang tidak dapat dihancurkan** dengan riwayat kepemilikan dan perubahan yang permanen.

## Perbedaan Utama: ERC20 vs ERC721

| Aspek | ERC20 | ERC721 (NFT) |
|-------|-------|--------------|
| Fungibility | Fungible (saling tukar) | Non-fungible (unik) |
| Identitas token | Semua token identik | Setiap token memiliki `tokenId` unik |
| Nilai | Setiap token bernilai sama | Setiap token bisa bernilai berbeda |
| Metadata | Tidak diperlukan | Menggunakan `tokenURI` untuk menyimpan metadata (gambar, atribut, dll) |

## Penggunaan NFT

NFT tidak terbatas pada seni digital. Berikut contoh penggunaannya:

- **Karya seni digital** – Lukisan, foto, ilustrasi.
- **Koleksi kartu trading** – Seperti CryptoPunks, Bored Ape.
- **Item dalam game** – Senjata, skin karakter, tanah virtual.
- **Sertifikat kepemilikan** – Rumah, kendaraan, atau aset fisik lainnya.
- **Tiket acara** – Konser, konferensi, pertandingan olahraga.
- **Identitas digital** – Domain ENS, verifikasi akun.

### Platform Perdagangan NFT

- [OpenSea](https://opensea.io/)
- [Rarible](https://rarible.com/)

NFT dapat diperdagangkan di pasar terdesentralisasi tersebut atau secara peer-to-peer di luar platform.

## Manfaat NFT bagi Seniman

Masalah dalam industri seni tradisional:
- Karya seni mudah disalin tanpa atribusi.
- Seniman jarang mendapat kompensasi yang adil dari hasil karyanya.

Solusi dengan NFT:
- **Royalty mekanisme terdesentralisasi** – Seniman mendapat persentase setiap kali NFT dijual kembali.
- **Rekam jejak transparan** – Semua transfer dan kepemilikan tercatat di blockchain.
- **Kompensasi langsung** – Tidak perlu perantara.

> *"Memiliki mekanisme royalty terdesentralisasi sangat penting agar seniman mendapat kompensasi yang akurat atas karya mereka."*

## Standar ERC721

ERC721 adalah standar token non-fungible pertama yang populer di Ethereum. Fungsi wajib dalam ERC721 antara lain:

| Fungsi | Deskripsi |
|--------|------------|
| `balanceOf(address owner)` | Jumlah NFT yang dimiliki suatu alamat. |
| `ownerOf(uint256 tokenId)` | Alamat pemilik dari suatu tokenId. |
| `safeTransferFrom(from, to, tokenId)` | Transfer NFT dengan pemeriksaan keamanan. |
| `transferFrom(from, to, tokenId)` | Transfer NFT. |
| `approve(address to, uint256 tokenId)` | Memberi izin ke alamat lain untuk mengelola token. |
| `getApproved(uint256 tokenId)` | Mendapatkan alamat yang disetujui untuk token. |
| `setApprovalForAll(operator, approved)` | Memberi/mencabut izin untuk semua token. |
| `isApprovedForAll(owner, operator)` | Cek apakah operator diizinkan. |

### Perbedaan Paling Mendasar

Setiap token ERC721 memiliki **tokenId** yang unik. Id inilah yang membedakan satu NFT dengan NFT lainnya, meskipun berasal dari kontrak yang sama.

## Token URI dan Metadata

Karena biaya gas di Ethereum tinggi, menyimpan gambar atau atribut lengkap di blockchain tidak praktis. Solusinya adalah **Token URI**.

### Cara Kerja Token URI

- `tokenURI(uint256 tokenId)` mengembalikan string URI (biasanya URL atau IPFS hash).
- URI tersebut mengarah ke data JSON dengan format standar, misalnya:

```json
{
  "name": "Artwork #1",
  "description": "Karya seni digital unik",
  "image": "ipfs://QmX.../image.png",
  "attributes": [
    { "trait_type": "Warna", "value": "Merah" },
    { "trait_type": "Ukuran", "value": "Besar" }
  ]
}
```

### Penyimpanan Metadata

| Metode | Kelebihan | Kekurangan |
|--------|-----------|-------------|
| **On-chain** | Permanen, terdesentralisasi penuh | Biaya gas sangat mahal |
| **Off-chain (IPFS)** | Terdesentralisasi, lebih murah | Perlu gateway IPFS, tidak sepenuhnya abadi |
| **Off-chain (server sentral)** | Sangat murah, mudah diubah | Risiko server mati → link rusak |

> **Peringatan:** Jika menggunakan server sentral dan server tersebut mati, semua NFT yang mengandalkan link tersebut akan kehilangan metadata (gambar, deskripsi, dll). Ini adalah risiko besar dalam praktik NFT saat ini.

## Tantangan dan Perdebatan

- **On-chain vs off-chain metadata** – Mana yang lebih baik? On-chain lebih aman tetapi mahal. Off-chain lebih praktis tetapi kurang terdesentralisasi.
- **Royalty enforcement** – Tidak semua pasar NFT menghormati royalty yang ditetapkan di kontrak.
- **Environmental impact** – Transaksi NFT di Ethereum menggunakan energi (kini sudah berkurang dengan proof-of-stake).

## Kesimpulan

- NFT adalah token unik (ERC721) yang mewakili kepemilikan atas aset digital atau fisik.
- Berbeda dengan ERC20 yang fungible, setiap NFT memiliki `tokenId` berbeda.
- NFT menyelesaikan masalah kompensasi dan atribusi bagi seniman melalui royalty otomatis.
- Metadata NFT biasanya disimpan off-chain dengan Token URI, menggunakan IPFS atau server sentral.
- Standar ERC721 adalah fondasi bagi ekosistem NFT yang terus berkembang.
