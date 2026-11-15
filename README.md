# ProjectJarkom_K4

| Nama | NRP |
|---|---|
|Abhista Athallah Dyfan | 5027251006|
|Ganestri Naurah Sawestri | 5027251014|
|Nur Rizki Syahbana | 5027251095|

# Soal 1: Perancangan Topologi

Topologi dibangun di Cisco Packet Tracer dengan skema **VLSM** (notasi prefix CIDR) dan blok dasar `10.32.0.0` (oktet kedua = 8 x nomor kelompok 4). R-PUSAT (Balai Kota) menjadi pusat jaringan: lima router wilayah (R-DC, R-UTARA, R-SELATAN, R-TIMUR, R-BARAT) terhubung langsung ke R-PUSAT, sedangkan R-PELABUHAN terhubung melalui R-UTARA (L6). Susunan backbone mengikuti Gambar 1 pada soal.

<img width="1074" height="514" alt="Screenshot 2026-11-15 at 21 01 35" src="https://github.com/user-attachments/assets/da14a455-5558-4bc0-9437-5f609a7734d2" />

## Penjelasan Rancangan

### 1. Backbone antar-router

| Link | Dari | Ke | Subnet |
|---|---|---|---|
| L1 | R-PUSAT | R-DC | `10.32.241.32/30` |
| L2 | R-PUSAT | R-UTARA | `10.32.241.36/30` |
| L3 | R-PUSAT | R-SELATAN | `10.32.241.40/30` |
| L4 | R-PUSAT | R-TIMUR | `10.32.241.44/30` |
| L5 | R-PUSAT | R-BARAT | `10.32.241.48/30` |
| L6 | R-UTARA | R-PELABUHAN | `10.32.241.52/30` |
| ISP | R-PUSAT | R-ISP | `203.0.113.16/30` |
| - | R-ISP | SRV-INTERNET (`8.8.8.8`) | `8.8.8.0/24` |
| GRE | R-PUSAT (Tunnel0) | R-PELABUHAN (Tunnel0) | `192.168.4.0/30`, jalur melewati L2 dan L6 (via R-UTARA) |

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

Router 2911 hanya membawa 3 port Gigabit bawaan, sehingga router dengan lebih dari 3 interface diberi modul port tambahan. Nama interface mengikuti tabel alamat interface pada Soal 2.

| Router | Interface | Kebutuhan | Penambahan |
|---|---|---|---|
| R-PUSAT | `Gi0/0` - `Gi0/7`, `Tunnel0` | 8 port Gigabit: 2 LAN (Pegawai dan Server Balai Kota), 5 link (L1 sampai L5), 1 link ISP | 5 port |
| R-UTARA | `Gi0/0` - `Gi0/3` | 4 port Gigabit: L2, WiFi Utara, Sensor Lalu Lintas, L6 | 1 port |
| R-DC | `Gi0/0` - `Gi0/2` | 3 port Gigabit: L1, CCTV Kota, Server DC | Tidak perlu |
| R-BARAT | `Gi0/0` - `Gi0/2` | 3 port Gigabit: L5, WiFi Barat, Puskesmas dan Sekolah | Tidak perlu |
| R-TIMUR | `Gi0/0` - `Gi0/2` | 3 port Gigabit: L4, WiFi Timur, Sensor Banjir | Tidak perlu |
| R-SELATAN | `Gi0/0` - `Gi0/2` | 3 port Gigabit: L3, WiFi Selatan, PJU | Tidak perlu |
| R-PELABUHAN | `Gi0/0` - `Gi0/2` | 3 port Gigabit: L6, SCADA, Kantor (ditambah `Tunnel0`) | Tidak perlu |
| R-ISP | `Gi0/0` - `Gi0/1` | 2 port Gigabit: ke R-PUSAT dan SRV-INTERNET | Tidak perlu |

### 4. Pertimbangan dan alasan

| Keputusan | Alasan |
|---|---|
| R-PUSAT sebagai pusat dan R-PELABUHAN lewat R-UTARA | Mengikuti Gambar 1. Lima wilayah satu hop dari pusat, rute mudah dibuat |
| Satu switch untuk setiap LAN | Setiap subnet menjadi satu domain broadcast sendiri dan sesuai dengan pembagian subnet pada tabel VLSM |
| Setiap link backbone memakai subnet `/30` | Link antar-router bersifat point-to-point sehingga hanya butuh 2 alamat host |
| Access point untuk LAN WiFi Publik | Pengguna WiFi publik memakai perangkat mobile (laptop dan smartphone) |
| Kabel UTP untuk sensor, CCTV, PJU, SCADA, dan server | Perangkat tetap yang membutuhkan koneksi stabil |
| Server ditempatkan pada LAN tersendiri | Layanan pusat (DNS, FTP, Web, Email) terpisah dari LAN pengguna sehingga mudah diatur |
| Satu perangkat perwakilan per LAN | Sesuai ketentuan soal. Perhitungan subnet tetap memakai jumlah host penuh |

### 5. Penamaan perangkat dan label LAN

Seluruh perangkat diganti namanya sesuai tempatnya (contoh: `R-PUSAT`, `SW-SERVER-BALAIKOTA`, `PC-CCTV-KOTA`, `AP-WIFI-UTARA`). Setiap LAN diberi label berisi nama subnet, Network ID/prefix, dan jumlah host, misalnya `Pegawai Balai Kota - 10.32.224.0/21 - 1.800 host`.

---

# Soal 2: Tabel Subnetting

## Penjelasan Perhitungan

- Kelompok 4 (K = 4), alamat dasar `10.32.0.0` (oktet kedua = 8 x K). Blok `10.0.0.0/8` adalah blok alamat private yang dipakai di lingkup internal.
- Jumlah host berasal dari soal UTS. Kebutuhan alamat = host + 2 (network ID dan broadcast). Prefix = 32 - pembulatan ke atas log2(kebutuhan).
- **VLSM**: subnet diurutkan dari host terbesar dan dialokasikan berurutan mulai dari alamat dasar (offset = offset sebelumnya + ukuran blok sebelumnya). Skema ini yang dipakai di Packet Tracer.
- **CIDR**: LAN dikelompokkan per router menjadi satu blok (supernet), lalu blok router digabung menjadi satu supernet di akar.
- Gateway setiap LAN adalah alamat host pertama subnet.
- Subnet link L1 sampai L6 tidak ada di lembar UTS, ditambahkan sebagai `/30` di akhir blok (dicatat pada tabel revisi).

### Tabel kebutuhan alamat

| Router | LAN | Jumlah Host | Kebutuhan Alamat (host + 2) | Prefix | Ukuran Blok | Subnet Mask |
|---|---|---|---|---|---|---|
| R-PUSAT | Pegawai Balai Kota | 1.800 | 1.802 | `/21` | 2.048 | `255.255.248.0` |
| R-PUSAT | Server Balai Kota | 20 | 22 | `/27` | 32 | `255.255.255.224` |
| R-DC | CCTV Kota | 6.000 | 6.002 | `/19` | 8.192 | `255.255.224.0` |
| R-DC | Server dan Storage DC | 180 | 182 | `/24` | 256 | `255.255.255.0` |
| R-UTARA | WiFi Publik Utara | 11.000 | 11.002 | `/18` | 16.384 | `255.255.192.0` |
| R-UTARA | Sensor Lalu Lintas Utara | 450 | 452 | `/23` | 512 | `255.255.254.0` |
| R-PELABUHAN | Sensor dan SCADA Pelabuhan | 5.000 | 5.002 | `/19` | 8.192 | `255.255.224.0` |
| R-PELABUHAN | Kantor Pelabuhan | 120 | 122 | `/25` | 128 | `255.255.255.128` |
| R-SELATAN | WiFi Publik Selatan | 6.500 | 6.502 | `/19` | 8.192 | `255.255.224.0` |
| R-SELATAN | PJU Pintar Selatan | 90 | 92 | `/25` | 128 | `255.255.255.128` |
| R-TIMUR | WiFi Publik Timur | 5.500 | 5.502 | `/19` | 8.192 | `255.255.224.0` |
| R-TIMUR | Sensor Banjir Timur | 700 | 702 | `/22` | 1.024 | `255.255.252.0` |
| R-BARAT | WiFi Publik Barat | 4.500 | 4.502 | `/19` | 8.192 | `255.255.224.0` |
| R-BARAT | Puskesmas dan Sekolah Barat | 220 | 222 | `/24` | 256 | `255.255.255.0` |

## Tabel Subnetting VLSM

| No | Nama Subnet | Router | Host | Prefix | Ukuran Blok | Network ID | Subnet Mask | Rentang Host | Broadcast | Gateway |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | WiFi Publik Utara | R-UTARA | 11.000 | `/18` | 16.384 | `10.32.0.0` | `255.255.192.0` | `10.32.0.1 - 10.32.63.254` | `10.32.63.255` | `10.32.0.1` |
| 2 | WiFi Publik Selatan | R-SELATAN | 6.500 | `/19` | 8.192 | `10.32.64.0` | `255.255.224.0` | `10.32.64.1 - 10.32.95.254` | `10.32.95.255` | `10.32.64.1` |
| 3 | CCTV Kota | R-DC | 6.000 | `/19` | 8.192 | `10.32.96.0` | `255.255.224.0` | `10.32.96.1 - 10.32.127.254` | `10.32.127.255` | `10.32.96.1` |
| 4 | WiFi Publik Timur | R-TIMUR | 5.500 | `/19` | 8.192 | `10.32.128.0` | `255.255.224.0` | `10.32.128.1 - 10.32.159.254` | `10.32.159.255` | `10.32.128.1` |
| 5 | Sensor dan SCADA Pelabuhan | R-PELABUHAN | 5.000 | `/19` | 8.192 | `10.32.160.0` | `255.255.224.0` | `10.32.160.1 - 10.32.191.254` | `10.32.191.255` | `10.32.160.1` |
| 6 | WiFi Publik Barat | R-BARAT | 4.500 | `/19` | 8.192 | `10.32.192.0` | `255.255.224.0` | `10.32.192.1 - 10.32.223.254` | `10.32.223.255` | `10.32.192.1` |
| 7 | Pegawai Balai Kota | R-PUSAT | 1.800 | `/21` | 2.048 | `10.32.224.0` | `255.255.248.0` | `10.32.224.1 - 10.32.231.254` | `10.32.231.255` | `10.32.224.1` |
| 8 | Sensor Banjir Timur | R-TIMUR | 700 | `/22` | 1.024 | `10.32.232.0` | `255.255.252.0` | `10.32.232.1 - 10.32.235.254` | `10.32.235.255` | `10.32.232.1` |
| 9 | Sensor Lalu Lintas Utara | R-UTARA | 450 | `/23` | 512 | `10.32.236.0` | `255.255.254.0` | `10.32.236.1 - 10.32.237.254` | `10.32.237.255` | `10.32.236.1` |
| 10 | Puskesmas dan Sekolah Barat | R-BARAT | 220 | `/24` | 256 | `10.32.238.0` | `255.255.255.0` | `10.32.238.1 - 10.32.238.254` | `10.32.238.255` | `10.32.238.1` |
| 11 | Server dan Storage DC | R-DC | 180 | `/24` | 256 | `10.32.239.0` | `255.255.255.0` | `10.32.239.1 - 10.32.239.254` | `10.32.239.255` | `10.32.239.1` |
| 12 | Kantor Pelabuhan | R-PELABUHAN | 120 | `/25` | 128 | `10.32.240.0` | `255.255.255.128` | `10.32.240.1 - 10.32.240.126` | `10.32.240.127` | `10.32.240.1` |
| 13 | PJU Pintar Selatan | R-SELATAN | 90 | `/25` | 128 | `10.32.240.128` | `255.255.255.128` | `10.32.240.129 - 10.32.240.254` | `10.32.240.255` | `10.32.240.129` |
| 14 | Server Balai Kota | R-PUSAT | 20 | `/27` | 32 | `10.32.241.0` | `255.255.255.224` | `10.32.241.1 - 10.32.241.30` | `10.32.241.31` | `10.32.241.1` |
| 15 | L1 (R-PUSAT - R-DC) | R-PUSAT - R-DC | 2 | `/30` | 4 | `10.32.241.32` | `255.255.255.252` | `10.32.241.33 - 10.32.241.34` | `10.32.241.35` | `10.32.241.33` |
| 16 | L2 (R-PUSAT - R-UTARA) | R-PUSAT - R-UTARA | 2 | `/30` | 4 | `10.32.241.36` | `255.255.255.252` | `10.32.241.37 - 10.32.241.38` | `10.32.241.39` | `10.32.241.37` |
| 17 | L3 (R-PUSAT - R-SELATAN) | R-PUSAT - R-SELATAN | 2 | `/30` | 4 | `10.32.241.40` | `255.255.255.252` | `10.32.241.41 - 10.32.241.42` | `10.32.241.43` | `10.32.241.41` |
| 18 | L4 (R-PUSAT - R-TIMUR) | R-PUSAT - R-TIMUR | 2 | `/30` | 4 | `10.32.241.44` | `255.255.255.252` | `10.32.241.45 - 10.32.241.46` | `10.32.241.47` | `10.32.241.45` |
| 19 | L5 (R-PUSAT - R-BARAT) | R-PUSAT - R-BARAT | 2 | `/30` | 4 | `10.32.241.48` | `255.255.255.252` | `10.32.241.49 - 10.32.241.50` | `10.32.241.51` | `10.32.241.49` |
| 20 | L6 (R-UTARA - R-PELABUHAN) | R-UTARA - R-PELABUHAN | 2 | `/30` | 4 | `10.32.241.52` | `255.255.255.252` | `10.32.241.53 - 10.32.241.54` | `10.32.241.55` | `10.32.241.53` |

## Tabel Subnetting CIDR

Blok router dialokasikan berurutan dari alamat dasar. Subnet link L1 sampai L6 berada di blok `10.33.208.0/27`.

| No | Nama Subnet | Router | Host | Prefix | Ukuran Blok | Network ID | Subnet Mask | Rentang Host | Broadcast | Gateway |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | WiFi Publik Utara | R-UTARA | 11.000 | `/18` | 16.384 | `10.32.0.0` | `255.255.192.0` | `10.32.0.1 - 10.32.63.254` | `10.32.63.255` | `10.32.0.1` |
| 2 | Sensor Lalu Lintas Utara | R-UTARA | 450 | `/23` | 512 | `10.32.64.0` | `255.255.254.0` | `10.32.64.1 - 10.32.65.254` | `10.32.65.255` | `10.32.64.1` |
| 3 | WiFi Publik Timur | R-TIMUR | 5.500 | `/19` | 8.192 | `10.32.128.0` | `255.255.224.0` | `10.32.128.1 - 10.32.159.254` | `10.32.159.255` | `10.32.128.1` |
| 4 | Sensor Banjir Timur | R-TIMUR | 700 | `/22` | 1.024 | `10.32.160.0` | `255.255.252.0` | `10.32.160.1 - 10.32.163.254` | `10.32.163.255` | `10.32.160.1` |
| 5 | CCTV Kota | R-DC | 6.000 | `/19` | 8.192 | `10.32.192.0` | `255.255.224.0` | `10.32.192.1 - 10.32.223.254` | `10.32.223.255` | `10.32.192.1` |
| 6 | Server dan Storage DC | R-DC | 180 | `/24` | 256 | `10.32.224.0` | `255.255.255.0` | `10.32.224.1 - 10.32.224.254` | `10.32.224.255` | `10.32.224.1` |
| 7 | WiFi Publik Barat | R-BARAT | 4.500 | `/19` | 8.192 | `10.33.0.0` | `255.255.224.0` | `10.33.0.1 - 10.33.31.254` | `10.33.31.255` | `10.33.0.1` |
| 8 | Puskesmas dan Sekolah Barat | R-BARAT | 220 | `/24` | 256 | `10.33.32.0` | `255.255.255.0` | `10.33.32.1 - 10.33.32.254` | `10.33.32.255` | `10.33.32.1` |
| 9 | Sensor dan SCADA Pelabuhan | R-PELABUHAN | 5.000 | `/19` | 8.192 | `10.33.64.0` | `255.255.224.0` | `10.33.64.1 - 10.33.95.254` | `10.33.95.255` | `10.33.64.1` |
| 10 | Kantor Pelabuhan | R-PELABUHAN | 120 | `/25` | 128 | `10.33.96.0` | `255.255.255.128` | `10.33.96.1 - 10.33.96.126` | `10.33.96.127` | `10.33.96.1` |
| 11 | WiFi Publik Selatan | R-SELATAN | 6.500 | `/19` | 8.192 | `10.33.128.0` | `255.255.224.0` | `10.33.128.1 - 10.33.159.254` | `10.33.159.255` | `10.33.128.1` |
| 12 | PJU Pintar Selatan | R-SELATAN | 90 | `/25` | 128 | `10.33.160.0` | `255.255.255.128` | `10.33.160.1 - 10.33.160.126` | `10.33.160.127` | `10.33.160.1` |
| 13 | Pegawai Balai Kota | R-PUSAT | 1.800 | `/21` | 2.048 | `10.33.192.0` | `255.255.248.0` | `10.33.192.1 - 10.33.199.254` | `10.33.199.255` | `10.33.192.1` |
| 14 | Server Balai Kota | R-PUSAT | 20 | `/27` | 32 | `10.33.200.0` | `255.255.255.224` | `10.33.200.1 - 10.33.200.30` | `10.33.200.31` | `10.33.200.1` |
| 15 | L1 (R-PUSAT - R-DC) | R-PUSAT - R-DC | 2 | `/30` | 4 | `10.33.208.0` | `255.255.255.252` | `10.33.208.1 - 10.33.208.2` | `10.33.208.3` | `10.33.208.1` |
| 16 | L2 (R-PUSAT - R-UTARA) | R-PUSAT - R-UTARA | 2 | `/30` | 4 | `10.33.208.4` | `255.255.255.252` | `10.33.208.5 - 10.33.208.6` | `10.33.208.7` | `10.33.208.5` |
| 17 | L3 (R-PUSAT - R-SELATAN) | R-PUSAT - R-SELATAN | 2 | `/30` | 4 | `10.33.208.8` | `255.255.255.252` | `10.33.208.9 - 10.33.208.10` | `10.33.208.11` | `10.33.208.9` |
| 18 | L4 (R-PUSAT - R-TIMUR) | R-PUSAT - R-TIMUR | 2 | `/30` | 4 | `10.33.208.12` | `255.255.255.252` | `10.33.208.13 - 10.33.208.14` | `10.33.208.15` | `10.33.208.13` |
| 19 | L5 (R-PUSAT - R-BARAT) | R-PUSAT - R-BARAT | 2 | `/30` | 4 | `10.33.208.16` | `255.255.255.252` | `10.33.208.17 - 10.33.208.18` | `10.33.208.19` | `10.33.208.17` |
| 20 | L6 (R-UTARA - R-PELABUHAN) | R-UTARA - R-PELABUHAN | 2 | `/30` | 4 | `10.33.208.20` | `255.255.255.252` | `10.33.208.21 - 10.33.208.22` | `10.33.208.23` | `10.33.208.21` |

### Penggabungan (supernet)

| Router / Blok | Anggota (LAN, prefix, jumlah alamat) | Jumlah Alamat Anggota | Prefix Blok | Ukuran Blok | Network ID Blok | Rentang Alamat Blok |
|---|---|---|---|---|---|---|
| R-UTARA | WiFi Publik Utara (/18, 16.384); Sensor Lalu Lintas Utara (/23, 512) | 16.896 | `/17` | 32.768 | `10.32.0.0` | `10.32.0.0 - 10.32.127.255` |
| R-TIMUR | WiFi Publik Timur (/19, 8.192); Sensor Banjir Timur (/22, 1.024) | 9.216 | `/18` | 16.384 | `10.32.128.0` | `10.32.128.0 - 10.32.191.255` |
| R-DC | CCTV Kota (/19, 8.192); Server dan Storage DC (/24, 256) | 8.448 | `/18` | 16.384 | `10.32.192.0` | `10.32.192.0 - 10.32.255.255` |
| R-BARAT | WiFi Publik Barat (/19, 8.192); Puskesmas dan Sekolah Barat (/24, 256) | 8.448 | `/18` | 16.384 | `10.33.0.0` | `10.33.0.0 - 10.33.63.255` |
| R-PELABUHAN | Sensor dan SCADA Pelabuhan (/19, 8.192); Kantor Pelabuhan (/25, 128) | 8.320 | `/18` | 16.384 | `10.33.64.0` | `10.33.64.0 - 10.33.127.255` |
| R-SELATAN | WiFi Publik Selatan (/19, 8.192); PJU Pintar Selatan (/25, 128) | 8.320 | `/18` | 16.384 | `10.33.128.0` | `10.33.128.0 - 10.33.191.255` |
| R-PUSAT | Pegawai Balai Kota (/21, 2.048); Server Balai Kota (/27, 32) | 2.080 | `/20` | 4.096 | `10.33.192.0` | `10.33.192.0 - 10.33.207.255` |
| Link backbone L1-L6 | 6 subnet link /30 (6 x 4 alamat) | 24 | `/27` | 32 | `10.33.208.0` | `10.33.208.0 - 10.33.208.31` |
| TOTAL (akar pohon) | Seluruh blok router + blok link | 118.816 | `/15` | 131.072 | `10.32.0.0` | `10.32.0.0 - 10.33.255.255` |

Jumlah alamat anggota = total ukuran blok LAN. Prefix blok = prefix terkecil yang menampung jumlah tersebut.

## Alamat Tambahan

Tidak dihitung saat UTS.

| Keperluan | Alamat | Keterangan |
|---|---|---|
| Link R-PUSAT ke R-ISP | `203.0.113.16/30` | 203.0.113.(4 x K)/30. R-PUSAT .17, R-ISP .18 |
| Server internet | `8.8.8.0/24` | Server 8.8.8.8, gateway 8.8.8.1 di R-ISP |
| Tunnel GRE | `192.168.4.0/30` | 192.168.K.0/30. Tunnel R-PUSAT .1, tunnel R-PELABUHAN .2 |

## Tabel Alamat Interface dan Perangkat

Skema VLSM. Nama interface `Gi0/x` berurutan dan dapat disesuaikan dengan modul port yang dipasang di Packet Tracer. Router memakai host pertama subnet (gateway), perangkat perwakilan memakai host kedua. Pada link `/30`, R-PUSAT dan R-UTARA (sisi dekat pusat) memakai host pertama.

| Perangkat | Interface | Terhubung ke | Alamat IP | Prefix | Subnet Mask | Default Gateway |
|---|---|---|---|---|---|---|
| R-PUSAT | `Gi0/0` | SW-PEGAWAI-BALAIKOTA | `10.32.224.1` | `/21` | `255.255.248.0` | - |
| R-PUSAT | `Gi0/1` | SW-SERVER-BALAIKOTA | `10.32.241.1` | `/27` | `255.255.255.224` | - |
| R-PUSAT | `Gi0/2` | R-DC (L1) | `10.32.241.33` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Gi0/3` | R-UTARA (L2) | `10.32.241.37` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Gi0/4` | R-SELATAN (L3) | `10.32.241.41` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Gi0/5` | R-TIMUR (L4) | `10.32.241.45` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Gi0/6` | R-BARAT (L5) | `10.32.241.49` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Gi0/7` | R-ISP | `203.0.113.17` | `/30` | `255.255.255.252` | - |
| R-PUSAT | `Tunnel0` | R-PELABUHAN (GRE) | `192.168.4.1` | `/30` | `255.255.255.252` | - |
| R-DC | `Gi0/0` | R-PUSAT (L1) | `10.32.241.34` | `/30` | `255.255.255.252` | - |
| R-DC | `Gi0/1` | SW-CCTV-KOTA | `10.32.96.1` | `/19` | `255.255.224.0` | - |
| R-DC | `Gi0/2` | SW-SERVER-DC | `10.32.239.1` | `/24` | `255.255.255.0` | - |
| R-BARAT | `Gi0/0` | R-PUSAT (L5) | `10.32.241.50` | `/30` | `255.255.255.252` | - |
| R-BARAT | `Gi0/1` | SW-WIFI-BARAT | `10.32.192.1` | `/19` | `255.255.224.0` | - |
| R-BARAT | `Gi0/2` | SW-PUSKESMAS-SEKOLAH-BARAT | `10.32.238.1` | `/24` | `255.255.255.0` | - |
| R-UTARA | `Gi0/0` | R-PUSAT (L2) | `10.32.241.38` | `/30` | `255.255.255.252` | - |
| R-UTARA | `Gi0/1` | SW-WIFI-UTARA | `10.32.0.1` | `/18` | `255.255.192.0` | - |
| R-UTARA | `Gi0/2` | SW-SENSOR-LALIN-UTARA | `10.32.236.1` | `/23` | `255.255.254.0` | - |
| R-UTARA | `Gi0/3` | R-PELABUHAN (L6) | `10.32.241.53` | `/30` | `255.255.255.252` | - |
| R-TIMUR | `Gi0/0` | R-PUSAT (L4) | `10.32.241.46` | `/30` | `255.255.255.252` | - |
| R-TIMUR | `Gi0/1` | SW-WIFI-TIMUR | `10.32.128.1` | `/19` | `255.255.224.0` | - |
| R-TIMUR | `Gi0/2` | SW-SENSOR-BANJIR-TIMUR | `10.32.232.1` | `/22` | `255.255.252.0` | - |
| R-SELATAN | `Gi0/0` | R-PUSAT (L3) | `10.32.241.42` | `/30` | `255.255.255.252` | - |
| R-SELATAN | `Gi0/1` | SW-WIFI-SELATAN | `10.32.64.1` | `/19` | `255.255.224.0` | - |
| R-SELATAN | `Gi0/2` | SW-PJU-SELATAN | `10.32.240.129` | `/25` | `255.255.255.128` | - |
| R-PELABUHAN | `Gi0/0` | R-UTARA (L6) | `10.32.241.54` | `/30` | `255.255.255.252` | - |
| R-PELABUHAN | `Gi0/1` | SW-SCADA-PELABUHAN | `10.32.160.1` | `/19` | `255.255.224.0` | - |
| R-PELABUHAN | `Gi0/2` | SW-KANTOR-PELABUHAN | `10.32.240.1` | `/25` | `255.255.255.128` | - |
| R-PELABUHAN | `Tunnel0` | R-PUSAT (GRE) | `192.168.4.2` | `/30` | `255.255.255.252` | - |
| R-ISP | `Gi0/0` | R-PUSAT | `203.0.113.18` | `/30` | `255.255.255.252` | - |
| R-ISP | `Gi0/1` | SRV-INTERNET | `8.8.8.1` | `/24` | `255.255.255.0` | - |
| PC-PEGAWAI-BALAIKOTA | `NIC / Wireless` | Switch / AP LAN | `10.32.224.2` | `/21` | `255.255.248.0` | `10.32.224.1` |
| SRV-BALAIKOTA | `NIC / Wireless` | Switch / AP LAN | `10.32.241.2` | `/27` | `255.255.255.224` | `10.32.241.1` |
| PC-CCTV-KOTA | `NIC / Wireless` | Switch / AP LAN | `10.32.96.2` | `/19` | `255.255.224.0` | `10.32.96.1` |
| SRV-DC | `NIC / Wireless` | Switch / AP LAN | `10.32.239.2` | `/24` | `255.255.255.0` | `10.32.239.1` |
| SP-WIFI-BARAT | `NIC / Wireless` | Switch / AP LAN | `10.32.192.2` | `/19` | `255.255.224.0` | `10.32.192.1` |
| PC-PUSKESMAS-SEKOLAH-BARAT | `NIC / Wireless` | Switch / AP LAN | `10.32.238.2` | `/24` | `255.255.255.0` | `10.32.238.1` |
| LAPTOP-WIFI-UTARA | `NIC / Wireless` | Switch / AP LAN | `10.32.0.2` | `/18` | `255.255.192.0` | `10.32.0.1` |
| PC-SENSOR-LALIN-UTARA | `NIC / Wireless` | Switch / AP LAN | `10.32.236.2` | `/23` | `255.255.254.0` | `10.32.236.1` |
| LAPTOP-WIFI-TIMUR | `NIC / Wireless` | Switch / AP LAN | `10.32.128.2` | `/19` | `255.255.224.0` | `10.32.128.1` |
| PC-SENSOR-BANJIR-TIMUR | `NIC / Wireless` | Switch / AP LAN | `10.32.232.2` | `/22` | `255.255.252.0` | `10.32.232.1` |
| SP-WIFI-SELATAN | `NIC / Wireless` | Switch / AP LAN | `10.32.64.2` | `/19` | `255.255.224.0` | `10.32.64.1` |
| PC-PJU-SELATAN | `NIC / Wireless` | Switch / AP LAN | `10.32.240.130` | `/25` | `255.255.255.128` | `10.32.240.129` |
| PC-SCADA-PELABUHAN | `NIC / Wireless` | Switch / AP LAN | `10.32.160.2` | `/19` | `255.255.224.0` | `10.32.160.1` |
| PC-KANTOR-PELABUHAN | `NIC / Wireless` | Switch / AP LAN | `10.32.240.2` | `/25` | `255.255.255.128` | `10.32.240.1` |
| SRV-INTERNET | `NIC / Wireless` | Switch / AP LAN | `8.8.8.8` | `/24` | `255.255.255.0` | `8.8.8.1` |

## Tabel Revisi

Kesalahan hitung pada jawaban UTS diperbaiki dan dicatat pada tabel berikut. Jawaban di UTS dibaca dari foto lembar VLSM (no 3). Subnet yang tidak tercantum di sini sama dengan lembar UTS.

| Subnet | Jawaban di UTS | Perbaikan | Alasan Kesalahan |
|---|---|---|---|
| Kantor Pelabuhan (R-PELABUHAN, /25) - subnet mask | 255.225.255.128 | 255.255.255.128 | Oktet ke-2 salah tulis (225). Mask /25 yang benar adalah 255.255.255.128. Prefix /25 dan blok 128 tidak berubah. |
| Sensor Banjir Timur (/22) | Network 10.32.236.0, broadcast 10.32.237.255 | Network 10.32.232.0, rentang 10.32.232.1 - 10.32.235.254, broadcast 10.32.235.255 | Blok /22 = 1.024 alamat (4 blok /24), tetapi rentang UTS hanya 512 alamat. Network ID juga harus lanjut tepat setelah Pegawai Balai Kota (berakhir 10.32.231.255). |
| Sensor Lalu Lintas Utara (/23) | Network 10.32.238.0, rentang 10.32.238.1 - 10.32.238.254, broadcast 10.32.238.255 | Network 10.32.236.0, rentang 10.32.236.1 - 10.32.237.254, broadcast 10.32.237.255 | Blok /23 = 512 alamat, tetapi rentang UTS hanya 256 alamat (seperti /24). Network ID harus lanjut dari 10.32.235.255. |
| Puskesmas dan Sekolah Barat (/24) | Network 10.32.239.0, rentang 10.32.239.1 - 10.32.239.254, broadcast 10.32.239.255 | Network 10.32.238.0, rentang 10.32.238.1 - 10.32.238.254, broadcast 10.32.238.255 | Network ID bergeser akibat kesalahan pada subnet sebelumnya. Harus lanjut dari 10.32.237.255. |
| Server dan Storage DC (/24) | Network 10.32.240.0, rentang 10.32.240.1 - 10.32.240.126, broadcast 10.32.240.127 | Network 10.32.239.0, rentang 10.32.239.1 - 10.32.239.254, broadcast 10.32.239.255 | Prefix /24 = 256 alamat (broadcast .255), tetapi rentang UTS hanya 128 alamat (seperti /25). Network ID juga bergeser akibat kesalahan sebelumnya. |
| Kantor Pelabuhan (/25) - alokasi | Network 10.32.240.128, rentang 10.32.240.129 - 10.32.240.254, broadcast 10.32.240.255 | Network 10.32.240.0, rentang 10.32.240.1 - 10.32.240.126, broadcast 10.32.240.127 | Network ID bergeser akibat kesalahan pada subnet sebelumnya. Harus lanjut dari 10.32.239.255. |
| PJU Pintar Selatan (/25) | Network 10.32.241.0, rentang 10.32.241.1 - 10.32.241.62, broadcast 10.32.241.63 | Network 10.32.240.128, rentang 10.32.240.129 - 10.32.240.254, broadcast 10.32.240.255 | Blok /25 = 128 alamat, tetapi rentang UTS hanya 64 alamat (seperti /26). Network ID harus lanjut dari 10.32.240.127. |
| Server Balai Kota (/27) | Network 10.32.241.64, rentang 10.32.241.65 - 10.32.241.254, broadcast 10.32.241.255 | Network 10.32.241.0, rentang 10.32.241.1 - 10.32.241.30, broadcast 10.32.241.31 | Blok /27 = 32 alamat, tetapi rentang UTS sampai .254 (seperti /24). Network ID harus lanjut dari 10.32.240.255. |
| Subnet link L1-L6 (6 subnet /30) | Tidak ada di lembar UTS | Ditambahkan: L1-L6 sebagai /30, dialokasikan setelah blok LAN | Backbone Gambar 1 membutuhkan alamat untuk setiap link. Penambahan tidak mengubah alokasi 14 LAN. |

---

# Soal 3: Static Routing

## Penjelasan Konfigurasi

Seluruh komunikasi antar-LAN dikonfigurasi menggunakan *static route* dan *default route* dengan hierarki routing sebagai berikut:

### 1. Default route ke arah atas / keluar

Setiap router wilayah dan pusat memiliki satu *default route* (`0.0.0.0 0.0.0.0`) yang mengarah langsung ke router satu tingkat di atasnya (atau keluar menuju R-ISP).

| Router | Perintah Default Route | Next-Hop (Arah) |
|---|---|---|
| R-DC | `ip route 0.0.0.0 0.0.0.0 10.32.241.33` | R-PUSAT (L1) |
| R-UTARA | `ip route 0.0.0.0 0.0.0.0 10.32.241.37` | R-PUSAT (L2) |
| R-SELATAN | `ip route 0.0.0.0 0.0.0.0 10.32.241.41` | R-PUSAT (L3) |
| R-TIMUR | `ip route 0.0.0.0 0.0.0.0 10.32.241.45` | R-PUSAT (L4) |
| R-BARAT | `ip route 0.0.0.0 0.0.0.0 10.32.241.49` | R-PUSAT (L5) |
| R-PELABUHAN | `ip route 0.0.0.0 0.0.0.0 10.32.241.53` | R-UTARA (L6) |
| R-PUSAT | `ip route 0.0.0.0 0.0.0.0 203.0.113.18` | R-ISP (Internet) |

### 2. Static route ke arah dalam (R-PUSAT dan R-UTARA)

Agar lalu lintas balik menuju LAN wilayah dapat diteruskan dengan tepat, **R-PUSAT** dan **R-UTARA** dilengkapi *static route* spesifik ke seluruh subnet di bawahnya.

Konfigurasi pada **R-UTARA** (menuju seluruh LAN di bawah R-PELABUHAN):

```
ip route 10.32.160.0 255.255.224.0 10.32.241.54
ip route 10.32.240.0 255.255.255.128 10.32.241.54
```

Konfigurasi pada **R-PUSAT** (menuju seluruh LAN milik R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, dan R-BARAT):

```
! Rute ke R-DC
ip route 10.32.96.0 255.255.224.0 10.32.241.34
ip route 10.32.239.0 255.255.255.0 10.32.241.34

! Rute ke R-UTARA (termasuk Pelabuhan dan link L6)
ip route 10.32.0.0 255.255.192.0 10.32.241.38
ip route 10.32.236.0 255.255.254.0 10.32.241.38
ip route 10.32.160.0 255.255.224.0 10.32.241.38
ip route 10.32.240.0 255.255.255.128 10.32.241.38
ip route 10.32.241.52 255.255.255.252 10.32.241.38

! Rute ke R-SELATAN
ip route 10.32.64.0 255.255.224.0 10.32.241.42
ip route 10.32.240.128 255.255.255.128 10.32.241.42

! Rute ke R-TIMUR
ip route 10.32.128.0 255.255.224.0 10.32.241.46
ip route 10.32.232.0 255.255.252.0 10.32.241.46

! Rute ke R-BARAT
ip route 10.32.192.0 255.255.224.0 10.32.241.50
ip route 10.32.238.0 255.255.255.0 10.32.241.50
```

## Bukti Pengujian Mandiri

| No | Pengujian | Perintah / cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Tabel routing R-PUSAT | `show ip route` di R-PUSAT | Terdapat rute statis (`S`) ke seluruh LAN wilayah dan default route (`S*`) ke `203.0.113.18` | <img width="436" height="238" alt="image" src="https://github.com/user-attachments/assets/b7fa2268-e721-4ae4-a831-6527f944dc8a" />
|
| 2 | Tabel routing R-UTARA dan router cabang | `show ip route` di R-UTARA dan R-SELATAN | R-UTARA memiliki rute ke Pelabuhan dan default route ke R-PUSAT; router cabang hanya memiliki default route ke atas | !<img width="433" height="209" alt="image" src="https://github.com/user-attachments/assets/a2d2c6f7-796c-4ea3-a607-d9cb93098e21" />
 |
| 3 | Konektivitas dan jalur antar-wilayah | `ping 10.32.232.11` dan `tracert 10.32.232.11` dari PC-PUSKESMAS-SEKOLAH-BARAT | Ping `Reply`, jalur melewati R-BARAT (`10.32.238.1`) -> R-PUSAT (`10.32.241.49`) -> R-TIMUR (`10.32.241.46`) -> tujuan | <img width="388" height="270" alt="image" src="https://github.com/user-attachments/assets/73b9bc7b-b211-4ef9-bc89-937221efddce" />
 |

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

# Soal 5: NAT

## Penjelasan Konfigurasi

Agar seluruh host internal yang menggunakan IP privat `10.32.0.0` dapat mengakses server internet `8.8.8.8` tanpa memberikan rute blok privat ke **R-ISP**, dikonfigurasi **NAT Overload (PAT)** pada **R-PUSAT**. Seluruh paket keluar dari jaringan internal ditranslasikan menggunakan satu IP publik milik interface eksternal R-PUSAT (`203.0.113.17`).

Konfigurasi pada **R-PUSAT** (nama interface mengikuti tabel alamat interface pada Soal 2):

```
interface GigabitEthernet0/7
 ip nat outside
 exit

interface GigabitEthernet0/0
 ip nat inside
 exit
interface GigabitEthernet0/1
 ip nat inside
 exit
interface GigabitEthernet0/2
 ip nat inside
 exit
interface GigabitEthernet0/3
 ip nat inside
 exit
interface GigabitEthernet0/4
 ip nat inside
 exit
interface GigabitEthernet0/5
 ip nat inside
 exit
interface GigabitEthernet0/6
 ip nat inside
 exit

access-list 1 permit any
ip nat inside source list 1 interface GigabitEthernet0/7 overload
```

Penjelasan perintah:

| Perintah | Fungsi |
|---|---|
| `ip nat outside` | Menandai interface yang terhubung ke jaringan publik (R-ISP, `203.0.113.17`) |
| `ip nat inside` | Menandai seluruh interface yang mengarah ke LAN internal dan backbone wilayah |
| `access-list 1 permit any` | Mengizinkan lalu lintas dari alamat privat internal untuk ditranslasikan |
| `ip nat inside source list 1 interface ... overload` | Mengaktifkan PAT (*Port Address Translation*) sehingga banyak host internal berbagi satu IP publik `203.0.113.17` |

Pada **R-ISP**, tidak dikonfigurasi static route maupun default route menuju blok alamat privat `10.32.0.0`. Tabel routing R-ISP hanya memuat jaringan publik yang terhubung langsung (`203.0.113.16/30` dan `8.8.8.0/24`).

## Bukti Pengujian Mandiri

| No | Pengujian | Perintah / cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Akses internet dari wilayah Barat dan Selatan | `ping 8.8.8.8` dan `tracert 8.8.8.8` dari PC-PUSKESMAS-SEKOLAH-BARAT dan PC-PJU-SELATAN | Ping `Reply` dari `8.8.8.8` di kedua wilayah | ![](docs/img/soal5-ping-8-8-8-8.png) |
| 2 | Translasi alamat aktif di R-PUSAT | `show ip nat translations` dan `show ip nat statistics` di R-PUSAT | Inside local (`10.32.x.x`) ditranslasikan ke Inside global (`203.0.113.17`) | ![](docs/img/soal5-show-ip-nat.png) |
| 3 | R-ISP tidak memiliki rute ke blok privat | `show ip route` di R-ISP | Hanya terdapat rute `203.0.113.16/30` dan `8.8.8.0/24`, tanpa rute `10.32.0.0` | ![](docs/img/soal5-show-ip-route-r-isp.png) |

---

# Soal 6: Tunnel GRE

## Penjelasan Konfigurasi

Jalur virtual *point-to-point* dibangun menggunakan antarmuka **Tunnel GRE (`Tunnel0`)** antara **R-PUSAT** dan **R-PELABUHAN** dengan blok alamat `192.168.4.0/30` (melewati jalur fisik L2 dan L6 via R-UTARA). Rute statis fisik lama dari R-PUSAT ke Pelabuhan dihapus dan diganti dengan rute statis yang mengarahkan lalu lintas antara subnet Balai Kota dan subnet Pelabuhan melalui IP tunnel.

Konfigurasi pada **R-PUSAT**:

```
no ip route 10.32.160.0 255.255.224.0 10.32.241.38
no ip route 10.32.240.0 255.255.255.128 10.32.241.38

interface Tunnel0
 ip address 192.168.4.1 255.255.255.252
 tunnel source GigabitEthernet0/3
 tunnel destination 10.32.241.54
 exit

ip route 10.32.160.0 255.255.224.0 192.168.4.2
ip route 10.32.240.0 255.255.255.128 192.168.4.2
```

Konfigurasi pada **R-PELABUHAN**:

```
interface Tunnel0
 ip address 192.168.4.2 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 10.32.241.37
 exit

ip route 10.32.224.0 255.255.248.0 192.168.4.1
ip route 10.32.241.0 255.255.255.224 192.168.4.1
```

### Analisis Pemilihan Jalur dan Keamanan GRE

| Pertanyaan | Penjelasan |
|---|---|
| Mengapa router memilih jalur tunnel dibandingkan jalur lain ke pelabuhan? | Router menerapkan aturan *Longest Prefix Match* (pencocokan prefix paling spesifik) saat memilih rute. Pada **R-PUSAT**, rute fisik melalui R-UTARA (`10.32.241.38`) telah dihapus dan digantikan oleh *static route* menuju subnet Pelabuhan (`/19` dan `/25`) dengan *next-hop* `192.168.4.2` (antarmuka `Tunnel0`). Pada **R-PELABUHAN**, *static route* menuju subnet Balai Kota (`10.32.224.0/21` dan `10.32.241.0/27`) via `192.168.4.1` memiliki prefix lebih spesifik dibandingkan *default route* (`0.0.0.0/0`), sehingga router memprioritaskan pengiriman paket melalui lorong virtual `Tunnel0` dan melewati R-UTARA secara transparan (tidak terlihat sebagai hop IP terpisah). |
| Mengapa GRE saja tidak mengenkripsi data? | GRE (*Generic Routing Encapsulation*) merupakan protokol *tunneling* dan enkapsulasi murni yang hanya membungkus paket IP asli dengan *header* GRE dan *header* IP luar baru agar paket dapat melintasi jaringan perantara. GRE tidak dilengkapi mekanisme kriptografi (seperti kerahasiaan/enkripsi maupun integritas data), sehingga muatan paket (*payload*) di dalamnya tetap dikirimkan dalam bentuk teks terang (*plaintext*) dan tetap rentan disadap (*eavesdropping*) apabila tidak dikombinasikan dengan protokol keamanan seperti IPsec. |

## Bukti Pengujian Mandiri

| No | Pengujian | Perintah / cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Status dan konfigurasi Tunnel0 di kedua sisi | `show interface Tunnel0` di R-PUSAT dan R-PELABUHAN | Status `Tunnel0 is up, line protocol is up`, IP `192.168.4.1/30` dan `192.168.4.2/30`, protokol `GRE/IP` | ![](docs/img/soal6-show-interface-tunnel0.png) |
| 2 | Lalu lintas Balai Kota ke Pelabuhan melewati tunnel | `tracert 10.32.240.11` dari PC-PEGAWAI-BALAIKOTA | Hop melewati gateway `10.32.224.1` -> IP tunnel `192.168.4.2` -> tujuan `10.32.240.11` (R-UTARA tidak muncul sebagai hop) | ![](docs/img/soal6-tracert-via-tunnel.png) |

---
# Soal 7: OSPF dan Failover

## Penjelasan Konfigurasi

### 1. Link cadangan L7 (R-DC dan R-UTARA)

Untuk membuat jalur cadangan, R-DC dan R-UTARA dihubungkan langsung memakai kabel Copper Cross-Over pada interface GigabitEthernet0/2 di kedua router. Link ini memakai subnet /30 baru (2 host).

| Item | Nilai |
|---|---|
| Nama subnet | L7 (R-DC - R-UTARA) |
| Network ID | 10.32.241.56 |
| Subnet mask | 255.255.255.252 (/30) |
| Rentang host | 10.32.241.57 - 10.32.241.58 |
| Broadcast | 10.32.241.59 |
| Interface R-DC | Gi0/2, 10.32.241.57 /30 |
| Interface R-UTARA | Gi0/2, 10.32.241.58 /30 |

```
! R-DC
interface GigabitEthernet0/2
 ip address 10.32.241.57 255.255.255.252
 no shutdown

! R-UTARA
interface GigabitEthernet0/2
 ip address 10.32.241.58 255.255.255.252
 no shutdown
```

Konfigurasi interface L7 di R-UTARA (Gi0/2 berubah menjadi up/up setelah `no shutdown`):

<img width="2880" height="1702" alt="Screenshot 2026-11-15 180050" src="https://github.com/user-attachments/assets/0aa8ee2b-49af-4111-9e67-00414d115d71" />

Setelah kedua sisi dikonfigurasi, link L7 diuji dengan ping dari R-DC ke R-UTARA:

<img width="1110" height="538" alt="Screenshot 2026-11-15 175549" src="https://github.com/user-attachments/assets/f1665b37-c969-41da-90f7-5a9cf3d21ef0" />

Ping pertama bernilai `.` (timeout) karena proses ARP, empat ping berikutnya berhasil (`!!!!`). Success rate 80 persen (4/5) adalah hal yang normal.

### 2. Menghapus static route

Sebelum OSPF dipakai, semua routing memakai static route. Static route harus dihapus supaya tabel routing hanya berisi rute hasil OSPF. Kalau tidak dihapus, static route (AD 1) lebih diprioritaskan daripada OSPF (AD 110) dan failover tidak akan terlihat.

Contoh penghapusan pada R-PUSAT:

```
no ip route 10.32.96.0 255.255.224.0 10.32.241.34
no ip route 10.32.239.0 255.255.255.0 10.32.241.34
no ip route 10.32.0.0 255.255.192.0 10.32.241.38
... (seluruh static route ke wilayah lain)
```

<img width="1402" height="638" alt="Screenshot 2026-11-15 175303" src="https://github.com/user-attachments/assets/e8f092ea-5031-4f9e-9650-c7340632c107" />


Pesan `%No matching route to delete` pada dua baris terakhir muncul karena route tersebut memang tidak ada. Ini bukan error.

Setelah itu, routing table R-PUSAT hanya berisi rute connected (`C` dan `L`) dan default route ke internet (`Gateway of last resort is 203.0.113.18`). Default route ini dipertahankan dan disebarkan ke router lain lewat `default-information originate`.

<img width="1240" height="1248" alt="Screenshot 2026-11-15 175334" src="https://github.com/user-attachments/assets/cd372809-7116-403c-9c5e-25450bc0cb11" />

Static route dan default route di router lain (R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, R-BARAT) dihapus dengan cara yang sama. Contoh pada R-DC (`no ip route 0.0.0.0 0.0.0.0 10.32.241.33`) terlihat di bagian bawah screenshot ping L7 di atas, dan contoh pada R-UTARA terlihat di bagian bawah screenshot konfigurasi L7. Setelah itu `show running-config | include ip route` tidak menampilkan keluaran apa pun.

### 3. Konfigurasi OSPF

Contoh konfigurasi R-PUSAT:

```
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface Serial0/0/0
 no passive-interface Serial0/0/1
 no passive-interface Serial0/1/0
 no passive-interface Serial0/1/1
 no passive-interface Serial0/2/0
 network 10.32.241.32 0.0.0.31 area 0
 network 10.32.224.0 0.0.7.255 area 0
 network 10.32.241.0 0.0.0.31 area 0
 default-information originate
```

Contoh konfigurasi R-UTARA (router yang memiliki link L7):

```
interface GigabitEthernet0/2
 ip ospf network point-to-point
router ospf 1
 router-id 3.3.3.3
 passive-interface default
 no passive-interface Serial0/0/0
 no passive-interface Serial0/0/1
 no passive-interface GigabitEthernet0/2
 network 10.32.241.32 0.0.0.31 area 0
 network 10.32.0.0 0.0.63.255 area 0
 network 10.32.236.0 0.0.1.255 area 0
```

| Perintah | Fungsi |
|---|---|
| `router ospf 1` | Mengaktifkan proses OSPF dengan process ID 1 |
| `router-id` | Identitas unik router di OSPF (R-PUSAT 1.1.1.1, R-DC 2.2.2.2, R-UTARA 3.3.3.3, R-PELABUHAN 4.4.4.4, R-SELATAN 5.5.5.5, R-TIMUR 6.6.6.6, R-BARAT 7.7.7.7) |
| `passive-interface default` | Semua interface pasif, tidak mengirim Hello, sehingga LAN pengguna tidak ikut membentuk neighbor |
| `no passive-interface <WAN>` | Hanya interface link antar router yang boleh membentuk neighbor |
| `network <net> <wildcard> area 0` | Mengiklankan network ke OSPF area 0. Blok 10.32.241.32/27 mencakup semua link serial dan L7 |
| `ip ospf network point-to-point` | Pada link L7, adjacency langsung terbentuk tanpa pemilihan DR/BDR |
| `default-information originate` | R-PUSAT menyebarkan default route ke seluruh router |

Ringkasan seluruh router:

| Router | Router ID | Interface OSPF aktif | Network yang diiklankan |
|---|---|---|---|
| R-PUSAT | 1.1.1.1 | Se0/0/0, Se0/0/1, Se0/1/0, Se0/1/1, Se0/2/0 | 10.32.241.32/27, 10.32.224.0/21, 10.32.241.0/27 |
| R-DC | 2.2.2.2 | Se0/0/0, Gi0/2 | 10.32.241.32/27, 10.32.96.0/19, 10.32.239.0/24 |
| R-UTARA | 3.3.3.3 | Se0/0/0, Se0/0/1, Gi0/2 | 10.32.241.32/27, 10.32.0.0/18, 10.32.236.0/23 |
| R-PELABUHAN | 4.4.4.4 | Se0/0/0 | 10.32.241.32/27, 10.32.160.0/19, 10.32.240.0/25 |
| R-SELATAN | 5.5.5.5 | Se0/0/0 | 10.32.241.32/27, 10.32.64.0/19, 10.32.240.128/25 |
| R-TIMUR | 6.6.6.6 | Se0/0/0 | 10.32.241.32/27, 10.32.128.0/19, 10.32.232.0/22 |
| R-BARAT | 7.7.7.7 | Se0/0/0 | 10.32.241.32/27, 10.32.192.0/19, 10.32.238.0/24 |

### 4. Cara kerja failover

- Jalur utama R-PUSAT ke R-DC adalah link serial langsung (10.32.241.32/30, cost 64).
- Jalur cadangan adalah R-PUSAT ke R-UTARA (serial, cost 64), lalu R-UTARA ke R-DC lewat L7 (Gigabit, cost 1), total cost 65.
- OSPF memilih cost terkecil, jadi trafik normal lewat link utama. Saat link utama diputus, OSPF menghitung ulang (SPF) dan otomatis memakai jalur cadangan lewat L7, tanpa konfigurasi manual.

## Bukti Pengujian Mandiri

Seluruh pengujian di bawah sudah dijalankan saat demo dan semuanya berhasil. Bukti visual yang dilampirkan hanya pada bagian konfigurasi di atas. Pengujian lainnya dijelaskan hasilnya pada tabel berikut.

### a. Persiapan (sebelum failover)

| No | Pengujian | Perintah / cara | Hasil |
|---|---|---|---|
| 1 | Link L7 aktif | `ping 10.32.241.58` di R-DC | Berhasil, balasan `!` (4 dari 5, ping pertama timeout karena ARP) |
| 2 | Konfigurasi L7 di R-UTARA | `show ip interface brief` | Gi0/2 10.32.241.58, status up/up |
| 3 | Static route sudah dihapus | `show running-config \| include ip route` di tiap router | R-PUSAT hanya tersisa default route ke 203.0.113.18, router lain tidak ada keluaran |

### b. Neighbor dan rute OSPF

| No | Pengujian | Perintah | Hasil |
|---|---|---|---|
| 1 | Neighbor R-PUSAT | `show ip ospf neighbor` | 5 neighbor (2.2.2.2, 3.3.3.3, 5.5.5.5, 6.6.6.6, 7.7.7.7) berstatus FULL |
| 2 | Neighbor R-UTARA | `show ip ospf neighbor` | Neighbor R-PUSAT, R-DC (lewat L7), dan R-PELABUHAN berstatus FULL |
| 3 | Rute OSPF di routing table | `show ip route ospf` di R-PUSAT | Rute berkode `O` ke seluruh LAN wilayah |
| 4 | Konektivitas antar wilayah | `ping 10.32.239.2`, `ping 10.32.240.11`, `ping 10.32.64.11` dari PC-PEGAWAI-BALAIKOTA | Semua balasan berhasil |
| 5 | Akses internet | `ping 8.8.8.8` dari PC-CCTV-KOTA | Balasan berhasil |

### c. Uji failover

Skenario: PC-PEGAWAI-BALAIKOTA mengirim ping terus-menerus ke SRV-DC (10.32.239.2). Link utama R-PUSAT ke R-DC diputus, lalu dipulihkan.

| No | Langkah | Perintah / cara | Hasil |
|---|---|---|---|
| 1 | Traceroute sebelum putus | `tracert 10.32.239.2` di PC-PEGAWAI-BALAIKOTA | Jalur langsung: PC, R-PUSAT, R-DC, SRV-DC |
| 2 | Ping kontinu | `ping -t 10.32.239.2` | Balasan terus berjalan |
| 3 | Putuskan link utama | Hapus kabel serial R-PUSAT ke R-DC | Ping sempat `Request timed out` (RTO) |
| 4 | Pemulihan lewat L7 | Tunggu OSPF konvergen | Ping kembali `Reply` tanpa konfigurasi manual |
| 5 | Traceroute sesudah putus | `tracert 10.32.239.2` | Jalur berubah: PC, R-PUSAT, R-UTARA, R-DC, SRV-DC (lewat L7) |
| 6 | Neighbor saat link putus | `show ip ospf neighbor` di R-PUSAT | Neighbor 2.2.2.2 hilang, 4 neighbor tersisa |
| 7 | Sambungkan kembali | Pasang kabel serial R-PUSAT ke R-DC | Neighbor 2.2.2.2 kembali FULL |
| 8 | Traceroute setelah dipulihkan | `tracert 10.32.239.2` | Jalur kembali langsung lewat R-PUSAT dan R-DC |

### d. Baris tambahan tabel VLSM

| Nama Subnet | Router | Host | Prefix | Ukuran Blok | Network ID | Subnet Mask | Rentang Host | Broadcast | Gateway |
|---|---|---|---|---|---|---|---|---|---|
| L7 (R-DC - R-UTARA) | R-DC - R-UTARA | 2 | /30 | 4 | 10.32.241.56 | 255.255.255.252 | 10.32.241.57 - 10.32.241.58 | 10.32.241.59 | 10.32.241.57 |

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
