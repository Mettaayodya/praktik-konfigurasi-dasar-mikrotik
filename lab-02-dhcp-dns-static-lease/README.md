# Lab 02 — DHCP, DNS, dan Static Lease pada MikroTik

## 1. Deskripsi

Praktikum ini merupakan lanjutan dari konfigurasi dasar MikroTik pada Lab 01. Fokus praktikum adalah memahami pengelolaan DHCP Lease, penerapan Static Lease, serta perubahan konfigurasi DNS yang diberikan oleh DHCP Server kepada klien.

Pengujian dilakukan menggunakan MikroTik hAP lite RB941-2nD dan laptop Windows yang terhubung melalui kabel Ethernet. Koneksi internet MikroTik menggunakan hotspot ponsel sebagai jaringan WAN.

## 2. Tujuan Praktikum

- Memeriksa informasi DHCP Lease pada MikroTik.
- Mengubah Dynamic Lease menjadi Static Lease agar perangkat memperoleh alamat IP yang tetap.
- Mengubah DNS Server yang diberikan melalui DHCP Server.
- Memperbarui konfigurasi jaringan pada klien Windows.
- Menguji konektivitas IP dan resolusi nama domain menggunakan CMD.

## 3. Topologi dan Skema IP

**Topologi jaringan:**

```text
Hotspot Ponsel
      |
      v
 wlan1 MikroTik
      |
      v
   LAN MikroTik
      |
      v
 Ethernet Laptop
```

| Parameter | Konfigurasi |
|---|---|
| Router | MikroTik hAP lite RB941-2nD |
| WAN | wlan1, terhubung ke hotspot ponsel |
| LAN | 192.168.88.0/24 |
| IP MikroTik (gateway) | 192.168.88.1 |
| IP laptop | 192.168.88.254 |
| DHCP Server | 192.168.88.1 |
| DNS Server akhir | 8.8.8.8 |

Praktikum melanjutkan konfigurasi jaringan dari Lab 01. Alamat IP dan koneksi WAN yang sudah berfungsi dipertahankan.

## 4. Tahapan Praktikum

### a. Memeriksa DHCP Lease

1. Membuka WinBox dan masuk ke menu **IP → DHCP Server → Leases**.
2. Memeriksa lease yang sesuai dengan laptop berdasarkan alamat IP dan MAC address.
3. Memastikan perangkat memperoleh alamat IP `192.168.88.254`.
4. Mengubah lease dari dynamic menjadi static.

**Hasil:** Lease laptop berubah menjadi static. Alamat IP yang digunakan tetap `192.168.88.254`.

Static Lease digunakan untuk mengikat pemberian alamat IP kepada perangkat berdasarkan identitas klien, umumnya MAC address.

### b. Mengubah DNS Server pada DHCP Networks

1. Membuka menu **IP → DHCP Server → Networks**.
2. Memeriksa konfigurasi jaringan `192.168.88.0/24`.
3. Mengubah DNS Server dari `8.8.8.8` menjadi `1.1.1.1`.
4. Memperbarui konfigurasi DHCP pada laptop menggunakan CMD:

   ```cmd
   ipconfig /renew "Ethernet 2"
   ```

5. Memeriksa konfigurasi yang diterima melalui:

   ```cmd
   ipconfig /all
   ```

**Hasil:** Laptop menerima DNS `1.1.1.1`, sedangkan alamat IP tetap `192.168.88.254`.

Pengujian ini menunjukkan bahwa perubahan parameter DNS melalui DHCP tidak mengharuskan alamat IP laptop berubah.

### c. Menguji Konektivitas dan Resolusi DNS

Konektivitas IP diuji menggunakan:

```cmd
ping 8.8.8.8
```

**Hasil:** Empat paket berhasil diterima dengan 0% packet loss. Ini menunjukkan konektivitas IP ke tujuan pengujian berjalan.

Selanjutnya, resolusi nama domain diuji menggunakan:

```cmd
nslookup google.com 1.1.1.1
nslookup google.com 8.8.8.8
```

**Hasil pengujian:**

- `1.1.1.1`: permintaan DNS mengalami timeout.
- `8.8.8.8`: server DNS merespons dan memberikan hasil resolusi untuk `google.com`.

Karena DNS `1.1.1.1` tidak merespons pada pengujian ini, konfigurasi DNS dikembalikan ke `8.8.8.8`.

Setelah konfigurasi diperbarui melalui DHCP, pengujian ulang `nslookup google.com` berhasil mendapatkan respons dari `dns.google` pada alamat `8.8.8.8`.

Catatan: Hasil resolusi menampilkan alias `forcesafesearch.google.com`. Penyebab pengalihan tersebut belum diperiksa lebih lanjut dalam praktikum ini.

## 5. Hasil dan Analisis

| Pengujian | Hasil |
|---|---|
| Pemeriksaan DHCP Lease | Berhasil |
| Konversi Dynamic Lease menjadi Static Lease | Berhasil |
| Perubahan DNS melalui DHCP Networks | Berhasil |
| Pembaruan konfigurasi DHCP pada laptop | Berhasil |
| Ping ke `8.8.8.8` | Berhasil, 0% packet loss |
| Resolusi DNS melalui `1.1.1.1` | Timeout |
| Resolusi DNS melalui `8.8.8.8` | Berhasil |

Praktikum menunjukkan bahwa konfigurasi alamat IP, gateway, dan DNS merupakan parameter jaringan yang berbeda. Static Lease mempertahankan alokasi IP perangkat, sedangkan DNS yang diberikan kepada klien dapat diperbarui melalui DHCP.

Keberhasilan ping ke `8.8.8.8` bersamaan dengan kegagalan permintaan DNS ke `1.1.1.1` juga menunjukkan pentingnya membedakan pengujian konektivitas IP dari pengujian resolusi nama domain.

Penyebab timeout pada `1.1.1.1` belum dipastikan. Pengujian ini hanya menunjukkan bahwa server tersebut tidak merespons melalui jalur yang digunakan saat praktikum.

## 6. Kesimpulan

Praktikum ini memberikan pengalaman dalam memeriksa dan mengelola DHCP Lease, menerapkan Static Lease, mengubah DNS Server yang didistribusikan melalui DHCP, serta melakukan verifikasi konfigurasi jaringan menggunakan Windows CMD.

Hasil pengujian menegaskan bahwa perubahan DNS dapat diterapkan tanpa mengubah alamat IP yang telah ditetapkan melalui Static Lease. Praktikum ini juga memperkuat pemahaman mengenai perbedaan konektivitas IP dan resolusi DNS.

## 7. Dokumentasi

Screenshot konfigurasi dan hasil pengujian disimpan pada file `dokumentasi` untuk mendukung setiap tahapan praktikum.
