# Menghitung nSLOC dengan CLOC untuk Estimasi Durasi Audit

## Pendahuluan

Dalam fase **Stats** dari dokumen onboarding, klien perlu menyediakan informasi tentang **jumlah baris kode** (nSLOC) dan kompleksitas. Sebagai auditor, Anda harus mampu menghitung sendiri data ini untuk memperkirakan durasi audit.

Alat yang direkomendasikan: [**CLOC**](https://github.com/AlDanial/cloc) – *Count Lines of Code*. CLOC mendukung berbagai bahasa pemrograman termasuk Solidity, Python, Rust, dan lainnya. Alat ini menghitung:
- Baris kosong (blank lines)
- Baris komentar
- Baris kode fisik (physical lines of source code)

---

## Instalasi CLOC

Berikut adalah perintah instalasi untuk berbagai sistem operasi:

| Sistem Operasi | Perintah Instalasi |
|----------------|--------------------|
| **Node.js (npm)** | `npm install -g cloc` |
| **Debian / Ubuntu** | `sudo apt install cloc` |
| **Red Hat / Fedora** | `sudo yum install cloc` |
| **Fedora 22+** | `sudo dnf install cloc` |
| **Arch Linux** | `sudo pacman -S cloc` |
| **Gentoo** | `sudo emerge -av dev-util/cloc` |
| **Alpine Linux** | `sudo apk add cloc` |
| **OpenBSD** | `doas pkg_add cloc` |
| **FreeBSD** | `sudo pkg install cloc` |
| **macOS (MacPorts)** | `sudo port install cloc` |
| **macOS (Homebrew)** | `brew install cloc` |
| **Windows (Chocolatey)** | `choco install cloc` |
| **Windows (Scoop)** | `scoop install cloc` |

Setelah instalasi, verifikasi dengan:

```bash
cloc --help
```

---

## Penggunaan CLOC

Perintah dasar:

```bash
cloc <direktori>
```

Untuk proyek PasswordStore:

```bash
cloc ./src/
```

### Contoh Output

```text
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Solidity                         1              2              2             12
-------------------------------------------------------------------------------
SUM:                             1              2              2             12
-------------------------------------------------------------------------------
```

- **blank** = baris kosong
- **comment** = baris komentar
- **code** = **nSLOC** (jumlah baris kode aktual)

> Pada contoh di atas, `PasswordStore.sol` memiliki **12 nSLOC**.

---

## Mengapa nSLOC Penting?

| Alasan | Penjelasan |
|--------|------------|
| **Estimasi durasi audit** | Dengan mengetahui nSLOC, Anda dapat memperkirakan berapa lama audit akan berlangsung berdasarkan kecepatan pribadi Anda. |
| **Perencanaan beban kerja** | Membantu Anda dan klien memahami skala pekerjaan yang akan dilakukan. |
| **Acuan untuk audit berikutnya** | Setelah mengaudit 1000 baris untuk pertama kali, Anda memiliki tolok ukur untuk audit serupa di masa depan. |
| **Seleksi kompetisi audit** | Platform competitive audit sering memiliki batasan waktu. Dengan mengetahui kecepatan Anda, Anda bisa memilih kontes yang sesuai (atau yang menantang untuk mempercepat). |

> *"When auditing 1000 lines of code for the first time, you now have an estimated timeline for subsequent audits or security reviews of 1000 lines codebases."*

---

## Faktor yang Mempengaruhi Kecepatan Audit

nSLOC hanyalah salah satu faktor. Kompleksitas juga sangat berpengaruh:

| Faktor | Dampak pada Durasi |
|--------|---------------------|
| **nSLOC rendah tapi kompleks** (misal: banyak logika matematika, assembly, proxy) | Durasi lebih lama dari perkiraan berbasis nSLOC saja. |
| **nSLOC tinggi tapi sederhana** (misal: ERC20 standar) | Durasi bisa lebih cepat. |
| **Kualitas dokumentasi dan test suite** | Dokumentasi dan test yang baik mempercepat pemahaman. |

> **Saran:** Gunakan nSLOC sebagai *baseline*, lalu sesuaikan dengan penilaian kompleksitas.

---

## Kesimpulan

- **nSLOC** adalah metrik penting untuk memperkirakan durasi audit.
- Gunakan **CLOC** untuk menghitung nSLOC secara akurat.
- Simpan catatan kecepatan audit Anda dari waktu ke waktu untuk meningkatkan akurasi estimasi.
- Ingatlah bahwa kompleksitas juga mempengaruhi durasi – jangan hanya mengandalkan nSLOC.
