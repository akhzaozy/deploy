# 🚀 AutoDeploy v2.3 — Panduan Instalasi

> Platform auto-hosting untuk mahasiswa ITK. Push ke GitHub → otomatis live di server.

---

## 📋 Daftar Isi

1. [Gambaran Sistem](#-gambaran-sistem)
2. [File yang Kamu Butuhkan](#-file-yang-kamu-butuhkan)
3. [Setup GitHub Repository](#-setup-github-repository)
4. [Cara Deploy Pertama Kali](#-cara-deploy-pertama-kali)
5. [Cara Login SSH dari Windows](#-cara-login-ssh-dari-windows)
6. [Setup Laravel (setelah deploy pertama)](#-setup-laravel-setelah-deploy-pertama)
7. [Cara Update Kode](#-cara-update-kode)
8. [Perintah Berguna di Server](#-perintah-berguna-di-server)
9. [Troubleshooting](#-troubleshooting)

---

## 🗺 Gambaran Sistem

```
Kamu push kode ke GitHub
        │
        ▼
GitHub Actions berjalan otomatis
  • Deteksi framework (Laravel / PHP Native)
  • Buat ZIP dari kode kamu
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

Kamu bisa SSH masuk untuk:
  • Install composer / npm
  • Setup file .env
  • Jalankan migrasi database
```

---

## 📁 File yang Kamu Butuhkan

| File | Fungsi |
|------|--------|
| `deploy.yml` | Diletakkan di `.github/workflows/` pada repo kamu |
| `cloudflaredclient.bat` | Install cloudflared di Windows (sekali saja) |
| `ssh.bat` | Login SSH ke server dari Windows |

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

Buka tab **Actions** di GitHub untuk melihat progress deploy.

---

## 🚀 Cara Deploy Pertama Kali

1. Push kode ke GitHub (langkah di atas).
2. Buka **GitHub → tab Actions → workflow terbaru**.
3. Tunggu hingga semua job selesai (centang hijau ✅).
4. Klik job **Package & Deploy** → lihat **Job Summary**.
5. Di summary kamu akan melihat:

```
Domain   : https://nama-repo.akhzafachrozy.my.id
SSH User : nama-repo
SSH Pass : nama-repo<3xxx
SSH Cmd  : ssh nama-repo@IP-SERVER
```

Simpan informasi ini, kamu butuhkan untuk login SSH.

---

## 💻 Cara Login SSH dari Windows

SSH ke server menggunakan **Cloudflare Tunnel** — kamu tidak perlu tahu IP server.

### Langkah 1 — Install cloudflared (sekali saja)

1. **Klik kanan** file `cloudflaredclient.bat` → **Run as administrator**
2. Tunggu proses download dan instalasi selesai
3. Tutup command prompt setelah muncul "BERHASIL!"

> ⚠️ Jika muncul "Windows protected your PC", klik **More info** → **Run anyway**

### Langkah 2 — Login SSH

1. **Klik kanan** file `ssh.bat` → **Run as administrator**
2. Masukkan **username SSH** dari output GitHub Actions
3. Tekan Enter, lalu masukkan **password SSH** saat diminta
4. Kamu sekarang berada di dalam server! 🎉

```
Contoh:
Username SSH: nama-repo
Password SSH: nama-repo<3123
```

---

## 🔧 Setup Laravel (setelah deploy pertama)

Setelah berhasil SSH masuk, jalankan langkah-langkah berikut:

```bash
# 1. Masuk ke folder project
cd /www/wwwroot/hosting/nama-repo

# 2. Install dependencies Composer
/www/server/php/83/bin/php /usr/local/bin/composer install --no-dev --optimize-autoloader

# 3. Salin dan edit file .env
cp .env.example .env
nano .env
```

Isi `.env` minimal seperti ini:
```env
APP_KEY=              # diisi langkah 4
APP_URL=https://nama-repo.akhzafachrozy.my.id
APP_ENV=production
APP_DEBUG=false
DB_CONNECTION=sqlite  # atau mysql jika pakai MySQL
```

```bash
# 4. Generate APP_KEY
/www/server/php/83/bin/php artisan key:generate

# 5. Jalankan migrasi database
/www/server/php/83/bin/php artisan migrate --force

# 6. Buat symlink storage
/www/server/php/83/bin/php artisan storage:link

# 7. Fix permission (wajib!)
sudo fix-perm-nama-repo

# 8. Aktifkan web
sudo systemctl restart autodeploy-nama-repo.service

# 9. Verifikasi
sudo systemctl status autodeploy-nama-repo.service
```

Web kamu sekarang live di `https://nama-repo.akhzafachrozy.my.id` 🎉

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
- `database/database.sqlite` — data tetap ada

Jika ada **migrasi database baru**, jalankan via SSH:
```bash
cd /www/wwwroot/hosting/nama-repo
/www/server/php/83/bin/php artisan migrate --force
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

Service belum berjalan. Cek dengan:
```bash
sudo systemctl status autodeploy-nama-repo.service
```
Jika status `failed` atau `waiting-setup`, jalankan setup Laravel dulu (langkah di atas).

### SSH gagal terkoneksi

Pastikan `cloudflared` sudah terinstall:
```
cloudflared --version
```
Jika tidak ditemukan, jalankan ulang `cloudflaredclient.bat`.

### Permission denied saat edit file

Jalankan:
```bash
sudo fix-perm-nama-repo
```

### composer install gagal

Pastikan kamu berada di folder yang benar:
```bash
pwd
# harus: /www/wwwroot/hosting/nama-repo
```

### Lupa password SSH

Buka GitHub Actions → tab **Actions** → klik deploy terbaru → **Job Summary** — password selalu ditampilkan di sana.

---

## 📞 Bantuan

Jika masih ada masalah, hubungi asisten praktikum atau lihat log lengkap:
```bash
cat /var/log/autodeploy/nama-repo.log
```

---

*AutoDeploy v2.3 — Sistem Informasi ITK*
