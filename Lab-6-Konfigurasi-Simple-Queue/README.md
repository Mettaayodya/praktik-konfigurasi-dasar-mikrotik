# Lab 06 — Manajemen Bandwidth Menggunakan Simple Queue

## 1. Tujuan Praktikum

Praktikum ini bertujuan untuk memahami dan menerapkan manajemen bandwidth menggunakan fitur **Simple Queue** pada MikroTik RouterOS.

Tujuan khusus:
- Memahami fungsi Simple Queue untuk membatasi bandwidth klien.
- Mengatur batas maksimum upload dan download berdasarkan alamat IP.
- Memahami parameter `target` dan `max-limit`.
- Mengamati statistik trafik melalui Simple Queue.
- Membandingkan performa jaringan sebelum dan sesudah pembatasan bandwidth.

## 2. Topologi dan Lingkungan Praktikum

Perangkat yang digunakan:
- MikroTik hAP lite RB941-2nD.
- Laptop Windows sebagai klien pengujian.
- Smartphone sebagai sumber koneksi internet melalui hotspot.
- WinBox untuk konfigurasi MikroTik.
- Speedtest untuk mengukur performa jaringan.

### Konfigurasi IP

| Perangkat | Interface | IP Address |
|---|---|---|
| MikroTik | bridgeLocal | 192.168.88.1/24 |
| Laptop Windows | ether2 melalui bridgeLocal | 192.168.88.254/24 |
| MikroTik WAN | wlan1 | 10.188.212.157/24 |

Target pembatasan bandwidth adalah laptop Windows dengan alamat IP `192.168.88.254`.

## 3. Pengukuran Bandwidth Sebelum Pembatasan

Pengujian awal dilakukan menggunakan Speedtest dari laptop Windows.

| Parameter | Hasil |
|---|---:|
| Download | 44,62 Mbps |
| Upload | 18,74 Mbps |
| Ping | 32 ms |

Hasil tersebut digunakan sebagai kondisi pembanding sebelum penerapan pembatasan bandwidth.

## 4. Konfigurasi Simple Queue

Simple Queue dikonfigurasi untuk membatasi bandwidth laptop berdasarkan alamat IP.

### Parameter konfigurasi awal

| Parameter | Nilai |
|---|---|
| Name | LAB06-Limit-Laptop |
| Target | 192.168.88.254/32 |
| Max Limit Upload | 1 Mbps |
| Max Limit Download | 2 Mbps |

Konfigurasi dilakukan melalui WinBox pada menu **Queues → Simple Queues**.

Konfigurasi juga dapat diterapkan melalui Terminal:

```routeros
/queue simple add name="LAB06-Limit-Laptop" target=192.168.88.254/32 max-limit=1M/2M
```

**Keterangan parameter:**
- `name`: nama identifikasi Simple Queue.
- `target`: alamat IP perangkat yang menjadi sasaran pembatasan.
- `max-limit`: batas maksimum upload/download.

## 5. Pengujian Setelah Pembatasan

Setelah Simple Queue diaktifkan, pengujian Speedtest dilakukan kembali.

### Hasil pengujian pertama

| Parameter | Hasil |
|---|---:|
| Download | 1.51 Mbps |
| Upload | 0,92 Mbps |
| Ping | 24 ms |

## 6. Pengujian Pembanding Tanpa Simple Queue

Untuk memastikan pengaruh pembatasan bandwidth, Simple Queue dinonaktifkan sementara melalui WinBox.

Hasil Speedtest:

| Parameter | Hasil |
|---|---:|
| Download | 45.19 Mbps |
| Upload | 17.34 Mbps |
| Ping | 40 ms |

Kecepatan kembali meningkat setelah queue dinonaktifkan. Hal ini memperkuat bukti bahwa pembatasan Simple Queue memengaruhi throughput laptop.

Setelah pengujian, queue diaktifkan kembali.

## 7. Pengujian Perubahan Batas Bandwidth

Pengujian tambahan dilakukan dengan mengubah nilai Max Limit melalui GUI WinBox.

| Parameter | Konfigurasi 1 | Konfigurasi 2 |
|---|---:|---:|
| Upload | 1 Mbps | 3 Mbps |
| Download | 2 Mbps | 5 Mbps |

Hasil Speedtest pada konfigurasi kedua menunjukkan kecepatan sesuai dengan batas yang diterapkan.

Pengujian ini memperlihatkan bahwa perubahan parameter Max Limit dapat memengaruhi batas bandwidth yang tersedia bagi klien.

## 9. Analisis

Berdasarkan seluruh pengujian, Simple Queue berhasil diterapkan untuk membatasi bandwidth berdasarkan alamat IP klien.

Ketika queue aktif dengan batas upload 1 Mbps dan download 2 Mbps, kecepatan Speedtest mengalami penurunan dibandingkan kondisi tanpa pembatasan.

Ketika queue dinonaktifkan, kecepatan kembali meningkat. Pengujian tambahan dengan batas upload 3 Mbps dan download 5 Mbps juga menunjukkan hasil sesuai dengan batas yang diterapkan.

Perbedaan hasil Speedtest pada beberapa percobaan dapat dipengaruhi oleh kondisi jaringan hotspot, variasi pengukuran, dan beban jaringan. Oleh karena itu, nilai Max Limit dipahami sebagai batas maksimum, bukan kecepatan yang selalu dijamin akan tercapai.

## 10. Kesimpulan

Praktikum ini berhasil menerapkan manajemen bandwidth menggunakan Simple Queue pada MikroTik RouterOS.

Pembatasan bandwidth dapat dilakukan berdasarkan alamat IP klien dengan mengatur parameter Target dan Max Limit. Hasil pengujian menunjukkan bahwa Simple Queue dapat mengendalikan throughput upload dan download serta menyediakan statistik untuk memantau trafik klien.

Praktikum ini memberikan pemahaman dasar mengenai penerapan Quality of Service (QoS) melalui pembatasan bandwidth pada jaringan lokal.


