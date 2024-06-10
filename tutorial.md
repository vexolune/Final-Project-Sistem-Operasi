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

Buka PowerShell atau terminal di mesin client dan jalankan:

```bash
ssh-keygen -t rsa -b 4096 -C ""
```

### Langkah 5: Copy SSH Key ke Server

Copy key yang telah dihasilkan:

```bash
cd ~/.ssh
cat id_rsa.pub
```

Salin key yang ditampilkan, kemudian di server, jalankan:

```bash
sudo mkdir -p /home/vexolune/.ssh
sudo vim /home/vexolune/.ssh/authorized_keys
```

Paste key yang telah disalin, simpan dan keluar dari Vim.

Setel izin direktori dan file:

```bash
sudo chown -R vexolune:vexolune /home/vexolune/.ssh
sudo chmod 700 /home/vexolune/.ssh
sudo chmod 600 /home/vexolune/.ssh/authorized_keys
```

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

```bash
sudo apt install nginx php-fpm php-mysql -y
```

### Langkah 10: Setup Direktori Website

```bash
sudo mkdir -p /var/www/html/testing-website
cd /var/www/html/testing-website
sudo wget https://raw.githubusercontent.com/Rizqirazkafi/testing-website/main/index.php
```

### Langkah 11: Konfigurasi Nginx

Buat file konfigurasi untuk website:

```bash
sudo vim /etc/nginx/conf.d/default.conf
```

Masuk mode edit dengan `i`, lalu tambahkan:

```nginx
server {
    listen 80;
    server_name <ip_address>;

    location / {
        root   /var/www/html/testing-website;
        index  index.html index.htm index.php;
    }

    location ~ \.php$ {
        root /var/www/html/testing-website;
        index index.html index.htm index.php;
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
```

Ganti `<ip_address>` dengan alamat IP server. Simpan dan keluar dari Vim.

### Langkah 12: Restart Nginx dan PHP-FPM

```bash
sudo systemctl restart nginx php8.1-fpm
```

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

### Langkah 16: Mount Storage Utama dan Backup

Buat direktori untuk mount point:

```bash
sudo mkdir /mnt/storage
sudo mkdir /mnt/backup
```

Edit file `/etc/fstab` untuk mount secara permanen:

```bash
sudo vim /etc/fstab
```

Tambahkan baris berikut (sesuaikan dengan partisi storage Anda, misalnya `/dev/sdb1` untuk storage utama dan `/dev/sdc1` untuk backup):

```plaintext
/dev/sdb1 /mnt/storage ext4 defaults 0 2
/dev/sdc1 /mnt/backup ext4 defaults 0 2
```

Simpan dan keluar dari Vim, lalu mount semua partisi yang tercantum di `/etc/fstab`:

```bash
sudo mount -a
```
