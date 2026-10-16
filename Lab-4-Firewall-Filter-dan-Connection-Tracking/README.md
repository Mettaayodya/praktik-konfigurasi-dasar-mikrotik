# Lab 4 — Firewall Filter dan Connection Tracking

## 1. Deskripsi

Lab ini mempelajari penggunaan Connection Tracking dan Firewall Filter pada MikroTik RouterOS untuk memahami status koneksi, perbedaan chain firewall, serta pengaruh urutan rule terhadap pemrosesan paket.

Konfigurasi dilakukan menggunakan WinBox pada MikroTik hAP lite RB941-2nD.

## 2. Tujuan

- Memahami fungsi Connection Tracking.
- Mengenal connection state `new`, `established`, `related`, dan `invalid`.
- Memahami penerapan Firewall Filter pada chain `input` dan `forward`.
- Membuat firewall filter berdasarkan connection state.
- Memahami pentingnya urutan firewall rules.
- Mengamati counter rule saat melakukan pengujian konektivitas.

### Konfigurasi jaringan

| Parameter | Konfigurasi |
|---|---|
| Interface LAN | `bridgeLocal` |
| Interface WAN | `wlan1` |
| IP router LAN | `192.168.88.1/24` |
| IP laptop | `192.168.88.254/24` |
| Gateway laptop | `192.168.88.1` |

## 3. Connection Tracking

Connection Tracking merupakan fitur MikroTik untuk melacak status koneksi jaringan. Informasi tersebut dapat digunakan oleh Firewall Filter untuk membedakan paket berdasarkan status koneksinya.

### Jenis connection state

| State | Penjelasan |
|---|---|
| `new` | Paket memulai koneksi baru atau belum dikenali sebagai bagian dari koneksi established. |
| `established` | Paket merupakan bagian dari koneksi yang telah dikenali. |
| `related` | Paket terkait dengan koneksi lain yang telah dikenali. |
| `invalid` | Paket tidak dapat dikaitkan dengan koneksi yang valid menurut Connection Tracking. |

Connection Tracking dapat diperiksa melalui menu **IP → Firewall → Connections**. Pada perangkat yang digunakan, status Connection Tracking adalah `enabled: auto`.

## 4. Firewall Filter

Firewall Filter digunakan untuk menentukan tindakan terhadap paket berdasarkan chain dan kondisi pencocokan tertentu.

### 4.1 Chain Input

Chain `input` memproses trafik yang ditujukan kepada router itu sendiri.

Contoh trafik input adalah laptop melakukan ping ke IP router `192.168.88.1` atau mengakses layanan administrasi router.

Rule yang digunakan dalam latihan:

| Rule | Chain | Kondisi | Action |
|---|---|---|---|
| Accept Established Related Input | `input` | `established,related` | `accept` |
| Drop Invalid Input | `input` | `invalid` | `drop` |
| Accept ICMP from LAN | `input` | ICMP dari `bridgeLocal` | `accept` |

Urutan rule aktif yang diamati:

1. Accept Established Related Input
2. Drop Invalid Input
3. Accept ICMP from LAN

### 4.2 Chain Forward

Chain `forward` memproses trafik yang melewati router, misalnya trafik dari laptop menuju internet.

Rule yang digunakan dalam latihan:

| Rule | Chain | Kondisi | Action |
|---|---|---|---|
| Accept Established Related Forward | `forward` | `established,related` | `accept` |
| Drop Invalid Forward | `forward` | `invalid` | `drop` |
| Accept New Forward from LAN | `forward` | `new`, dari `bridgeLocal` | `accept` |

Urutan rule aktif yang digunakan:

1. Accept Established Related Forward
2. Drop Invalid Forward
3. Accept New Forward from LAN

Rule `Accept New Forward from LAN` ditempatkan setelah `Drop Invalid Forward` agar paket yang cocok dengan kondisi `invalid` diproses terlebih dahulu.

## 5. Pengujian Konektivitas

Pengujian dilakukan dari laptop Windows yang terhubung ke MikroTik melalui interface LAN.

### 5.1 Ping ke router

Perintah:

```powershell
ping 192.168.88.1
```

Hasil pengujian:

- 4 paket dikirim.
- 4 paket diterima.
- Packet loss 0%.
- Rata-rata waktu respons sekitar 1 ms.

Hasil ini menunjukkan bahwa laptop dapat berkomunikasi dengan alamat LAN router.

### 5.2 Ping ke internet

Perintah:

```powershell
ping 8.8.8.8
```

Hasil pengujian:

- 4 paket dikirim.
- 4 paket diterima.
- Packet loss 0%.
- Rata-rata waktu respons sekitar 31 ms.

Hasil ini menunjukkan bahwa laptop dapat mencapai alamat IP tujuan di internet pada saat pengujian.

## 6. Pengamatan Firewall Counter

Counter pada Firewall Filter diamati sebelum dan sesudah pengujian untuk melihat rule yang cocok dengan trafik.

Salah satu hasil pengamatan:

| Rule | Counter yang diamati | Penjelasan |
|---|---:|---|
| Accept Established Related Forward | Sekitar 12.000 paket | Bertambah seiring adanya paket yang cocok dengan rule. |
| Drop Invalid Forward | 0 paket | Tidak ada paket yang cocok selama pengamatan tersebut. |
| Accept New Forward from LAN | 138 paket | Paket yang cocok dengan kondisi koneksi baru dari LAN. |

Nilai counter dapat berubah sesuai aktivitas jaringan. Counter menunjukkan jumlah paket yang cocok dengan rule, bukan jumlah pengguna atau jumlah koneksi unik.

Counter `Drop Invalid Forward` yang bernilai nol juga tidak otomatis membuktikan seluruh trafik valid; artinya hanya tidak ada paket yang cocok dengan rule tersebut selama interval pengamatan.

## 7. Urutan Firewall Rules

MikroTik mengevaluasi rule secara berurutan dalam setiap chain. Ketika sebuah paket cocok dengan rule yang memberikan tindakan `accept` atau `drop`, pemrosesan paket tersebut pada chain itu berhenti.

Karena itu, urutan rule harus disesuaikan dengan kondisi trafik dan tujuan firewall.

Dalam latihan ini, urutan rule pada chain `forward` adalah:

1. Accept Established Related Forward
2. Drop Invalid Forward
3. Accept New Forward from LAN

Penempatan `Drop Invalid Forward` sebelum `Accept New Forward from LAN` memastikan paket yang cocok dengan kondisi `invalid` tidak diterima oleh rule koneksi baru di bawahnya.

Rule pada chain `input` tidak otomatis mengatur trafik yang melewati router melalui chain `forward`. Pemilihan chain bergantung pada tujuan dan jalur paket.

## 8. Kesimpulan

Melalui Lab 4, saya mempelajari penggunaan Connection Tracking untuk mengenali status koneksi dan penerapannya pada Firewall Filter.

Saya juga memahami penggunaan chain `input` dan `forward` serta pentingnya urutan rule dalam menentukan tindakan terhadap paket.

Pengujian ping dan pengamatan counter membantu memverifikasi konektivitas serta melihat rule yang cocok dengan trafik selama pengujian.

Konfigurasi ini merupakan latihan dasar untuk memahami firewall MikroTik.
