# Persiapan Audit – Onboarding dan Scoping (PasswordStore)

## Pendahuluan

Setelah klien memberikan codebase yang diperbarui ([**Security Review CodeV2**](https://github.com/Cyfrin/3-passwordstore-audit)), auditor harus melakukan **onboarding** – mengajukan pertanyaan penting kepada klien untuk memastikan protokol siap diaudit. Fase ini krusial karena menentukan lingkup, ekspektasi, dan fokus audit.

---

## Apa yang Ditemukan di Codebase V2?

| Aspek | Status | Catatan |
|-------|--------|---------|
| Struktur folder | ✅ Ada (`src/`, script deployment) | Lengkap secara teknis. |
| README | ⚠️ Masih template Foundry | Perlu disesuaikan dengan kebutuhan protokol. |
| Test folder | ❌ Tidak ada | **Red flag** – tanpa test suite, sulit memverifikasi perilaku kode. |
| Lingkup audit | ❌ Tidak jelas | Auditor perlu bertanya: apakah library, script, deployment termasuk? |

> **Poin penting:** Jangan mulai audit sebelum lingkup jelas. Konfirmasi ke klien apa yang harus (dan tidak harus) direview.

---

## Onboarding Questions: Pertanyaan Wajib ke Klien

Dokumen referensi:
- [**Minimal Onboarding Questions**](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/minimal-onboarding-questions.md)
- [**Extensive Onboarding Questions**](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/extensive-onboarding-questions.md) (lebih lengkap, mirip dengan yang digunakan Cyfrin untuk private audit)

Berikut adalah 7 kategori pertanyaan beserta alasan mengapa penting:

### 1. About the Project (Tentang Proyek)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Apa tujuan protokol? Bagaimana bisnis logikanya? | 80% kerentanan berasal dari **implementasi business logic** yang salah. Anda perlu tahu apa yang *seharusnya* dilakukan kode. |

### 2. Stats (Statistik)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Berapa jumlah baris kode (nSLOC)? Berapa tingkat kompleksitas? | Membantu memperkirakan **waktu dan tenaga** yang dibutuhkan untuk audit. |

### 3. Setup (Cara Menjalankan)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Bagaimana cara build dan test proyek? Framework apa yang digunakan? | Agar auditor dapat menjalankan kode secara lokal dan mereproduksi masalah. |

### 4. Review Scope (Lingkup Review)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Apa **commit hash** yang akan dideploy? Kontrak mana saja yang masuk scope? | Anda tidak ingin menghabiskan waktu mengaudit kode yang sudah diubah atau tidak dipakai. Klien harus memberikan URL GitHub dan daftar kontrak secara eksplisit. |

### 5. Compatibilities (Kompatibilitas)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Versi Solidity? Rantai target (Ethereum, Polygon, dll)? Token apa yang diintegrasikan? | Kerentanan bisa berbeda antar rantai atau versi compiler. |

### 6. Roles (Peran dan Hak Akses)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Siapa saja aktor dalam sistem? Apa yang boleh dan tidak boleh mereka lakukan? | Untuk memverifikasi bahwa kontrol akses sudah sesuai dengan desain. |

### 7. Known Issues (Masalah yang Diketahui)
| Pertanyaan Contoh | Mengapa Penting |
|------------------|------------------|
| Apakah ada bug atau kerentanan yang sudah diketahui dan sedang diperbaiki? | Agar auditor tidak membuang waktu melaporkan hal yang sudah diketahui, dan fokus pada isu tersembunyi. |

> **Jika klien memberikan push back atau enggan menjawab, itu adalah red flag.** Sebagai auditor, Anda juga berperan sebagai **pendidik** – ajari mereka pentingnya keamanan dan dokumentasi.

---

## Setelah Klien Menjawab: Codebase V3

Klien yang baik akan memperbarui codebase dan mengisi kuesioner. Contoh: [**Security Review CodeV3**](https://github.com/Cyfrin/3-passwordstore-audit/tree/onboarded) dan [**minimal-onboarding-filled.md**](https://github.com/Cyfrin/3-passwordstore-audit/blob/onboarded/minimal-onboarding-filled.md).

### Contoh Jawaban Klien untuk PasswordStore

#### About
> *"A smart contract application for storing a password. Users should be able to store a password and then retrieve it later. Others should not be able to access the password."*

#### Setup (Instruksi lengkap)
```bash
git clone https://github.com/Cyfrin/3-passwordstore-audit
cd 3-passwordstore-audit
forge build
make anvil          # jalankan local node
make deploy         # deploy
forge test          # jalankan test
forge coverage      # lihat coverage
```

#### Scope
```
./src/
└── PasswordStore.sol
```
> Hanya satu kontrak. Dalam competitive audit, scope yang ditentukan adalah satu-satunya kode yang valid.

#### Compatibilities
```
Solc Version: 0.8.18
Chain(s) to deploy contract to: Ethereum
```

#### Roles
```
Owner: The user who can set the password and read the password.
Outsiders: No one else should be able to set or read the password.
```

#### Known Issues
> *"No known issues"* – Klien percaya diri. (Tentu saja akan ada temuan nanti 😄)

---

## Setup Lokal: Cloning dan Checkout Commit Hash

Setelah klien memberikan **commit hash** (7 karakter pertama), lakukan langkah berikut:

1. **Clone repositori**
   ```bash
   git clone https://github.com/Cyfrin/3-passwordstore-audit
   cd 3-passwordstore-audit
   code .   # buka di VS Code
   ```

2. **Checkout ke commit hash yang ditentukan**
   ```bash
   git checkout <commithash>
   ```
   Ini akan membawa Anda ke status **detached HEAD** (perubahan tidak akan tersimpan).

3. **Buat branch baru untuk bekerja**
   ```bash
   git switch -c passwordstore-audit
   ```

4. **Verifikasi branch**
   ```bash
   git branch
   ```

> **Catatan:** Selalu gunakan commit hash yang disepakati. Jangan asal mengaudit branch `main` terbaru karena bisa saja kode sudah berubah setelah audit dimulai.

---

## Ringkasan dan Pesan Penting

- **Onboarding questions** bukan formalitas – ini adalah alat untuk memastikan klien siap diaudit.
- **Tanpa test suite, dokumentasi, dan scope yang jelas, jangan mulai audit.**
- Auditor adalah **pendidik** – ajari klien praktik terbaik keamanan.
- **Gunakan commit hash** untuk memastikan Anda mengaudit versi yang tepat.
- Proses ini akan menjadi **kebiasaan** (second nature) setelah Anda melakukannya berulang kali.

Analisis src/PasswordStore.sol:                                     
                                                                      
  Bug 1 — CRITICAL: setPassword no access control
  function setPassword(string memory newPassword) external {          
      s_password = newPassword;  // ← siapapun bisa set!            
      emit SetNetPassword();                            
  }                                                                   
  Roles bilang: hanya owner boleh set password. Tapi msg.sender !=    
  s_owner check tidak ada. Siapapun bisa overwrite password.          
                                                                      
  ---                                                               
  Bug 2 — CRITICAL: s_password bukan benar-benar private              
  string private s_password;                                        
  private hanya cegah kontrak lain akses via Solidity. Data on-chain =
   public. Siapapun bisa baca storage slot langsung via               
  eth_getStorageAt. Password exposed ke semua orang.                  
                                                                      
  ---                                                                 
  Bug 3 — INFORMATIONAL: Typo event name + misleading NatSpec
  event SetNetPassword();  // ← "Net" harusnya "New"                  
  Dan NatSpec getPassword:                          
  * @param newPassword The new password to set.  // ← getPassword     
  tidak punya param                                                   
                   
  ---                                                                 
  Bug 4 — INFORMATIONAL: console.log di production                    
  import {console} from "forge-std/console.sol";                      
  // ...                                                              
  console.log(s_owner);  // di constructor
  Debug artifact — hapus sebelum deploy.                              
   
  ---                                                                 
  Summary vulnerability:
                                                                      
  ┌─────┬──────────┬────────────────────────────────────────────┐
  │  #  │ Severity │                   Issue                    │     
  ├─────┼──────────┼────────────────────────────────────────────┤
  │ 1   │ Critical │ setPassword no owner check                 │
  ├─────┼──────────┼────────────────────────────────────────────┤
  │ 2   │ Critical │ Password on-chain bisa dibaca siapapun     │     
  ├─────┼──────────┼────────────────────────────────────────────┤
  │ 3   │ Info     │ Typo event SetNetPassword → SetNewPassword │     
  ├─────┼──────────┼────────────────────────────────────────────┤     
  │ 4   │ Info     │ console.log tertinggal di constructor      │
  └─────┴──────────┴────────────────────────────────────────────┘     
                  
  Setup audit selesai:                                                
  - Repo: /home/wildanf/code/foundry/security/3-passwordstore-audit
  - Branch: passwordstore-audit                                       
  - Commit scope: 40248da      
  - Build: pass, coverage: 100% 
