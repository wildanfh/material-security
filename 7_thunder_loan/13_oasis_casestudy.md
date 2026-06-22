# Ringkasan Materi: Studi Kasus Oasis – Sentralisasi sebagai Kerentanan

## Pendahuluan

Pada pelajaran sebelumnya, kita membahas **risiko sentralisasi (Centralization Risk)** sebagai salah satu temuan dalam audit ThunderLoan. Sekarang, kita akan melihat studi kasus nyata yang melibatkan protokol **Oasis** untuk memahami dampak nyata dari kerentanan ini.

> Artikel mendetail tentang kasus ini dapat dibaca di [Medium](https://medium.com/@observer1/uk-court-ordered-oasis-to-exploit-own-security-flaw-to-recover-120k-weth-stolen-in-wormhole-hack-fcadc439ca9d).

---

## Apa Itu Oasis?

**Oasis.app** adalah sistem yang memungkinkan pengguna meminjam dan meminjamkan aset di protokol Maker. Platform ini mempromosikan diri sebagai **terdesentralisasi dan tanpa izin (permissionless)** – namun kenyataannya tidak sepenuhnya demikian.

---

## Kronologi Kejadian

### 1. Peretasan Wormhole Bridge (Februari 2022)

- Wormhole bridge mengalami peretasan yang mengakibatkan hilangnya **~120.000 ETH** (sekitar 120 ribu WETH).
- Seperti biasa, peretas memindahkan dana curian ke berbagai protokol dalam upaya mengaburkan jejak.
- Pada suatu titik, peretas menyetor dana tersebut ke **Oasis.app** – dan meninggalkan celah untuk dieksploitasi balik.

### 2. Celah di Oasis: Upgradeable Proxy dengan Multisig

Oasis beroperasi pada **multisig upgradeable proxy 4 dari 12** – artinya dengan **4 tanda tangan** dari 12 penandatangan, protokol dapat diubah sesuka hati.

Ini berarti Oasis **tidak benar-benar terdesentralisasi** – ada sekelompok kecil orang yang memiliki kekuatan untuk mengubah seluruh protokol.

### 3. Pengadilan Tinggi Inggris Memerintahkan Eksploitasi

Pada tahun 2023, **Pengadilan Tinggi Inggris** mengeluarkan perintah yang memaksa Oasis untuk:
- **Meningkatkan (upgrade) protokol** mereka.
- **Memanfaatkan celah keamanan** mereka sendiri untuk menutup vault peretas dan mengambil kembali dana curian.

> *"The whole saga entailed coordination between the Oasis' founding team and the wormhole developer from Jump Crypto, the trading firm that had lost their money in the first place."* — Extract from Blockworks Research Article.

---

## Mengapa Ini Bisa Terjadi?

- Jika Oasis benar-benar terdesentralisasi dan **tahan sensor (censorship-resistant)**, perintah pengadilan seperti ini **tidak mungkin** dilaksanakan.
- Kemampuan untuk meng-upgrade protokol dengan 4 tanda tangan membuktikan bahwa Oasis **masih memiliki otoritas terpusat**.
- Ini adalah **kontradiksi langsung** dengan klaim desentralisasi mereka.

---

## Dilema Sentralisasi: Manis atau Pahit?

| Sisi Positif | Sisi Negatif |
|--------------|--------------|
| Dana curian berhasil **dipulihkan** (hal yang baik). | **Kurangnya desentralisasi** terbukti – seluruh sistem masih dibangun di atas **kepercayaan**, bukan *trustless*. |
| Korban mendapatkan kembali aset mereka. | Jika tim Oasis jahat, mereka bisa melakukan hal yang sama untuk **mencuri dana** kapan saja. |
| | Citra Oasis sebagai platform terdesentralisasi **rusak**. |
| | Pengguna menyadari bahwa protokol yang mengklaim desentralisasi **tidak sepenuhnya aman** dari intervensi pihak ketiga. |

---

## Pelajaran untuk Security Researcher

1. **Sentralisasi adalah kerentanan** – meskipun sering dianggap "known issue", risiko ini harus selalu dicatat dalam laporan audit.

2. **Klaim desentralisasi tidak cukup** – auditor harus memeriksa apakah protokol benar-benar memiliki mekanisme yang mencegah otoritas terpusat (misal: timelock, multi-sig dengan threshold tinggi, DAO).

3. **Kasus Oasis menunjukkan** bahwa bahkan protokol besar pun dapat dimanipulasi oleh pengadilan atau aktor jahat jika memiliki backdoor berupa upgradeability tanpa perlindungan yang memadai.

4. **Web3 harus striving untuk trustless** – jika sebuah protokol masih bergantung pada kepercayaan kepada sekelompok kecil orang, maka ia belum sepenuhnya terdesentralisasi.

---

## Kesimpulan

Kasus Oasis adalah contoh sempurna dari **dilema sentralisasi dalam DeFi**:

- Di satu sisi, kemampuan untuk memulihkan dana curian adalah **kemenangan** bagi korban.
- Di sisi lain, ini membuktikan bahwa **sistem masih rentan terhadap otoritas terpusat** – baik itu pengadilan, tim pengembang, atau aktor jahat.

> *"The whole system was still built on trust and Web3 should strive to be trustless!"*

---

## Referensi

- Artikel detail: [UK Court Ordered Oasis to Exploit Own Security Flaw](https://medium.com/@observer1/uk-court-ordered-oasis-to-exploit-own-security-flaw-to-recover-120k-weth-stolen-in-wormhole-hack-fcadc439ca9d)
- Repositori eksperimen: [sc-exploits-minimized – Centralization](https://github.com/Cyfrin/sc-exploits-minimized/tree/main/src/centralization)
