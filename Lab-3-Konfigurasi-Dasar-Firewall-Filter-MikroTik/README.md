# Lab 3 — Konfigurasi Dasar Firewall Filter MikroTik

## 1. Deskripsi

Lab ini membahas konfigurasi dasar Firewall Filter pada MikroTik RouterOS untuk memahami cara kerja penyaringan paket jaringan menggunakan chain `input`, `forward`, dan `output`.

Praktik dilakukan menggunakan MikroTik hAP lite RB941-2nD dengan koneksi internet dari hotspot HP dan laptop Windows yang terhubung melalui kabel Ethernet.

Pengujian dilakukan dengan membuat rule `accept` dan `drop` untuk protokol ICMP, kemudian mengamati hasil konektivitas dan perubahan packet counter pada setiap rule.

## 2. Tujuan

- Memahami fungsi Firewall Filter pada MikroTik.
- Membedakan chain `input`, `forward`, dan `output`.
- Memahami action `accept` dan `drop`.
- Mengetahui pengaruh urutan rule terhadap pemrosesan paket.
- Menguji hasil konfigurasi firewall melalui ping dan packet counter.
- Memastikan akses manajemen router tetap berfungsi selama pengujian.

## 3. Perangkat dan Software

| Komponen | Keterangan |
|---|---|
| Router | MikroTik hAP lite RB941-2nD |
| Komputer | Laptop Windows |
| Koneksi WAN | Hotspot HP |
| Aplikasi konfigurasi | WinBox |
| Pengujian konektivitas | Command Prompt Windows dan Terminal MikroTik |
| Interface WAN | `wlan1` |
| Interface LAN | `bridgeLocal` |
| Port LAN laptop | `ether2` |

## 4. Topologi Jaringan

```text
            Hotspot HP
                 |
                 | Wi-Fi
                 |
              wlan1
          +-------------+
          |   MikroTik  |
          |   hAP lite  |
          |             |
          | bridgeLocal |
          +-------------+
                 |
               ether2
                 |
                 |
          +-------------+
          |   Laptop    |
          |   Windows   |
          +-------------+
```

### Alamat IP

| Perangkat | Alamat IP |
|---|---|
| MikroTik LAN | `192.168.88.1/24` |
| Laptop | `192.168.88.254/24` |
| Gateway laptop | `192.168.88.1` |
| Tujuan pengujian internet | `8.8.8.8` |

## 5. Dasar Teori

### 5.1 Firewall Filter

Firewall Filter merupakan fitur MikroTik yang digunakan untuk memeriksa paket jaringan dan menentukan tindakan berdasarkan kondisi rule yang dikonfigurasi.

### 5.2 Chain Firewall

| Chain | Fungsi | Contoh |
|---|---|---|
| `input` | Memproses paket yang ditujukan ke router | Ping dari laptop ke `192.168.88.1` |
| `forward` | Memproses paket yang melewati router | Ping dari laptop ke `8.8.8.8` |
| `output` | Memproses paket yang dibuat oleh router | Ping ke `8.8.8.8` dari Terminal MikroTik |

### 5.3 Action

- `accept`: mengizinkan paket melewati rule.
- `drop`: membuang paket tanpa mengirimkan respons penolakan.

Rule diperiksa berdasarkan urutannya. Jika paket cocok dengan rule yang menghasilkan keputusan akhir seperti `accept` atau `drop`, paket tidak melanjutkan pemeriksaan ke rule berikutnya dalam chain tersebut.

### 5.4 Protocol ICMP

ICMP digunakan untuk berbagai pesan diagnostik jaringan, termasuk permintaan dan balasan ping. Pada praktik ini, ICMP digunakan untuk menguji pengaruh Firewall Filter terhadap konektivitas.

## 6. Langkah Konfigurasi dan Pengujian

### 6.1 Pemeriksaan Awal

Sebelum konfigurasi firewall dilakukan, pemeriksaan awal menunjukkan:

- Filter Rules dalam kondisi kosong.
- Laptop dapat mengakses internet.
- WinBox terhubung melalui `ether2`.
- Interface `ether2` merupakan anggota bridge `bridgeLocal`.
- IP `192.168.88.1/24` terpasang pada `bridgeLocal`.

Backup konfigurasi dibuat sebelum memulai pengujian.

### 6.2 Pengujian Chain Input

Rule pertama dibuat untuk mengizinkan ICMP yang masuk melalui LAN dan ditujukan ke router.

| Parameter | Nilai |
|---|---|
| Chain | `input` |
| In. Interface | `bridgeLocal` |
| Protocol | `icmp` |
| Action | `accept` |

Selanjutnya, rule `drop` dibuat dengan kondisi ICMP yang sama dan ditempatkan di atas rule `accept`.

**Pengujian:**

```cmd
ping 192.168.88.1
```

**Hasil:**

- Ping ke `192.168.88.1` gagal ketika rule `drop` berada di atas rule `accept`.
- Packet counter pada rule `drop` bertambah menjadi 4 selama pengujian.
- WinBox tetap terhubung melalui LAN.

**Analisis:**

Paket ICMP dari laptop ditujukan ke router sehingga diproses oleh chain `input`. Paket yang cocok dengan rule `drop` dibuang sebelum mencapai rule `accept`. Koneksi WinBox tetap berfungsi karena menggunakan TCP, bukan ICMP.

### 6.3 Pengujian Chain Forward

Rule dibuat untuk membuang paket ICMP yang berasal dari LAN dan keluar menuju WAN.

| Parameter | Nilai |
|---|---|
| Chain | `forward` |
| In. Interface | `bridgeLocal` |
| Out. Interface | `wlan1` |
| Protocol | `icmp` |
| Action | `drop` |

**Pengujian:**

```cmd
ping 192.168.88.1
ping 8.8.8.8
```

**Hasil:**

- Ping ke `192.168.88.1` berhasil.
- Ping ke `8.8.8.8` gagal ketika rule `drop` aktif.
- Packet counter pada rule `forward` bertambah.
- WinBox tetap terhubung.

**Analisis:**

Ping ke alamat LAN router diproses oleh chain `input`, sedangkan ping dari laptop ke internet melewati router dan diproses oleh chain `forward`. Oleh karena itu, rule `drop` pada chain `forward` tidak langsung memblokir ping ke router.

### 6.4 Pengujian Chain Output

Rule observasi dibuat untuk mengizinkan ICMP yang dibuat oleh MikroTik sendiri.

| Parameter | Nilai |
|---|---|
| Chain | `output` |
| Protocol | `icmp` |
| Action | `accept` |

Pengujian dilakukan melalui Terminal MikroTik:

```routeros
/ping 8.8.8.8 count=5
```

**Hasil:**

- Ping dari MikroTik ke `8.8.8.8` berhasil.
- Packet counter pada rule `output` bertambah sebanyak 5 sesuai jumlah permintaan ping yang cocok dengan rule.

**Analisis:**

Paket ping yang dibuat langsung oleh router diproses melalui chain `output`. Pengujian ini menunjukkan bahwa trafik yang dibuat router berbeda dari trafik yang diteruskan dari laptop melalui chain `forward`.

Rule `accept` digunakan untuk observasi counter dan bukan bukti bahwa rule tersebut diperlukan agar ping berhasil.

## 7. Ringkasan Hasil Pengujian

| Pengujian | Hasil | Kesimpulan |
|---|---|---|
| Ping laptop ke `192.168.88.1` saat `input drop` aktif | Gagal | Paket ICMP dibuang oleh chain `input` |
| WinBox saat `input drop` aktif | Tetap terhubung | Trafik WinBox menggunakan TCP |
| Ping laptop ke `8.8.8.8` saat `forward drop` aktif | Gagal | Paket ICMP dibuang oleh chain `forward` |
| Ping laptop ke `192.168.88.1` saat `forward drop` aktif | Berhasil | Paket ditujukan ke router, bukan diteruskan |
| Ping MikroTik ke `8.8.8.8` | Berhasil | Router dapat mengirim ping ke internet |
| Counter rule `output` | Bertambah 5 | Lima permintaan ping cocok dengan rule observasi |

## 8. Kesimpulan

Praktik ini menunjukkan bahwa Firewall Filter MikroTik dapat mengontrol paket jaringan berdasarkan chain, kondisi rule, dan action yang digunakan.

Chain `input` menangani paket yang ditujukan ke router, chain `forward` menangani paket yang melewati router, dan chain `output` menangani paket yang dibuat oleh router itu sendiri.

Pengujian juga menunjukkan pentingnya urutan rule. Rule `drop` yang ditempatkan sebelum rule `accept` dapat membuang paket yang cocok dengan kondisinya. Oleh karena itu, penyusunan firewall perlu mempertimbangkan tujuan trafik, interface, protokol, urutan rule, dan kebutuhan akses manajemen.


