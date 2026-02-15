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

## Rekomendasi
1. Tutup port Telnet (23) pada gateway (36.88.105.225) dan gunakan SSH untuk keamanan.
2. Lakukan audit pada layanan Webmin yang tersebar di banyak server.
3. Identifikasi perangkat yang terhubung melalui interface `enx42f2e93361aa` karena jalur ini melewati konfigurasi jaringan utama.
