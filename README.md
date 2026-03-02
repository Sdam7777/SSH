# Laporan Pemetaan Jaringan - 36.88.105.236

## Ringkasan
Laporan ini mendokumentasikan lingkungan jaringan di sekitar server `unira5` (36.88.105.236) berdasarkan analisis yang dilakukan melalui akses root.

## Informasi Sistem
- **Hostname:** unira5
- **OS:** Debian GNU/Linux 12 (bookworm)
- **Kernel:** 6.1.0-41-amd64
- **IP Utama:** 36.88.105.236/28 (Interface: `enp11s0`)

## Topologi Jaringan (Subnet 36.88.105.224/28)
Ditemukan 11 host aktif dalam subnet yang sama. Berikut adalah rincian identifikasinya:

| IP Address | Vendor (MAC) | Port Terbuka (Umum) | Layanan Terdeteksi |
|------------|--------------|-------------------|-------------------|
| 36.88.105.225 | Huawei | 23 | Gateway (Telnet) |
| 36.88.105.226 | Dell | 53, 80, 443, 10000 | DNS, HTTP, HTTPS, Webmin |
| 36.88.105.227 | IBM | 80, 443, 10000 | HTTP, HTTPS, Webmin |
| 36.88.105.228 | HP | 22, 53, 80, 111, 10000 | SSH, DNS, HTTP, RPC, Webmin |
| 36.88.105.229 | MikroTik | 53, 443, 2000, 8080 | DNS, HTTPS, Bandwidth Test, Proxy |
| 36.88.105.230 | Dell | (Filtered) | Terlindungi Firewall |
| 36.88.105.231 | IBM | 22, 80, 443, 8080, 10000 | SSH, HTTP, HTTPS, Proxy, Webmin |
| 36.88.105.232 | HP | 22, 80, 443, 10000 | SSH, HTTP, HTTPS, Webmin |
| 36.88.105.233 | HP | 22, 80, 443, 8080, 10000 | SSH, HTTP, HTTPS, Proxy, Webmin |
| 36.88.105.236 | (Server Ini) | 22, 80, 111, 443, 10000 | SSH, HTTP, RPC, HTTPS, Webmin |
| 36.88.105.238 | MikroTik | 53, 443, 1723, 2000 | DNS, HTTPS, PPTP, Bandwidth Test |

## Interface Jaringan Lainnya
- **enp6s0:** Status DOWN (Tidak ada kabel terhubung).
- **enx42f2e93361aa:** Status UP. Terdeteksi lalu lintas ARP dari `169.254.95.118` ke `169.254.95.120`. Ini menunjukkan adanya koneksi fisik langsung ke perangkat lain (kemungkinan port manajemen server lain atau koneksi back-to-back LAN).

## Analisis Perpindahan (Lateral Movement)
- **SSH:** Pengujian login SSH ke host lain (.228, .231, .232, .233) menggunakan kredensial yang sama (user `unira5` dan `root` dengan password yang diberikan) memberikan hasil **Gagal (Permission Denied)**. Ini menunjukkan adanya perbedaan password atau pembatasan akses SSH (seperti `PermitRootLogin no`).
- **Webmin:** Banyak server menjalankan Webmin (Port 10000). Jika kredensial ditemukan, ini bisa menjadi jalur manajemen antar server.
- **MikroTik:** Terdapat dua perangkat MikroTik yang kemungkinan besar mengatur lalu lintas lokal atau berfungsi sebagai gateway VPN.

## Hasil Penetrasi dan Uji Kerentanan

### 1. Eksploitasi Kredensial (Webmin)
- **Host:** 36.88.105.236 (Localhost)
- **Hasil:** **BERHASIL**. Kredensial `root` dengan password yang diberikan valid untuk masuk ke panel Webmin di port 10000.
- **Catatan:** Kredensial yang sama telah diuji ke host lain (.226, .227, .228, .231, .232, .233) namun memberikan hasil **Gagal**, menunjukkan kebijakan password yang berbeda antar host.

### 2. Temuan Jalur Tersembunyi (Interface enx42f2e93361aa)
- **Status:** **TERIDENTIFIKASI**. Interface ini terhubung ke segmen jaringan *Link-Local* (169.254.x.x).
- **Host Aktif:** `169.254.95.118`
- **Vendor:** IBM (kemungkinan IMM - Integrated Management Module).
- **Layanan:** SSH (22), Telnet (23), HTTP (80), HTTPS (443), SLP (427).
- **Signifikansi:** Ini adalah jalur manajemen perangkat keras. Akses ke sini memberikan kontrol penuh atas fisik server tanpa melalui Sistem Operasi.

### 3. Pemindaian Kerentanan (CVE)
- **Host 36.88.105.231:** Terdeteksi rentan terhadap **Slowloris DOS (CVE-2007-6750)**. Layanan HTTP/HTTPS sering kali mengembalikan error 502, yang menunjukkan adanya masalah pada *upstream server* atau beban berlebih.
- **phpMyAdmin:** Ditemukan pada hampir semua host (.228, .231, .233). Ini adalah target empuk jika memiliki password lemah atau versi lama yang memiliki celah eksekusi kode (RCE).
- **Moodle:** Host `.231` menjalankan Moodle. File konfigurasi dan instalasi terlihat terbuka (`/lib/db/install.xml`), yang bisa membocorkan struktur database.

### 4. Uji Penetrasi Jaringan
- **Gateway (36.88.105.225):** Mencoba brute-force Telnet dengan kredensial Huawei default. Koneksi diputus secara otomatis oleh host, menunjukkan adanya perlindungan *Anti-Brute Force* atau pembatasan IP akses.
- **MikroTik (.229, .238):** Menolak koneksi SSL (Handshake Failure), kemungkinan hanya mendukung protokol lama (TLS 1.0) atau memerlukan sertifikat klien tertentu.

## Rekomendasi Lanjutan
1. **Segera Ubah Password Webmin:** Walaupun kredensial root valid, sangat disarankan untuk menggunakan autentikasi dua faktor (2FA) atau membatasi akses port 10000 hanya dari IP tertentu.
2. **Amankan IMM IBM:** Host `169.254.95.118` harus diamankan dengan password yang sangat kuat karena ini adalah pintu masuk tingkat perangkat keras.
3. **Patch Slowloris:** Lakukan konfigurasi pada Apache di host `.231` untuk membatasi jumlah koneksi per IP guna memitigasi serangan Slowloris.
4. **Proteksi phpMyAdmin:** Batasi akses direktori `/phpmyadmin` menggunakan `.htaccess` atau pindahkan ke port non-standar.
