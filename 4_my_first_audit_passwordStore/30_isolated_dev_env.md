# Isolated Development Environments (Docker & Dev Containers)

## Pendahuluan

Menurut Chain Analysis, pada tahun 2024, **jenis serangan paling umum adalah kebocoran kunci privat (private key leak)**. Salah satu vektor utama kebocoran adalah menjalankan **kode yang tidak diverifikasi** langsung di mesin host. Materi ini membahas cara memitigasi risiko tersebut dengan **lingkungan pengembangan terisolasi** menggunakan **Docker** dan **Dev Containers** di VS Code.

---

## Mengapa Isolasi Diperlukan?

### Risiko Menjalankan Kode di Host Machine

Komputer kita memiliki:
- **Hardware** (CPU, RAM, disk)
- **Host Operating System** (Linux, macOS, Windows)
- Di dalam OS terdapat: **jaringan (network)** , **file system**, dan **aplikasi**

Jika kita menjalankan script (misal: `npm run` atau bash script) langsung di host machine, script tersebut memiliki **akses penuh** ke:
- Jaringan
- File system
- Aplikasi lain

> Kode yang tidak diverifikasi bisa berisi malware yang mencuri kunci privat, mengakses file sensitif, atau merusak sistem.

### Solusi: Docker Containers

Docker memungkinkan kita membuat **versi terisolasi** dari komponen komputer (file system, jaringan, dll) yang berjalan terpisah dari host.

- Kode yang mencurigakan dijalankan **di dalam container**, bukan langsung di host.
- Container memiliki lingkungan terbatas – tidak bisa mengakses file atau jaringan host kecuali diizinkan secara eksplisit.

---

## Cara Kerja Dev Container di VS Code

**Dev Container** adalah fitur VS Code yang mengintegrasikan Docker langsung ke alur kerja pengembangan.

### Langkah-langkah Setup

1. **Clone repositori** yang sudah dikonfigurasi dengan `.devcontainer`.
2. **Pastikan Docker berjalan** (Docker Desktop atau Docker Engine).
3. Di VS Code, buka **Command Palette** (`Ctrl+Shift+P` atau `Cmd+Shift+P`).
4. Pilih: **`Reopen in Container`**.
5. VS Code akan membangun container sesuai konfigurasi dan membuka ulang proyek di dalam container.

> Untuk memverifikasi container berjalan, buka Docker Desktop – container akan terlihat aktif.

### Struktur Folder `.devcontainer`

| File | Fungsi |
|------|--------|
| `devcontainer.json` | Konfigurasi utama untuk VS Code (mounts, extensions, post-create commands, dll). |
| `Dockerfile` | Instruksi untuk membangun image Docker (dari base image, instalasi tools, dll). |

### Contoh Dockerfile (untuk Foundry)

- Mulai dari **Debian Linux** (base image kosong).
- Menginstal berbagai tools:
  - ZSH (shell)
  - Rust & UV (package manager Python modern)
  - Alat Solidity (forge, cast, anvil)
  - Foundry suite

### Workspace Mount

Dev container menggunakan **workspace mount** – folder proyek lokal di-mount ke dalam container.

> **Peringatan:** Anda harus **spesifik** tentang akses apa yang diberikan ke container. Jangan memberikan akses ke seluruh file system jika tidak diperlukan.

### Memeriksa Direktori di Dalam Container

Jalankan di terminal VS Code (yang sudah berada di dalam container):

```bash
pwd
```

Output akan menunjukkan /workspaces/... – ini adalah direktori kerja di dalam container, bukan di host.

Mengkloning Repositori Lain di Dalam Container

Dari terminal container, Anda bisa menjalankan:

```bash
git clone <repo-url>
cd <repo-name>
code .
```

Perintah code . akan membuka direktori tersebut di jendela VS Code yang masih berada di dalam container yang sama (atau membuat jendela baru yang terhubung ke container).

---

Keamanan: Tips Penting

Tips Penjelasan
Selalu curiga pada kode yang tidak 100% Anda percayai Jangan pernah menjalankan skrip dari sumber tidak dikenal langsung di host.
Gunakan isolated environment Docker, VM, atau sandbox lainnya untuk menguji kode mencurigakan.
Tidak ada jaminan keamanan 100% Isolasi mengurangi risiko, tetapi tidak menghilangkan sama sekali. Tetap waspada.

---

Kesimpulan

· Kebocoran kunci privat adalah ancaman terbesar di Web3 (2024).
· Menjalankan kode tidak terverifikasi di host machine sangat berbahaya.
· Docker containers (khususnya Dev Containers di VS Code) menyediakan lingkungan terisolasi untuk menjalankan kode tanpa membahayakan host.
· Selalu periksa konfigurasi mount – batasi akses container seminimal mungkin.
· Tidak ada solusi 100% aman, tetapi isolasi adalah langkah penting untuk mitigasi risiko.

"Keep these in mind to stay safe out there!"