# Isolated Development Environments

## Mengapa Ini Penting?

Menurut Chain Analysis, **jenis serangan paling populer di 2024 adalah private key leak** — dan salah satu vector-nya adalah menjalankan kode yang tidak diverifikasi langsung di mesin host.

> Setiap developer dan security researcher perlu memahami cara melindungi mesin mereka dari kode yang belum terpercaya.

---

## Masalah Utama: Akses Tak Terbatas

Ketika kamu menjalankan script di host machine (misalnya `npm run` atau bash script), program tersebut mendapat akses penuh ke:

```
Host Machine
├── Hardware
└── Operating System (Linux / macOS / Windows)
    ├── 🌐 Network
    ├── 📁 Files (termasuk private key, .env, wallet!)
    └── 💻 Applications
```

**Contoh skenario berbahaya:**
- Clone repo audit target → jalankan `npm install` → ada postinstall script berbahaya
- Download tool baru → script mencuri file `.env` atau `~/.ssh/id_rsa`
- Jalankan PoC dari internet → malware exfiltrate private key

---

## Solusi: Docker Container / Dev Container

Alih-alih program berjalan langsung di host, kita isolasi di dalam **container** — versi terisolasi dari komputer kita.

```
Host Machine
├── Hardware
└── Host OS
    ├── Network (host)
    ├── Files (host)  
    └── Docker Container  ← program tidak terpercaya berjalan di sini
        ├── Network (isolated)
        ├── Files (isolated)
        └── Applications (isolated)
```

Program di dalam container **tidak bisa mengakses** network, file, atau aplikasi di luar container — kecuali kamu secara eksplisit mengizinkannya.

---

## Dev Container di VS Code

**Dev Container** adalah fitur bawaan VS Code yang memudahkan setup Docker untuk development.

### Setup Awal (Sekali Saja):

```
1. Install Docker Desktop
2. Install extension "Dev Containers" di VS Code
3. Clone repo yang ingin di-review
4. Buka VS Code → Command Palette (Ctrl+Shift+P)
5. Ketik: "Reopen in Container"
6. VS Code akan build dan masuk ke dalam container
```

### Struktur File Dev Container:

```
repo/
└── .devcontainer/
    ├── devcontainer.json   ← konfigurasi VS Code + container settings
    └── Dockerfile          ← instruksi apa yang diinstall di container
```

### Contoh Dockerfile untuk Foundry:

```dockerfile
# Mulai dari Linux Debian bersih
FROM debian:latest

# Install tools yang diperlukan
# - ZSH
# - Rust
# - UV (Python package manager)
# - Solidity tools
# - Foundry
```

Container ini **dimulai dari nol** — tidak ada akses ke file host kamu sama sekali.

---

## Workspace Mount — Bagian Kritis

Di `devcontainer.json` ada pengaturan **workspace mount** yang menentukan folder mana dari host yang bisa diakses container.

```json
{
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspaces,type=bind",
  "workspaceFolder": "/workspaces"
}
```

> ⚠️ **Hati-hati dengan mount settings ini.** Semakin sedikit akses yang kamu berikan ke container, semakin aman.

Cek lokasi direktori aktif di dalam container:
```bash
pwd
# Output: /workspaces
```

---

## Workflow Aman dengan Dev Container

```bash
# 1. Buka repo audit target di Dev Container
#    (bukan di host langsung!)

# 2. Clone repo lain yang ingin di-review DI DALAM container
git clone https://github.com/target-protocol/contracts.git

# 3. Buka di VS Code (masih dalam container)
code .

# 4. Jalankan semua tools di sini
npm install      # ← kalau ada script berbahaya, hanya container yang kena
forge build
forge test
slither .
```

Kalau ada script berbahaya yang berjalan → **hanya container yang terekspos**, bukan mesin kamu.

---

## Perbandingan: Tanpa vs Dengan Isolasi

| Situasi | Tanpa Docker | Dengan Dev Container |
|---|---|---|
| `npm install` script berbahaya | ❌ Akses ke semua file host | ✅ Terisolasi di container |
| Malicious bash script | ❌ Bisa steal private key | ✅ Tidak ada akses ke host |
| Forge fork test exploit | ❌ Network host terekspos | ✅ Network terisolasi |
| Private key di `~/.foundry` | ❌ Script bisa baca | ✅ Tidak ada di container |

---

## Kasus Penggunaan untuk Security Researcher

Ini sangat relevan untuk workflow bug bounty:

```
❌ Cara berbahaya:
git clone https://github.com/unknown-protocol/contracts
cd contracts
npm install        ← bisa ada postinstall malicious
forge build
forge test

✅ Cara aman:
1. Buka Dev Container
2. Di DALAM container:
   git clone https://github.com/unknown-protocol/contracts
   npm install     ← kalau ada malware, container yang kena, bukan host
   forge build
   forge test
```

---

## Quick Tips

> ⚠️ **Selalu berbahaya menjalankan kode yang tidak 100% kamu percaya**

> 🐳 **Docker container membantu melindungi dari script berbahaya yang tidak diketahui**

> 🔓 **Tidak ada cara yang 100% aman** — Docker mengurangi risiko, bukan menghilangkannya

---

## Resource

- [The Red Guild Blog on Dev Containers](https://theredguild.org) — referensi utama materi ini
- [VS Code Dev Containers Documentation](https://code.visualstudio.com/docs/devcontainers/containers)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
