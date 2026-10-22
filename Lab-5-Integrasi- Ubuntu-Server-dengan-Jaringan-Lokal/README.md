# Lab 5 — Integrasi Ubuntu Server dan SSH pada Jaringan Lokal

## 1. Deskripsi

Praktikum ini berfokus pada integrasi Ubuntu Server 24.04.4 yang berjalan pada VMware Workstation ke jaringan lokal MikroTik, pengelolaan alamat IP menggunakan DHCP Static Lease, serta konfigurasi layanan SSH agar server dapat diakses dari laptop Windows.

## 2. Tujuan Praktikum

- Mengintegrasikan Ubuntu Server ke jaringan lokal MikroTik.
- Memeriksa alamat IP, gateway, dan konektivitas jaringan Ubuntu Server.
- Memastikan layanan SSH berjalan dan terhubung dengan port 22.
- Mengonfigurasi UFW untuk mengizinkan koneksi SSH dari jaringan lokal.
- Melakukan remote access dari Windows menggunakan PuTTY.
- Memeriksa status koneksi TCP dan log autentikasi SSH.

## 3. Perangkat dan Lingkungan Praktikum

| Komponen | Konfigurasi |
|---|---|
| Router | MikroTik hAP lite RB941-2nD |
| Virtualisasi | VMware Workstation |
| Sistem operasi server | Ubuntu Server 24.04.4 |
| Sistem operasi klien | Windows |
| Aplikasi SSH client | PuTTY |
| Firewall Ubuntu | UFW |
| Antarmuka jaringan Ubuntu | `ens33` |
| Mode jaringan VM | Bridged |
| Alamat IP MikroTik | `192.168.88.1/24` |
| Alamat IP Windows | `192.168.88.254/24` |
| Alamat IP Ubuntu Server | `192.168.88.252/24` |
| Gateway Ubuntu Server | `192.168.88.1` |
| Port SSH | TCP 22 |

Koneksi internet MikroTik menggunakan hotspot ponsel melalui antarmuka `wlan1`. Laptop Windows terhubung ke jaringan lokal MikroTik melalui Ethernet.

## 4. Topologi Jaringan

```text
Internet
   |
Hotspot Ponsel
   |
wlan1 — MikroTik hAP lite
   |
bridgeLocal (192.168.88.1/24)
   |
   +--- Laptop Windows
   |    IP: 192.168.88.254
   |    SSH Client: PuTTY
   |
   +--- Ubuntu Server VM
        IP: 192.168.88.252
        Gateway: 192.168.88.1
        SSH Server: TCP 22
        Firewall: UFW
```

Ubuntu Server menggunakan mode jaringan Bridged pada VMware sehingga dapat terhubung ke jaringan lokal yang sama dengan laptop Windows.

## 5. Tahapan Praktikum

### 5.1 Memeriksa Jaringan Ubuntu Server

Ubuntu Server dijalankan melalui VMware Workstation dengan adapter jaringan dalam mode Bridged.

Pemeriksaan alamat IP dilakukan menggunakan:

```bash
ip address
```

Informasi routing diperiksa menggunakan:

```bash
ip route
```

Hasil pemeriksaan menunjukkan bahwa Ubuntu Server menggunakan antarmuka `ens33` dengan alamat IP `192.168.88.252/24` dan gateway `192.168.88.1`.

Konektivitas ke router diuji menggunakan:

```bash
ping -c 4 192.168.88.1
```

Konektivitas ke alamat IP internet diuji menggunakan:

```bash
ping -c 4 8.8.8.8
```

**Hasil:** Ubuntu Server berhasil berkomunikasi dengan MikroTik dan memperoleh konektivitas IP ke tujuan pengujian di internet.

### 5.2 Mengatur DHCP Static Lease pada MikroTik

Agar alamat IP Ubuntu Server tetap, DHCP Lease dikonfigurasi menjadi Static Lease pada MikroTik.

Informasi klien:

| Parameter | Nilai |
|---|---|
| Hostname | `metta` |
| MAC Address | `00:0C:29:F4:16:6B` |
| IP Address | `192.168.88.252` |
| Status lease | Bound |

Static Lease digunakan agar MikroTik memberikan alamat IP yang sama kepada klien berdasarkan identitas DHCP-nya, dalam konfigurasi ini berdasarkan MAC address.

### 5.3 Memeriksa Layanan SSH

Status layanan SSH diperiksa menggunakan:

```bash
sudo systemctl status ssh
```

 

```bash
sudo ss -tlnp | grep ':22'
```
Hasil pemeriksaan menunjukkan bahwa port TCP 22 mendengarkan pada alamat:

```text
0.0.0.0:22
[::]:22
```

Alamat `0.0.0.0:22` menunjukkan bahwa SSH mendengarkan pada seluruh alamat antarmuka IPv4 yang sesuai. `[::]:22` menunjukkan listener IPv6.

Pemeriksaan konfigurasi SSH dilakukan menggunakan:

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin|passwordauthentication)'
```

Hasil yang diperoleh:

```text
port 22
permitrootlogin without-password
passwordauthentication yes
```

Konfigurasi tersebut menunjukkan bahwa SSH menggunakan port 22, autentikasi password diizinkan, dan login root langsung menggunakan password tidak diizinkan.

### 5.4 Mengonfigurasi Firewall UFW

Sebelum konfigurasi firewall, status UFW diperiksa menggunakan:

```bash
sudo ufw status verbose
```

Aturan ditambahkan agar koneksi SSH hanya diizinkan dari jaringan lokal `192.168.88.0/24`:

```bash
sudo ufw allow from 192.168.88.0/24 to any port 22 proto tcp
```

UFW kemudian diaktifkan:

```bash
sudo ufw enable
```

Status firewall diperiksa kembali:

```bash
sudo ufw status verbose
```

Hasil konfigurasi menunjukkan bahwa UFW aktif dengan kebijakan default incoming `deny` dan outgoing `allow`. Akses TCP port 22 diizinkan dari jaringan `192.168.88.0/24`.

### 5.5 Mengakses Ubuntu Server melalui PuTTY

Koneksi SSH dilakukan dari laptop Windows menggunakan PuTTY.

Parameter koneksi:

| Parameter | Nilai |
|---|---|
| Host Name | `192.168.88.252` |
| Port | `22` |
| Connection type | SSH |
| Username | `metta` |

### 5.6 Memeriksa Koneksi TCP dan Log SSH

Koneksi TCP yang sedang berstatus established diperiksa menggunakan:

```bash
sudo ss -tnp state established
```

Perintah ini menampilkan koneksi TCP yang sudah terbentuk pada saat pemeriksaan.

Log autentikasi SSH diperiksa menggunakan:

```bash
sudo journalctl -u ssh --since "30 minutes ago"
```

Pada pengujian, log menampilkan informasi autentikasi berhasil dari alamat klien `192.168.88.254` untuk akun `metta`.

Contoh informasi yang diperoleh:

```text
Accepted password for metta from 192.168.88.254
```

**Hasil:** Log mengonfirmasi bahwa autentikasi SSH dari laptop Windows berhasil.

## 6. Hasil Pengujian

| Pengujian | Hasil |
|---|---|
| Ubuntu Server memperoleh alamat IP | Berhasil |
| Gateway Ubuntu Server dapat dijangkau | Berhasil |
| Konektivitas IP ke `8.8.8.8` | Berhasil |
| DHCP Static Lease pada MikroTik | Berhasil |
| SSH mendengarkan pada port 22 | Berhasil |
| UFW mengizinkan SSH dari jaringan lokal | Berhasil |
| Remote access melalui PuTTY | Berhasil |
| Log autentikasi SSH | Menunjukkan autentikasi berhasil |

## 7. Analisis

Praktikum ini menunjukkan bahwa Ubuntu Server yang berjalan sebagai virtual machine dapat terhubung ke jaringan lokal MikroTik menggunakan mode Bridged pada VMware Workstation.

DHCP Static Lease membantu mempertahankan alokasi alamat IP server sehingga klien dapat menggunakan alamat tujuan yang konsisten untuk remote access. Layanan SSH memungkinkan administrasi server dilakukan dari laptop Windows tanpa harus berinteraksi langsung dengan konsol virtual machine.

UFW digunakan untuk membatasi koneksi masuk ke port SSH berdasarkan jaringan sumber. Keberhasilan koneksi PuTTY dan catatan autentikasi pada log menjadi bukti bahwa jalur koneksi dan proses autentikasi berhasil pada saat pengujian.

Pemeriksaan `LISTEN`, `ESTABLISHED`, dan log SSH memberikan informasi yang berbeda sehingga dapat digunakan bersama-sama dalam proses troubleshooting.

## 8. Kesimpulan

Praktikum Lab 5 berhasil mengintegrasikan Ubuntu Server 24.04.4 ke jaringan lokal MikroTik, menetapkan alamat IP melalui DHCP Static Lease, serta menyediakan akses administrasi jarak jauh menggunakan SSH.

Pengujian melalui PuTTY membuktikan bahwa laptop Windows dapat mengakses Ubuntu Server pada `192.168.88.252` melalui TCP port 22. Aturan UFW membatasi koneksi SSH masuk agar hanya diizinkan dari jaringan lokal yang telah ditentukan.

Praktikum ini memberikan pemahaman dasar mengenai jaringan virtual, DHCP, layanan SSH, firewall host, serta penggunaan log dan status koneksi untuk verifikasi layanan.
