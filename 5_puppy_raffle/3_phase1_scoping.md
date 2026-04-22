# Puppy Raffle – Fase Scoping

## Pendahuluan

Setelah mencoba melakukan review mandiri, sekarang kita melakukan **scoping bersama** pada repositori Puppy Raffle. Fase ini bertujuan untuk memahami lingkup, konfigurasi, dan dokumentasi yang disediakan oleh protokol.

---

## Membaca README

README repositori [**Puppy Raffle**](https://github.com/Cyfrin/4-puppy-raffle-audit) sudah cukup lengkap. Terdapat bagian-bagian yang kita harapkan:

- **About** – penjelasan proyek
- **Setup** – instruksi instalasi
- **Scope** – kontrak yang diaudit
- **Compatibilities** – kompatibilitas (rantai, versi Solidity, token)
- **Roles** – peran aktor dalam sistem
- **Known Issues** – masalah yang diketahui (tidak ada)

### Getting Started

Instruksi lokal:

```bash
git clone https://github.com/Cyfrin/4-puppy-raffle-audit.git
cd 4-puppy-raffle-audit.git
make
```

> **Periksa `Makefile`** untuk memahami apa yang dilakukan: membersihkan repo, menginstal paket (Foundry, OpenZeppelin, base64), lalu `forge build`.

---

## Testing dan Coverage

Setelah menjalankan `make`, langkah selanjutnya adalah memeriksa **test suite** protokol.

```bash
forge coverage
```

Hasil coverage terlihat **tidak bagus** (rendah). Ini memiliki implikasi berbeda tergantung jenis audit:

| Jenis Audit | Implikasi Coverage Rendah |
|-------------|---------------------------|
| **Competitive Audit** | Menarik – banyak peluang bug tersembunyi. |
| **Private Audit** | Kurang optimis – menandakan codebase belum matang. Tanggung jawab auditor lebih besar. |

---

## Scope (Lingkup Audit)

Dari README, kita mendapatkan detail scope:

```bash
git checkout <commitHash>
```

Perintah di atas memastikan kita mengaudit versi yang tepat. Kontrak yang masuk scope:

```
./src/
└── PuppyRaffle.sol
```

Hanya satu kontrak, tetapi lebih kompleks dari PasswordStore.

---

## Compatibilities (Kompatibilitas)

Perhatikan bagian **Compatibilities**:

- **Versi Solidity:** `0.7.6` (aneh – versi lama, catat sebagai potensi masalah)
- Rantai target: (sesuai README)
- Token yang kompatibel: (sesuai kebutuhan)

> Versi Solidity yang tidak umum atau usang harus dicatat sebagai area yang perlu diperiksa lebih lanjut.

---

## Roles (Peran)

Protokol mendokumentasikan peran dan wewenang:

| Peran | Hak |
|-------|-----|
| **Owner** | Deployer protokol. Dapat mengubah alamat dompet penerima biaya melalui `changeFeeAddress`. |
| **Player** | Peserta undian. Dapat masuk ke undian (`enterRaffle`) dan mengembalikan dana (`refund`). |

> Memahami peran ini penting untuk mendeteksi pelanggaran akses kontrol.

### Known Issues

Tidak ada masalah yang diketahui (*no known issues*). 😏

---

## Wrap Up

Scoping berjalan baik – protokol menyediakan dokumentasi memadai. Kita bahkan sudah menemukan **satu keanehan** (versi Solidity 0.7.6).  

Selanjutnya, kita akan mulai menggunakan **tools static analysis** (Slither, Aderyn) untuk menemukan kerentanan sebelum benar-benar membaca kode baris per baris.