# ProjectJarkom_K4

| Nama | NRP |
|---|---|
|Abhista Athallah Dyfan | 5027251006|
|Ganestri Naurah Sawestri | 5027251014|
|Nur Rizki Syahbana | 5027251095|

# Soal 1: Perancangan Topologi

Topologi dibangun di Cisco Packet Tracer dengan skema **VLSM** (notasi prefix CIDR) dan blok dasar `10.32.0.0` (oktet kedua = 8 x nomor kelompok 4). Pusat jaringan berada di R-PUSAT (Balai Kota) dengan enam link backbone (L1 sampai L6) ke router wilayah dan satu link ke R-ISP.

![soal1-topologi](docs/img/soal1-topologi.png)

## Penjelasan Rancangan

### 1. Backbone antar-router

Susunan backbone mengikuti Gambar 1 pada soal: R-PUSAT sebagai pusat (hub) yang terhubung langsung ke setiap router wilayah.

| Link | Dari | Ke | Media |
|---|---|---|---|
| L1 | R-PUSAT | R-DC | Kabel serial (DCE di sisi R-PUSAT) |
| L2 | R-PUSAT | R-UTARA | Kabel serial (DCE di sisi R-PUSAT) |
| L3 | R-PUSAT | R-SELATAN | Kabel serial (DCE di sisi R-PUSAT) |
| L4 | R-PUSAT | R-TIMUR | Kabel serial (DCE di sisi R-PUSAT) |
| L5 | R-PUSAT | R-BARAT | Kabel serial (DCE di sisi R-PUSAT) |
| L6 | R-PUSAT | R-PELABUHAN | Kabel serial (DCE di sisi R-PUSAT) |
| ISP | R-PUSAT | R-ISP | Kabel crossover |
| - | R-ISP | SRV-INTERNET (`8.8.8.8`) | Kabel crossover |

### 2. Desain setiap lokasi

Setiap LAN memakai satu switch sendiri dan satu perangkat perwakilan (sesuai ketentuan soal). Perhitungan subnet tetap memakai jumlah host penuh.

| Lokasi (Router) | LAN | Switch | Perangkat perwakilan | Media |
|---|---|---|---|---|
| Balai Kota (R-PUSAT) | Pegawai Balai Kota | SW-PEGAWAI-BALAIKOTA | PC-PEGAWAI-BALAIKOTA | Kabel UTP |
| Balai Kota (R-PUSAT) | Server Balai Kota | SW-SERVER-BALAIKOTA | SRV-BALAIKOTA | Kabel UTP |
| Data Center (R-DC) | CCTV Kota | SW-CCTV-KOTA | PC-CCTV-KOTA | Kabel UTP |
| Data Center (R-DC) | Server dan Storage DC | SW-SERVER-DC | SRV-DC | Kabel UTP |
| Utara (R-UTARA) | WiFi Publik Utara | SW-WIFI-UTARA, AP-WIFI-UTARA | LAPTOP-WIFI-UTARA | Kabel UTP ke AP, lalu wireless |
| Utara (R-UTARA) | Sensor Lalu Lintas Utara | SW-SENSOR-LALIN-UTARA | PC-SENSOR-LALIN-UTARA | Kabel UTP |
| Selatan (R-SELATAN) | WiFi Publik Selatan | SW-WIFI-SELATAN, AP-WIFI-SELATAN | SP-WIFI-SELATAN (smartphone) | Kabel UTP ke AP, lalu wireless |
| Selatan (R-SELATAN) | PJU Pintar Selatan | SW-PJU-SELATAN | PC-PJU-SELATAN | Kabel UTP |
| Timur (R-TIMUR) | WiFi Publik Timur | SW-WIFI-TIMUR, AP-WIFI-TIMUR | LAPTOP-WIFI-TIMUR | Kabel UTP ke AP, lalu wireless |
| Timur (R-TIMUR) | Sensor Banjir Timur | SW-SENSOR-BANJIR-TIMUR | PC-SENSOR-BANJIR-TIMUR | Kabel UTP |
| Barat (R-BARAT) | WiFi Publik Barat | SW-WIFI-BARAT, AP-WIFI-BARAT | SP-WIFI-BARAT (smartphone) | Kabel UTP ke AP, lalu wireless |
| Barat (R-BARAT) | Puskesmas dan Sekolah Barat | SW-PUSKESMAS-SEKOLAH-BARAT | PC-PUSKESMAS-SEKOLAH-BARAT | Kabel UTP |
| Pelabuhan (R-PELABUHAN) | Kantor Pelabuhan | SW-KANTOR-PELABUHAN | PC-KANTOR-PELABUHAN | Kabel UTP |
| Pelabuhan (R-PELABUHAN) | Sensor dan SCADA Pelabuhan | SW-SCADA-PELABUHAN | PC-SCADA-PELABUHAN | Kabel UTP |

### 3. Penambahan port router

| Router | Kebutuhan port | Penambahan |
|---|---|---|
| R-PUSAT | 6 serial (L1 sampai L6), 3 Gigabit (ISP, LAN Pegawai, LAN Server Balai Kota) | 3 modul HWIC-2T (6 port serial) |
| R-DC | 1 serial (link ke R-PUSAT), 2 Gigabit (CCTV Kota dan Server DC) | 1 modul HWIC-2T |
| R-UTARA | 1 serial (link ke R-PUSAT), 2 Gigabit (WiFi Utara dan Sensor Lalu Lintas) | 1 modul HWIC-2T |
| R-SELATAN | 1 serial (link ke R-PUSAT), 2 Gigabit (WiFi Selatan dan PJU) | 1 modul HWIC-2T |
| R-TIMUR | 1 serial (link ke R-PUSAT), 2 Gigabit (WiFi Timur dan Sensor Banjir) | 1 modul HWIC-2T |
| R-BARAT | 1 serial (link ke R-PUSAT), 2 Gigabit (WiFi Barat dan Puskesmas Sekolah) | 1 modul HWIC-2T |
| R-PELABUHAN | 1 serial (link ke R-PUSAT), 2 Gigabit (Kantor dan SCADA Pelabuhan) | 1 modul HWIC-2T |
| R-ISP | 1 Gigabit (ke R-PUSAT), 1 Gigabit (ke SRV-INTERNET) | Tidak perlu, port bawaan cukup |

### 4. Pertimbangan dan alasan

| Keputusan | Alasan |
|---|---|
| Topologi star dengan R-PUSAT sebagai pusat | Setiap wilayah satu hop dari pusat, rute mudah dibuat dan satu gangguan link tidak memutus wilayah lain |
| Satu switch untuk setiap LAN | Setiap subnet menjadi satu domain broadcast sendiri dan sesuai dengan pembagian subnet pada tabel VLSM |
| Link backbone memakai kabel serial | Link antar-router bersifat point-to-point sehingga cukup subnet `/30` |
| Access point untuk LAN WiFi Publik | Pengguna WiFi publik memakai perangkat mobile (laptop dan smartphone) |
| Kabel UTP untuk sensor, CCTV, PJU, SCADA, dan server | Perangkat tetap yang membutuhkan koneksi stabil |
| Server ditempatkan pada LAN tersendiri | Layanan pusat (DNS, FTP, Web, Email) terpisah dari LAN pengguna sehingga mudah diatur |
| Satu perangkat perwakilan per LAN | Sesuai ketentuan soal. Perhitungan subnet tetap memakai jumlah host penuh |

### 5. Penamaan perangkat dan label LAN

Seluruh perangkat diganti namanya sesuai tempatnya (contoh: `R-PUSAT`, `SW-SERVER-BALAIKOTA`, `PC-CCTV-KOTA`, `AP-WIFI-UTARA`). Setiap LAN diberi label berisi nama subnet, Network ID/prefix, dan jumlah host, misalnya `Pegawai Balai Kota - 10.32.224.0/21 - 1.800 host`.

---

# Soal 2: Tabel Subnetting

## Penjelasan Perhitungan

- Blok dasar `10.32.0.0` (oktet kedua = 8 x 4). Seluruh alamat LAN berada dalam agregat CIDR `10.32.0.0/16`.
- Metode VLSM: subnet diurutkan dari kebutuhan host terbesar ke terkecil, lalu diberi prefix terkecil yang masih mencukupi (`2^(32-prefix) - 2 >= host`). Blok berikutnya dimulai tepat setelah broadcast blok sebelumnya sehingga tidak ada tumpang tindih.
- Gateway setiap LAN adalah alamat host pertama pada subnet. Alamat `.1` sampai `.10` dikecualikan dari DHCP (lihat Soal 4).

## Tabel Subnetting LAN (VLSM dan CIDR)

| No | Nama Subnet | Router | Kebutuhan Host | Network ID/Prefix | Netmask | Rentang Host | Broadcast | Gateway | Host Tersedia |
|---|---|---|---|---|---|---|---|---|---|
| 1 | WiFi Publik Utara | R-UTARA | 11.000 | `10.32.0.0/18` | `255.255.192.0` | `10.32.0.1 - 10.32.63.254` | `10.32.63.255` | `10.32.0.1` | 16.382 |
| 2 | WiFi Publik Selatan | R-SELATAN | 6.500 | `10.32.64.0/19` | `255.255.224.0` | `10.32.64.1 - 10.32.95.254` | `10.32.95.255` | `10.32.64.1` | 8.190 |
| 3 | CCTV Kota | R-DC | 6.000 | `10.32.96.0/19` | `255.255.224.0` | `10.32.96.1 - 10.32.127.254` | `10.32.127.255` | `10.32.96.1` | 8.190 |
| 4 | WiFi Publik Timur | R-TIMUR | 5.500 | `10.32.128.0/19` | `255.255.224.0` | `10.32.128.1 - 10.32.159.254` | `10.32.159.255` | `10.32.128.1` | 8.190 |
| 5 | Sensor dan SCADA Pelabuhan | R-PELABUHAN | 5.000 | `10.32.160.0/19` | `255.255.224.0` | `10.32.160.1 - 10.32.191.254` | `10.32.191.255` | `10.32.160.1` | 8.190 |
| 6 | WiFi Publik Barat | R-BARAT | 4.500 | `10.32.192.0/19` | `255.255.224.0` | `10.32.192.1 - 10.32.223.254` | `10.32.223.255` | `10.32.192.1` | 8.190 |
| 7 | Pegawai Balai Kota | R-PUSAT | 1.800 | `10.32.224.0/21` | `255.255.248.0` | `10.32.224.1 - 10.32.231.254` | `10.32.231.255` | `10.32.224.1` | 2.046 |
| 8 | Sensor Banjir Timur | R-TIMUR | 700 | `10.32.232.0/22` | `255.255.252.0` | `10.32.232.1 - 10.32.235.254` | `10.32.235.255` | `10.32.232.1` | 1.022 |
| 9 | Sensor Lalu Lintas Utara | R-UTARA | 450 | `10.32.236.0/23` | `255.255.254.0` | `10.32.236.1 - 10.32.237.254` | `10.32.237.255` | `10.32.236.1` | 510 |
| 10 | Puskesmas dan Sekolah Barat | R-BARAT | 220 | `10.32.238.0/24` | `255.255.255.0` | `10.32.238.1 - 10.32.238.254` | `10.32.238.255` | `10.32.238.1` | 254 |
| 11 | Server dan Storage DC | R-DC | 180 | `10.32.239.0/24` | `255.255.255.0` | `10.32.239.1 - 10.32.239.254` | `10.32.239.255` | `10.32.239.1` | 254 |
| 12 | Kantor Pelabuhan | R-PELABUHAN | 120 | `10.32.240.0/25` | `255.255.255.128` | `10.32.240.1 - 10.32.240.126` | `10.32.240.127` | `10.32.240.1` | 126 |
| 13 | PJU Pintar Selatan | R-SELATAN | 90 | `10.32.240.128/25` | `255.255.255.128` | `10.32.240.129 - 10.32.240.254` | `10.32.240.255` | `10.32.240.129` | 126 |
| 14 | Server Balai Kota | R-PUSAT | 20 | `10.32.241.0/27` | `255.255.255.224` | `10.32.241.1 - 10.32.241.30` | `10.32.241.31` | `10.32.241.1` | 30 |

## Tabel Subnetting Link Antar-Router dan Layanan

| No | Keperluan | Network ID/Prefix | Netmask | Rentang Host | Broadcast | Keterangan |
|---|---|---|---|---|---|---|
| 15 | Link L1 R-PUSAT ke R-DC | `10.32.241.32/30` | `255.255.255.252` | `10.32.241.33 - 10.32.241.34` | `10.32.241.35` | Point-to-point |
| 16 | Link L2 R-PUSAT ke R-UTARA | `10.32.241.36/30` | `255.255.255.252` | `10.32.241.37 - 10.32.241.38` | `10.32.241.39` | Point-to-point |
| 17 | Link L3 R-PUSAT ke R-SELATAN | `10.32.241.40/30` | `255.255.255.252` | `10.32.241.41 - 10.32.241.42` | `10.32.241.43` | Point-to-point |
| 18 | Link L4 R-PUSAT ke R-TIMUR | `10.32.241.44/30` | `255.255.255.252` | `10.32.241.45 - 10.32.241.46` | `10.32.241.47` | Point-to-point |
| 19 | Link L5 R-PUSAT ke R-BARAT | `10.32.241.48/30` | `255.255.255.252` | `10.32.241.49 - 10.32.241.50` | `10.32.241.51` | Point-to-point |
| 20 | Link L6 R-PUSAT ke R-PELABUHAN | `10.32.241.52/30` | `255.255.255.252` | `10.32.241.53 - 10.32.241.54` | `10.32.241.55` | Point-to-point |
| 21 | Link R-PUSAT ke R-ISP | `203.0.113.16/30` | `255.255.255.252` | `203.0.113.17 - 203.0.113.18` | `203.0.113.19` | `203.0.113.(4 x K)/30` dengan K = 4 |
| 22 | Server internet | `8.8.8.0/24` | `255.255.255.0` | `8.8.8.1 - 8.8.8.254` | `8.8.8.255` | Server `8.8.8.8`, gateway `8.8.8.1` di R-ISP |
| 23 | Tunnel GRE | `192.168.4.0/30` | `255.255.255.252` | `192.168.4.1 - 192.168.4.2` | `192.168.4.3` | `192.168.K.0/30`. Tunnel R-PUSAT `192.168.4.1`, tunnel R-PELABUHAN `192.168.4.2` |

## Tabel Revisi

Perbaikan kesalahan hitung pada jawaban UTS dicatat pada tabel berikut.

| Subnet | Jawaban di UTS | Perbaikan | Alasan Kesalahan |
|---|---|---|---|
| - | - | - | - |

---

# Soal 4: DHCP

## Penjelasan Konfigurasi

### 1. IP statis interface router dan server

Interface router diberi IP statis (`ip address ... ` lalu `no shutdown`) sesuai tabel alamat VLSM. Server memakai IP statis dari menu Desktop, IP Configuration, Static.

| Server | IPv4 | Mask | Gateway | DNS |
|---|---|---|---|---|
| SRV-DC | `10.32.239.2` | `255.255.255.0` | `10.32.239.1` | `10.32.239.2` |
| SRV-BALAIKOTA | `10.32.241.2` | `255.255.255.224` | `10.32.241.1` | `10.32.239.2` |

### 2. DHCP di router (LAN pengguna)

Setiap router punya satu pool DHCP untuk tiap LAN pengguna di bawahnya. Contoh konfigurasi R-UTARA:

```
ip dhcp excluded-address 10.32.0.1 10.32.0.10
ip dhcp excluded-address 10.32.236.1 10.32.236.10
ip dhcp pool WIFI-UTARA
 network 10.32.0.0 255.255.192.0
 default-router 10.32.0.1
 dns-server 10.32.239.2
 domain-name k4.smartcity.lab
ip dhcp pool SENSOR-LALIN-UTARA
 network 10.32.236.0 255.255.254.0
 default-router 10.32.236.1
 dns-server 10.32.239.2
 domain-name k4.smartcity.lab
```

Penjelasan perintah:

| Perintah | Fungsi |
|---|---|
| `ip dhcp excluded-address` | Mengecualikan `.1` sampai `.10` dari pool agar alamat statis (gateway, server, infrastruktur) tidak bentrok dengan alamat dinamis. Pool dimulai dari `.11` |
| `ip dhcp pool <nama>` | Membuat pool untuk satu LAN |
| `network` | Menentukan network dan mask yang dibagikan |
| `default-router` | Gateway yang diberikan ke perangkat |
| `dns-server` | DNS server yang diberikan ke perangkat (`10.32.239.2`, soal 8) |
| `domain-name` | Domain yang diberikan ke perangkat (`k4.smartcity.lab`) |

Ringkasan seluruh pool:

| Router | Pool | Network dan mask | Gateway | Excluded |
|---|---|---|---|---|
| R-PUSAT | PEGAWAI-BALAIKOTA | `10.32.224.0 255.255.248.0` | `10.32.224.1` | `.1 - .10` |
| R-UTARA | WIFI-UTARA | `10.32.0.0 255.255.192.0` | `10.32.0.1` | `.1 - .10` |
| R-UTARA | SENSOR-LALIN-UTARA | `10.32.236.0 255.255.254.0` | `10.32.236.1` | `.1 - .10` |
| R-PELABUHAN | SCADA-PELABUHAN | `10.32.160.0 255.255.224.0` | `10.32.160.1` | `.1 - .10` |
| R-PELABUHAN | KANTOR-PELABUHAN | `10.32.240.0 255.255.255.128` | `10.32.240.1` | `.1 - .10` |
| R-SELATAN | WIFI-SELATAN | `10.32.64.0 255.255.224.0` | `10.32.64.1` | `.1 - .10` |
| R-SELATAN | PJU-SELATAN | `10.32.240.128 255.255.255.128` | `10.32.240.129` | `.129 - .138` |
| R-TIMUR | WIFI-TIMUR | `10.32.128.0 255.255.224.0` | `10.32.128.1` | `.1 - .10` |
| R-TIMUR | SENSOR-BANJIR-TIMUR | `10.32.232.0 255.255.252.0` | `10.32.232.1` | `.1 - .10` |
| R-BARAT | WIFI-BARAT | `10.32.192.0 255.255.224.0` | `10.32.192.1` | `.1 - .10` |
| R-BARAT | PUSKESMAS-SEKOLAH-BARAT | `10.32.238.0 255.255.255.0` | `10.32.238.1` | `.1 - .10` |

### 3. DHCP khusus CCTV Kota (server DHCP di LAN Server dan Storage DC)

Subnet CCTV Kota tidak dilayani router. R-DC tidak memiliki pool DHCP. Interface R-DC ke LAN CCTV hanya meneruskan permintaan DHCP ke SRV-DC:

```
interface GigabitEthernet0/0
 ip address 10.32.96.1 255.255.224.0
 ip helper-address 10.32.239.2
```

`ip helper-address` mengubah broadcast DHCP dari LAN CCTV menjadi unicast ke `10.32.239.2`, karena router tidak meneruskan broadcast antar jaringan.

Di SRV-DC (Services, DHCP, Service On) dibuat pool:

| Field | Nilai |
|---|---|
| Pool Name | CCTV-KOTA |
| Default Gateway | 10.32.96.1 |
| DNS Server | 10.32.239.2 |
| Start IP Address | 10.32.96.11 |
| Subnet Mask | 255.255.224.0 |
| Maximum Number of Users | 6000 |

## Bukti Pengujian Mandiri

| No | Pengujian | Perintah / cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Interface router memakai IP statis | `show ip interface brief` | Kolom Method `manual`, status up/up | <img width="1404" height="450" alt="soal4-show-ip-interface-brief" src="https://github.com/user-attachments/assets/e0bce59c-38b5-469c-9a62-582e7d909578" /> |
| 2 | IP statis server | `ipconfig` di SRV-DC dan SRV-BALAIKOTA | `10.32.239.2` dan `10.32.241.2` | <img width="1190" height="708" alt="soal4-ipconfig-srv-balaikota" src="https://github.com/user-attachments/assets/4de2c7d9-4c8a-4546-a5dd-49c6b9bfcc4f" /> <img width="1422" height="1046" alt="soal4-ipconfig-srv-dc" src="https://github.com/user-attachments/assets/3c005ae8-7dfb-4ecf-996f-506fd2656a1c" /> |
| 3 | Pool dan alamat yang dikecualikan | `show ip dhcp pool` dan `show running-config \| include excluded` | Pool sesuai tabel, excluded `.1 - .10` | <img width="1444" height="1058" alt="soal4-show-ip-dhcp-pool" src="https://github.com/user-attachments/assets/3e1b0e62-81f0-4517-bfa1-db08201577c3" /> |
| 4 | Perangkat pengguna mendapat IP, gateway, dan DNS | IP Configuration, DHCP, lalu `ipconfig /all` (PC-KANTOR-PELABUHAN) | IP `10.32.240.11`, gateway `10.32.240.1`, DNS `10.32.239.2` | <img width="1432" height="1256" alt="soal4-ipconfig-pc-kantor-pelabuhan" src="https://github.com/user-attachments/assets/e256f16d-47dc-4e84-a4d6-c76b1eadc54a" /> |
| 5 | Perangkat pengguna di wilayah lain | `ipconfig /all` | IP dari subnet yang sesuai, mulai `.11` |<img width="1428" height="1698" alt="soal4-ipconfig-wilayah-lain" src="https://github.com/user-attachments/assets/02165885-6619-4f76-91b1-9859f778bdcb" /> |
| 6 | Tidak bentrok dengan IP statis | `show ip dhcp binding` | Alamat yang tersewa mulai dari `.11` |<img width="1458" height="520" alt="soal4-show-ip-dhcp-binding" src="https://github.com/user-attachments/assets/06f1b8bd-16a6-401f-88b1-7593ffd30166" /> |
| 7 | R-DC tidak punya pool DHCP | `show running-config \| include dhcp` di R-DC | Tidak ada keluaran | <img width="814" height="136" alt="image" src="https://github.com/user-attachments/assets/0aab07be-84df-49e2-a1b7-33f81f6e95c1" /> |
| 8 | Relay di R-DC | `show ip interface GigabitEthernet0/0` | `Helper address is 10.32.239.2` | <img width="1206" height="712" alt="soal4-r-dc-helper-address" src="https://github.com/user-attachments/assets/252ad078-616b-4412-94ca-c7707c680b81" /> |
| 9 | Pool CCTV di server | SRV-DC, Services, DHCP | Pool `CCTV-KOTA` terdaftar | <img width="2880" height="1232" alt="soal4-srv-dc-dhcp-cctv" src="https://github.com/user-attachments/assets/6aec5677-1d65-4b88-ac64-857702fe9dca" /> |
| 10 | CCTV mendapat IP dari server | `ipconfig /all` di PC-CCTV-KOTA | IP `10.32.96.11`, DHCP Servers `10.32.239.2` | <img width="1446" height="1246" alt="soal4-ipconfig-pc-cctv" src="https://github.com/user-attachments/assets/fa4c9f3a-ed5b-4137-b827-38fc989c472a" /> |

---

# Soal 8: Layanan Aplikasi Smart City

Domain internal: `k4.smartcity.lab`

## Penjelasan Konfigurasi

| Server | Layanan |
|---|---|
| SRV-DC (`10.32.239.2`, LAN Server dan Storage DC) | DNS, FTP, dan DHCP (pool CCTV) |
| SRV-BALAIKOTA (`10.32.241.2`, LAN Server Balai Kota) | Web (HTTP), Email (SMTP dan POP3) |

### a. DNS (SRV-DC, Services, DNS, Service On)

| Name | Type | Address |
|---|---|---|
| `www.k4.smartcity.lab` | A Record | `10.32.241.2` |
| `ftp.k4.smartcity.lab` | A Record | `10.32.239.2` |
| `mail.k4.smartcity.lab` | A Record | `10.32.241.2` |

`www` dan `mail` menunjuk ke SRV-BALAIKOTA, `ftp` menunjuk ke SRV-DC. Alamat DNS `10.32.239.2` dibagikan ke semua perangkat lewat DHCP (soal 4), sehingga nama bisa di-resolve dari seluruh wilayah.

### b. Web (SRV-BALAIKOTA, Services, HTTP, On)

Isi `index.html`:

```html
<html><body>
<h1>Portal Smart City Surabaya</h1>
<p>Kelompok 4</p>
</body></html>
```

### c. FTP (SRV-DC, Services, FTP, On)

| Item | Nilai |
|---|---|
| Username | `cctv` |
| Password | `smartcity4` |
| Permission | Write, Read, Delete, Rename, List |
| Akun bawaan `cisco` | Dihapus |

### d. Email (SRV-BALAIKOTA, Services, EMAIL)

| Item | Nilai |
|---|---|
| SMTP Service dan POP3 Service | On |
| Domain Name | `k4.smartcity.lab` |
| Akun Balai Kota | `pegawai.bk` / `bk123` |
| Akun Pelabuhan | `pegawai.pel` / `pel123` |

Pengaturan klien email (Desktop, Email, Configure Mail):

| Field | PC-PEGAWAI-BALAIKOTA | PC-KANTOR-PELABUHAN |
|---|---|---|
| Email Address | `pegawai.bk@k4.smartcity.lab` | `pegawai.pel@k4.smartcity.lab` |
| Incoming dan Outgoing Mail Server | `mail.k4.smartcity.lab` | `mail.k4.smartcity.lab` |
| User Name | `pegawai.bk` | `pegawai.pel` |
| Password | `bk123` | `pel123` |

## Bukti Pengujian Mandiri

Pengujian dilakukan dari lebih dari satu wilayah.

### a. DNS

| No | Pengujian | Perintah | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Record DNS di server | SRV-DC, Services, DNS | Tiga record `www`, `ftp`, `mail` | <img width="1094" height="658" alt="image" src="https://github.com/user-attachments/assets/5abd06a3-4052-4bc4-8f0f-ddc3b7d69d3d" /> |
| 2 | Dari Pelabuhan | `nslookup www.k4.smartcity.lab`, `nslookup ftp.k4.smartcity.lab`, `nslookup mail.k4.smartcity.lab` | `10.32.241.2`, `10.32.239.2`, `10.32.241.2` |<img width="1402" height="956" alt="image" src="https://github.com/user-attachments/assets/627ceb50-4533-41b3-bb8a-9f32ac725aea" />|
| 3 | Dari wilayah lain | perintah yang sama | hasil yang sama | <img width="1446" height="964" alt="image" src="https://github.com/user-attachments/assets/bc8a3191-8b48-4d19-87db-2f585ea30fbe" /> |
| 4 | Dari CCTV | perintah yang sama | hasil yang sama | <img width="1426" height="1178" alt="soal8-nslookup-cctv" src="https://github.com/user-attachments/assets/6856d913-f166-4164-8146-0ba140584ff9" /> |

### b. Web

| No | Pengujian | Cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Dari Pelabuhan | Web Browser, `http://www.k4.smartcity.lab` | Halaman "Portal Smart City Surabaya" dan "Kelompok 4" | ![](docs/img/soal8-web-pelabuhan.png) |
| 2 | Dari wilayah lain | Web Browser, `http://www.k4.smartcity.lab` | Halaman yang sama | <img width="1472" height="1720" alt="soal8-web-wilayah-lain" src="https://github.com/user-attachments/assets/7e0f5953-3261-4938-982e-7f2e683edb16" /> |

### c. FTP

| No | Pengujian | Perintah | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Akun dan layanan FTP | SRV-DC, Services, FTP | User `cctv` terdaftar, `cisco` tidak ada | <img width="1422" height="1020" alt="image" src="https://github.com/user-attachments/assets/e97067b3-f06a-4cfd-bb1d-997911b600fb" /> |
| 2 | Login dan unggah dari LAN CCTV | `ftp ftp.k4.smartcity.lab`, login `cctv`, `put rekaman02.txt`, `dir` | `Transfer complete`, file muncul di `dir` | <img width="1438" height="1704" alt="soal8-ftp-upload-cctv" src="https://github.com/user-attachments/assets/a4f154af-8fdb-4d56-b06f-fca99bc4f725" /> |
| 3 | Unduh dari wilayah lain | `ftp ftp.k4.smartcity.lab`, login `cctv`, `get rekaman02.txt` | File berhasil diunduh |<img width="1438" height="1536" alt="image" src="https://github.com/user-attachments/assets/f338c924-4bd7-4da3-ac58-c81d12691d3c" /> |

### d. Email

| No | Pengujian | Cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Layanan dan akun email | SRV-BALAIKOTA, Services, EMAIL | Domain `k4.smartcity.lab`, dua akun terdaftar | <img width="1450" height="898" alt="image" src="https://github.com/user-attachments/assets/51c62dc6-0ec1-4da3-9ea0-6171825c9698" /> |
| 2 | Kirim dari Balai Kota | Compose ke `pegawai.pel@k4.smartcity.lab` | Email terkirim | <img width="1410" height="1120" alt="image" src="https://github.com/user-attachments/assets/9ef8d8f8-ca3c-43d6-9984-c136d0c8d7fe" /> |
| 3 | Diterima di Pelabuhan | Receive di PC-KANTOR-PELABUHAN | Email muncul | <img width="1454" height="1350" alt="image" src="https://github.com/user-attachments/assets/ba86267c-b24d-4ccc-8fe1-980c46eb7120" /> |
| 4 | Balasan dari Pelabuhan | Compose ke `pegawai.bk@k4.smartcity.lab`, lalu Receive di Balai Kota | Balasan muncul | <img width="1454" height="1350" alt="image" src="https://github.com/user-attachments/assets/d33e8063-2aa8-4fd6-a5c1-396278b644c2" /> |
