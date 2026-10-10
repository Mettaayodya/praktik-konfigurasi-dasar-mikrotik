# Praktik Konfigurasi Dasar MikroTik hAP lite

## 1. Gambaran Umum

Proyek ini merupakan praktik konfigurasi jaringan menggunakan MikroTik hAP lite (RB941-2nD). Praktik ini dilakukan untuk memahami dasar pengelolaan jaringan, mulai dari menghubungkan router ke internet melalui hotspot HP hingga menyediakan koneksi internet untuk perangkat laptop melalui jaringan lokal.

Konfigurasi yang diterapkan meliputi koneksi WAN melalui Wi-Fi, pengaturan LAN menggunakan bridge, DHCP Server, serta Network Address Translation (NAT).

## 2. Tujuan

- Menghubungkan MikroTik ke internet melalui hotspot HP.
- Mengonfigurasi jaringan lokal menggunakan bridge.
- Menetapkan alamat IP gateway untuk jaringan LAN.
- Mengatur DHCP Server agar perangkat klien memperoleh alamat IP secara otomatis.
- Mengonfigurasi NAT agar perangkat LAN dapat mengakses internet.
- Menguji konektivitas jaringan lokal dan internet.

## 3. Perangkat yang Digunakan

| Perangkat | Fungsi |
|---|---|
| MikroTik hAP lite (RB941-2nD) | Router untuk menghubungkan jaringan lokal dengan internet |
| HP | Menyediakan koneksi internet melalui hotspot |
| Laptop | Perangkat klien untuk pengujian jaringan |
| Kabel LAN | Menghubungkan laptop dengan MikroTik |

## 4. Topologi Jaringan

![Topologi Jaringan MikroTik](topologi-jaringan-mikrotik.png)

Gambaran topologi jaringan dalam bentuk teks:

    Hotspot HP
         |
       Wi-Fi
         |
    wlan1 (WAN)
    172.25.164.157/24
         |
    MikroTik hAP lite
         |
    bridge (LAN)
    192.168.88.1/24
         |
       ether2
         |
       Laptop
    192.168.88.254
         |
    Akses Internet

**Keterangan:**

- `wlan1` berfungsi sebagai antarmuka WAN yang terhubung ke hotspot HP.
- `bridge` berfungsi sebagai antarmuka jaringan lokal atau LAN.
- `ether2` menghubungkan laptop ke jaringan lokal melalui bridge.
- NAT digunakan agar perangkat di jaringan lokal dapat mengakses internet melalui koneksi WAN MikroTik.

## 5. Konfigurasi Jaringan

| Komponen | Konfigurasi |
|---|---|
| Antarmuka WAN | `wlan1` |
| Metode koneksi WAN | Wi-Fi Client |
| Perolehan IP WAN | DHCP Client |
| Alamat IP WAN | `172.25.164.157/24` |
| Antarmuka LAN | `bridge` |
| Alamat IP gateway LAN | `192.168.88.1/24` |
| Alamat IP laptop | `192.168.88.254` |
| DHCP Server | Berjalan pada `bridge` |
| NAT Chain | `srcnat` |
| NAT Out. Interface | `wlan1` |
| NAT Action | `masquerade` |

**Catatan:** Alamat IP WAN dan laptop di atas merupakan alamat yang diperoleh saat praktik dilakukan.

## 6. Tahapan Implementasi

### 6.1. Konfigurasi WAN

Mengonfigurasi `wlan1` sebagai antarmuka Wi-Fi Client dan menghubungkannya ke hotspot HP. DHCP Client diaktifkan pada `wlan1` agar MikroTik memperoleh alamat IP secara otomatis dari jaringan hotspot.

### 6.2. Konfigurasi LAN

Menggunakan antarmuka `bridge` sebagai gateway jaringan lokal dengan alamat IP `192.168.88.1/24`. Port Ethernet yang menjadi anggota bridge dapat digunakan untuk menghubungkan perangkat klien ke jaringan lokal.

### 6.3. Konfigurasi DHCP Server

Mengaktifkan DHCP Server pada `bridge` agar perangkat klien memperoleh alamat IP secara otomatis tanpa perlu melakukan konfigurasi IP secara manual.

### 6.4. Konfigurasi NAT

Membuat aturan NAT dengan parameter berikut:

| Parameter | Nilai |
|---|---|
| Chain | `srcnat` |
| Out. Interface | `wlan1` |
| Action | `masquerade` |

Aturan ini memungkinkan alamat IP sumber dari perangkat di jaringan lokal diterjemahkan menggunakan alamat IP antarmuka WAN saat paket keluar melalui `wlan1`.

Dengan demikian, laptop dapat mengakses internet melalui koneksi WAN MikroTik.

## 7. Pengujian dan Hasil

| Pengujian | Hasil |
|---|---|
| MikroTik memperoleh alamat IP WAN melalui DHCP Client | Berhasil |
| MikroTik dapat melakukan ping ke `8.8.8.8` | Berhasil |
| Laptop memperoleh alamat IP dari DHCP Server | Berhasil |
| Laptop dapat melakukan ping ke gateway `192.168.88.1` | Berhasil |
| Laptop dapat melakukan ping ke `8.8.8.8` | Berhasil |
| Laptop dapat melakukan resolusi DNS dan ping ke `google.com` | Berhasil |

Hasil pengujian menunjukkan bahwa koneksi WAN, jaringan LAN, DHCP Server, dan NAT telah berfungsi sesuai dengan tujuan praktik.

## 8. Dokumentasi

Dokumentasi konfigurasi dan pengujian dapat dilihat pada berkas berikut:

- **Topologi jaringan:** 
- **Dokumentasi konfigurasi:** tangkapan layar konfigurasi dan hasil pengujian yang disimpan dalam folder dokumentasi.

## 9. Kompetensi yang Dipraktikkan

- Konfigurasi dasar MikroTik RouterOS.
- Pemahaman peran WAN dan LAN.
- Konfigurasi DHCP Client dan DHCP Server.
- Konfigurasi source NAT menggunakan masquerade.
- Pengujian konektivitas dan troubleshooting dasar.

## 10. Batasan Praktik

Praktik ini berfokus pada konfigurasi routing dasar, DHCP, dan NAT. Konfigurasi lanjutan seperti firewall, VLAN, manajemen bandwidth, dan VPN belum termasuk dalam cakupan proyek ini.

## 11. Kesimpulan

Praktik ini berhasil membangun koneksi jaringan dari hotspot HP menuju MikroTik dan meneruskannya ke laptop melalui jaringan lokal. Konfigurasi yang dilakukan memberikan pemahaman praktis mengenai hubungan antara WAN, LAN, DHCP, dan NAT dalam membangun konektivitas jaringan dasar.
