# Tugas Akhir Sistem Operasi via Linux
Sebuah perusahaan ingin membuat sebuah server . Perusahaan ingin server tersebut dapat diakses menggunakan ssh dengan port 9005 dan menggunakan ssh-key sebagai basis authentifikasi tanpa password. Dalam server tersebut terdapat 1 user lain sebagai guest user dengan akses hanya di home directory. Server tersebut harus dapat melayani permintaan website php sederhana dengan file yang telah disediakan dan harus menggunakan https. Server tersebut wajib memiliki 2 storage yaitu storage utama dan backup yang di mount secara permanen.

Mahasiswa dapat memilih Sistem Operasi yang digunakan:
Windows
GNU/Linux
*BSD
Mahasiswa bebas memilih Web Server yang digunakan.

## 1. Install SSH

### Langkah 1: Instalasi OpenSSH Server dan Vim

```bash
sudo apt install openssh-server vim -y
```

## 2. Setting SSH dengan port 9005

### Langkah 2: Konfigurasi File SSH

Edit file konfigurasi SSH:

```bash
sudo vim /etc/ssh/sshd_config
```

Masuk mode edit dengan `i` dan ganti baris `#Port 22` menjadi:

```
Port 9005
```

Simpan dan keluar dari Vim dengan menekan `esc`, lalu ketik `:wq` dan tekan enter.

### Langkah 3: Restart SSH

```bash
sudo systemctl restart ssh
```

## 3. Menggunakan SSH Key sebagai basis autentikasi tanpa password

### Langkah 4: Generate SSH Key di Mesin Client

Buka PowerShell atau Command Prompt di mesin client dan jalankan:

```bash
ssh-keygen -t rsa -b 4096 -C ""
```

### Langkah 5: Copy SSH Key ke Server

Copy key yang telah dihasilkan:

```bash
cd .ssh
```

```bash
type id_rsa.pub
```

Salin key yang ditampilkan, kemudian di server, jalankan:

```bash
sudo mkdir -p /home/vexolune/.ssh
sudo vim /home/vexolune/.ssh/authorized_keys
```

Paste key yang telah disalin, simpan dan keluar dari Vim.

### Langkah 6: Konfigurasi Ulang SSH untuk Autentikasi Kunci Publik

Edit file konfigurasi SSH lagi:

```bash
sudo vim /etc/ssh/sshd_config
```

Pastikan baris berikut tidak dikomentari:

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
```

Simpan dan keluar dari Vim, lalu restart SSH:

```bash
sudo systemctl restart ssh
```

## 4. Membuat Guest User dengan Akses Hanya di Home Directory

### Langkah 7: Tambah User Guest

```bash
sudo adduser guestuser
```

### Langkah 8: Batasi Akses User Guest

Edit file `/etc/ssh/sshd_config`:

```bash
sudo vim /etc/ssh/sshd_config
```

Tambahkan baris berikut di akhir file:

```
Match User guestuser
    ChrootDirectory /home/guestuser
    AllowTCPForwarding no
    X11Forwarding no
```

Simpan dan keluar dari Vim, lalu restart SSH:

```bash
sudo systemctl restart ssh
```

## 5. Melayani Permintaan Website PHP Sederhana

### Langkah 9: Instalasi Nginx, PHP, dan MySQL

Install nginx, php-fpm, php-mysql, openssl.
```bash
sudo apt install nginx php-fpm php-mysql openssl
```

### Langkah 10: Setup Direktori Website

Buat direktori baru di /var/www/html.
```bash
cd /var/www/html
```
```bash
sudo mkdir <nama_project>
```

(Misal `<nama_project>` bernama testing-website, maka gunakan testing-website setiap ada `<nama_project>`. Contoh `cd <nama_project>` maka menjadi `cd testing-website`)

Masuk ke direktori yang telah dibuat

    cd <nama_project>

Copy file php yang sudah disiapkan dari github

    sudo wget https://raw.githubusercontent.com/Rizqirazkafi/testing-website/main/index.php

Kembali ke direktori home dengan cara 

    cd ~

Buat file konfigurasi dengan nama `default.conf` pada `/etc/nginx/conf.d` dengan cara

    cd /etc/nginx/conf.d

lalu

    sudo touch default.conf

Buka file default conf

    sudo vim default.conf

Masuk mode edit dengan klik `i` hingga muncul label `--INSERT--` pada pojok kiri bawah

Copy dan paste konfigurasi berikut

    server {
            listen 80;
            server_name  <ip_address>;

            location / {
                root   /var/www/html/<nama_project>;
                index  index.html index.htm index.php;
            }

    }

(ip_address bisa dilihat pada `ip a`)

Save dan quit dari vim dengan klik `esc` lalu ketik `:wq`

Kembali ke direktori home dengan cara 

    cd ~

Buat file html pada `/var/www/html/<nama_project>` untuk mencoba apakah konfigurasi sudah benar

    cd /var/www/html/<nama_project>

lalu 
    
    sudo touch index.html

Buka file html

    sudo vim index.html

Masuk mode edit dengan klik `i` hingga muncul label `--INSERT--` pada pojok kiri bawah

Copy dan paste kode berikut

    <!doctype html>
    <html>
            <head>
                    <title>
                            Testing website
                    </title>
            </head>

            <body>
                    <h1>Lorem ipsum</h1>
                    <p>Lorem ipsum dolor sit amet</p>
            </body>
    </html>

Save dan quit dari vim dengan klik `esc` lalu ketik `:wq`

Kembali ke direktori home dengan cara 

    cd ~

Restart nginx dengan command 

    sudo systemctl restart nginx

Buka browser lalu ketikkan ip address pada searchbar

### Langkah 11: Konfigurasi Nginx agar bisa membaca file PHP

Cek dulu versi php dengan cara

    sudo systemctl status php 
    
Jangan langsung di enter melainkan di tab (pencet tab pada keyboard)


Lihat pada file `php8.1-fpm.service` (jika php bukan versi 8.1, bisa jadi bernama `php7.4-fpm.service` misal php versi 7.4 atau bisa jadi hanya ada `php-fpm.service`)

Enable php-fpm service (enable biar ga perlu start tiap nyalain ubuntu)

    sudo systemctl enable --now php8.1-fpm
(sesuaikan dengan versi php seperti cara diatas)

Konfigurasi `default.conf`

    sudo vim /etc/nginx/conf.d/default.conf

Masuk mode edit dengan klik `i` hingga muncul label `--INSERT--` pada pojok kiri bawah

Tambahkan konfigurasi berikut dibawah `location`

     location ~ \.php$ {
        root /var/www/html/<nama_project>;
        index index.html index.htm index.php;
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

(nama_project sesuaikan dengan nama direktori yang dipakai untuk menyimpan file php)

Sesuaikan versi php pada `php-fpm.sock` (karena menggunakan php8.1 maka `php8.1-fpm.sock`)

### Langkah 12: Restart Nginx dan PHP-FPM

```bash
sudo systemctl restart nginx php8.1-fpm
```
Lalu pergi ke browser dan coba akses `<ip_address>/index.php` (ip_address sesuaikan)

## 6. Menggunakan HTTPS

### Langkah 13: Generate SSL Certificate

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

Isi fields sesuai instruksi.

### Langkah 14: Konfigurasi Nginx untuk HTTPS

Edit file konfigurasi Nginx:

```bash
sudo vim /etc/nginx/conf.d/default.conf
```

Ubah `listen` dan tambahkan `ssl_certificate`:

```nginx
listen 443 ssl;
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```

Simpan dan keluar dari Vim.

### Langkah 15: Restart Nginx dan PHP-FPM

```bash
sudo systemctl restart nginx php8.1-fpm
```

## 7. Konfigurasi 2 Storage

### Langkah 16: Mount Storage Backup

Lebih lengkapnya dapat dilihat disini: 

[5-Partition File System][(https://www.google.com](https://github.com/Rizqirazkafi/sistem-operasi-2024/blob/main/Linux-Path/5-partiton-filesystem/main.md))

Buat direktori untuk storage Backup:

```bash
mkdir backup
```

Edit file `/etc/fstab` untuk mount secara permanen:

```bash
sudo vim /etc/fstab
```

Tambahkan baris berikut:

```plaintext
/dev/sdb1 /home/vexolune/backup ext4 defaults 0 1
```

Simpan dan keluar dari Vim, lalu mount semua partisi yang tercantum di `/etc/fstab`:

```bash
sudo mount -a
```

Untuk mengecek apakah berhasil coba cek

```bash
lsblk
```

