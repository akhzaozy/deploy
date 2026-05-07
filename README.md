# 🚀 AutoDeploy Sistem Informasi Institut Teknologi Kalimantan — Panduan Lengkap

> Platform auto-hosting untuk mahasiswa ITK. Push ke GitHub → otomatis live di server.

---

## 📋 Daftar Isi

1. [Gambaran Sistem](#-gambaran-sistem)
2. [File yang Kamu Butuhkan](#-file-yang-kamu-butuhkan)
3. [Setup GitHub Repository](#-setup-github-repository)
4. [Cara Deploy Pertama Kali](#-cara-deploy-pertama-kali)
5. [Install cloudflared](#-langkah-1--install-cloudflared-sekali-saja)
6. [Login SSH ke Server](#-langkah-2--login-ssh-ke-server)
7. [Masuk ke Direktori Project](#-langkah-3--masuk-ke-direktori-project)
8. [Setup Laravel](#-langkah-4--setup-laravel)
9. [Cara Update Kode](#-cara-update-kode)
10. [Perintah Berguna](#-perintah-berguna-di-server)
11. [Troubleshooting](#-troubleshooting)

---

## 🗺 Gambaran Sistem

```
Kamu push kode ke GitHub
        │
        ▼
GitHub Actions berjalan otomatis
  • Deteksi framework (Laravel / PHP Composer / PHP Native)
  • Deteksi apakah ada package.json (npm)
  • Buat ZIP dari kode kamu (dotfiles disertakan, .env dikecualikan)
  • Kirim ke server via HTTPS
        │
        ▼
Server AutoDeploy menerima ZIP
  • Ekstrak file ke /www/wwwroot/hosting/<repo>
  • Buat user SSH khusus untuk kamu
  • Setup Nginx + Systemd service
  • Daftarkan subdomain di Cloudflare
        │
        ▼
Web kamu live di:
  https://<nama-repo>.akhzafachrozy.my.id

Di GitHub Actions → tab Actions → job terbaru → Job Summary
kamu akan menemukan SEMUA informasi + panduan langkah selanjutnya.
```

---

## 📁 File yang Kamu Butuhkan

| File | Fungsi |
|------|--------|
| `deploy.yml` | Diletakkan di `.github/workflows/` pada repo kamu |
| `cloudflaredclient.bat` | Install cloudflared di **Windows** (sekali saja) |
| `ssh.bat` | Login SSH di **Windows** |
| — | macOS/Linux: gunakan terminal biasa (lihat panduan di bawah) |

---

## ⚙️ Setup GitHub Repository

### Langkah 1 — Tambahkan workflow ke repo kamu

Buat folder dan salin file `deploy.yml`:

```
nama-repo-kamu/
├── .github/
│   └── workflows/
│       └── deploy.yml   ← taruh di sini
├── index.php            ← kode kamu
└── ...
```

### Langkah 2 — Pastikan branch `main` atau `master` ada

Workflow hanya berjalan saat ada push ke branch `main` atau `master`.

### Langkah 3 — Push ke GitHub

```bash
git add .
git commit -m "first deploy"
git push origin main
```

---

## 🚀 Cara Deploy Pertama Kali

1. Push kode ke GitHub (langkah di atas).
2. Buka **GitHub → tab Actions → workflow terbaru**.
3. Tunggu hingga semua job selesai (centang hijau ✅).
4. Klik job **Package & Deploy** → lihat **Job Summary**.

Di Job Summary kamu akan menemukan:
- 🌐 Domain web kamu
- 🔑 Username & password SSH
- 📋 **Semua langkah selanjutnya** yang perlu kamu ikuti

> **Selalu baca Job Summary setelah deploy** — semua panduan ada di sana, disesuaikan otomatis dengan repo kamu.

---

## 💻 Langkah 1 — Install cloudflared (sekali saja)

SSH ke server menggunakan **Cloudflare Tunnel**. Kamu perlu install `cloudflared` terlebih dahulu.

### 🪟 Windows

1. Download file `cloudflaredclient.bat` dari github
2. **Klik kanan** → **Run as administrator**
3. Tunggu hingga muncul **"BERHASIL!"**
4. Tutup command prompt

> ⚠️ Jika muncul *"Windows protected your PC"*, klik **More info** → **Run anyway**

### 🍎 macOS

```bash
brew install cloudflare
```

> Belum punya Homebrew? Install dulu di [brew.sh](https://brew.sh)

Verifikasi instalasi:

```bash
cloudflared --version
```

---

## 🔑 Langkah 2 — Login SSH ke Server

Ambil **username dan password SSH** dari **Job Summary** di GitHub Actions.

### 🪟 Windows

1. Download file `ssh.bat` dari github
2. **Klik kanan** → **Run as administrator**
3. Masukkan **username SSH** saat diminta → Enter
4. Masukkan **password SSH** saat diminta → Enter

### 🍎 macOS / 🐧 Linux

Jalankan perintah SSH yang ada di Job Summary, contohnya:

1. Download file `ssh.sh` dari github
2. chmod +x ssh.sh
3. untuk membuka ketik ./ssh.sh
4. Masukkan **username SSH** saat diminta → Enter
5. Masukkan **password SSH** saat diminta → Enter

Jika diminta konfirmasi (pertama kali):
```
Are you sure you want to continue connecting? (yes/no)
```
Ketik `yes` lalu Enter.

> 💡 Password SSH selalu bisa dilihat kembali di GitHub Actions → tab Actions → klik deploy terbaru → **Job Summary**.

---

## 📂 Langkah 3 — Masuk ke Direktori Project

Setelah berhasil masuk SSH, sudah otomatis masuk ke dalam project direktori:

---

## ⚙️ Langkah 4 — Setup Laravel

> Langkah ini hanya perlu dilakukan **sekali** setelah deploy pertama. Update kode selanjutnya cukup push ke GitHub.

### 4a. Install dependencies Composer

```bash
composer install --no-dev --optimize-autoloader
```

### 4b. Salin & edit file `.env`

```bash
cp .env.example .env
nano .env
```

Isi konfigurasi berikut (sesuaikan bagian lain jika perlu):

```env
APP_KEY=               # akan diisi otomatis di langkah 4d
APP_URL=https://nama-repo.akhzafachrozy.my.id
APP_ENV=production
APP_DEBUG=false

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama database
DB_USERNAME=deploysiitk
DB_PASSWORD=deploysiitk2026
```

Simpan file: tekan **Ctrl+X** → ketik **Y** → **Enter**

### 4c. Generate APP_KEY

```bash
php artisan key:generate
```

### 4d. Jalankan migrasi database

```bash
php artisan migrate --force
```

### 4e. Buat symlink storage

```bash
php artisan storage:link
```
### 4f. Install dependencies NPM & build aset (jika ada `package.json`)

```bash
npm install
npm run build
```

### 4g. Restart service

```bash
sudo systemctl restart autodeploy-nama-repo.service
```

### 4h. Verifikasi service berjalan

```bash
sudo systemctl status autodeploy-nama-repo.service
```

Output yang berarti sukses: `● autodeploy-nama-repo.service ... active (running)`

---

## 🎉 Web Kamu Sekarang Live!

Buka di browser: `https://nama-repo.akhzafachrozy.my.id`

---

## 🔄 Cara Update Kode

Cukup push kode baru ke GitHub — proses deploy berjalan otomatis:

```bash
git add .
git commit -m "update fitur baru"
git push origin main
```

Yang **aman** saat update (tidak ditimpa):
- `.env` — konfigurasi kamu tetap aman
- `storage/` — file upload user tetap ada
- `vendor/` — dependensi tetap ada
- `node_modules/` — dependensi NPM tetap ada
- `database/database.sqlite` — data SQLite tetap ada

Jika ada **migrasi database baru**, jalankan via SSH:

```bash
php artisan migrate --force
```

---

## 🛠 Perintah Berguna di Server

```bash
# Lihat status service
sudo systemctl status autodeploy-nama-repo.service

# Restart service (setelah ganti .env dll)
sudo systemctl restart autodeploy-nama-repo.service

# Fix permission satu perintah
sudo fix-perm-nama-repo

# Lihat log aplikasi (live)
tail -f /var/log/autodeploy/nama-repo-app.log

# Lihat log systemd (live)
sudo journalctl -u autodeploy-nama-repo.service -f

# Cek apakah port sudah listening
ss -tlnp | grep :PORT
```

---

## ❓ Troubleshooting

### Web tidak bisa diakses (502 Bad Gateway)

Service belum berjalan atau belum di-setup. Cek dengan:

```bash
sudo systemctl status autodeploy-nama-repo.service
```

Jika status `failed` atau `waiting-setup`, pastikan kamu sudah menjalankan semua langkah setup Laravel di atas.

### SSH gagal terkoneksi

Pastikan `cloudflared` sudah terinstall:

```bash
cloudflared --version
```

Jika tidak ditemukan, ulangi instalasi cloudflared sesuai OS kamu.

### Permission denied saat edit file

```bash
sudo fix-perm-nama-repo
```


### `npm install` atau `npm run build` gagal

Cek versi Node.js:

```bash
node --version
npm --version
```

Jika versi terlalu lama atau tidak tersedia, hubungi dosen terkait

### Lupa password SSH

Buka **GitHub → tab Actions → klik deploy terbaru → Job Summary** — password selalu ditampilkan di sana.

### `.env` tidak ditemukan

File `.env` tidak pernah dikirim ke server (sengaja dikecualikan dari ZIP untuk keamanan). Kamu perlu membuatnya manual via SSH:

```bash
cp .env.example .env
nano .env
```

---

## 📞 Bantuan

Jika masih ada masalah:
1. Cek **Job Summary** di GitHub Actions untuk pesan error
2. Lihat log lengkap via SSH:
   ```bash
   cat /var/log/autodeploy/nama-repo.log
   ```
3. Hubungi dosen terkait dengan menyertakan output log

---

*AutoDeploy v2.3 — Sistem Informasi ITK*
