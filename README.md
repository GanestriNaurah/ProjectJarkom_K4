# ProjectJarkom_K4

| Nama | NRP |
|---|---|
|Abhista Athallah Dyfan | 5027251006|
|Ganestri Naurah Sawestri | 5027251014|
|Nur Rizki Syahbana | 5027251095|

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
| 1 | Interface router memakai IP statis | `show ip interface brief` | Kolom Method `manual`, status up/up | ![](docs/img/soal4-show-ip-interface-brief.png) |
| 2 | IP statis server | `ipconfig` di SRV-DC dan SRV-BALAIKOTA | `10.32.239.2` dan `10.32.241.2` | ![](docs/img/soal4-ipconfig-srv-dc.png) ![](docs/img/soal4-ipconfig-srv-balaikota.png) |
| 3 | Pool dan alamat yang dikecualikan | `show ip dhcp pool` dan `show running-config \| include excluded` | Pool sesuai tabel, excluded `.1 - .10` | ![](docs/img/soal4-show-ip-dhcp-pool.png) |
| 4 | Perangkat pengguna mendapat IP, gateway, dan DNS | IP Configuration, DHCP, lalu `ipconfig /all` (PC-KANTOR-PELABUHAN) | IP `10.32.240.11`, gateway `10.32.240.1`, DNS `10.32.239.2` | ![](docs/img/soal4-ipconfig-pc-kantor-pelabuhan.png) |
| 5 | Perangkat pengguna di wilayah lain | `ipconfig /all` | IP dari subnet yang sesuai, mulai `.11` | ![](docs/img/soal4-ipconfig-wilayah-lain.png) |
| 6 | Tidak bentrok dengan IP statis | `show ip dhcp binding` | Alamat yang tersewa mulai dari `.11` | ![](docs/img/soal4-show-ip-dhcp-binding.png) |
| 7 | R-DC tidak punya pool DHCP | `show running-config \| include dhcp` di R-DC | Tidak ada keluaran | ![](docs/img/soal4-r-dc-tanpa-pool.png) |
| 8 | Relay di R-DC | `show ip interface GigabitEthernet0/0` | `Helper address is 10.32.239.2` | ![](docs/img/soal4-r-dc-helper-address.png) |
| 9 | Pool CCTV di server | SRV-DC, Services, DHCP | Pool `CCTV-KOTA` terdaftar | ![](docs/img/soal4-srv-dc-dhcp-cctv.png) |
| 10 | CCTV mendapat IP dari server | `ipconfig /all` di PC-CCTV-KOTA | IP `10.32.96.11`, DHCP Servers `10.32.239.2` | ![](docs/img/soal4-ipconfig-pc-cctv.png) |

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
| 1 | Record DNS di server | SRV-DC, Services, DNS | Tiga record `www`, `ftp`, `mail` | ![](docs/img/soal8-dns-srv-dc.png) |
| 2 | Dari Pelabuhan | `nslookup www.k4.smartcity.lab`, `nslookup ftp.k4.smartcity.lab`, `nslookup mail.k4.smartcity.lab` | `10.32.241.2`, `10.32.239.2`, `10.32.241.2` | ![](docs/img/soal8-nslookup-pelabuhan.png) |
| 3 | Dari wilayah lain | perintah yang sama | hasil yang sama | ![](docs/img/soal8-nslookup-wilayah-lain.png) |
| 4 | Dari CCTV | perintah yang sama | hasil yang sama | ![](docs/img/soal8-nslookup-cctv.png) |

### b. Web

| No | Pengujian | Cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Dari Pelabuhan | Web Browser, `http://www.k4.smartcity.lab` | Halaman "Portal Smart City Surabaya" dan "Kelompok 4" | ![](docs/img/soal8-web-pelabuhan.png) |
| 2 | Dari wilayah lain | Web Browser, `http://www.k4.smartcity.lab` | Halaman yang sama | ![](docs/img/soal8-web-wilayah-lain.png) |

### c. FTP

| No | Pengujian | Perintah | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Akun dan layanan FTP | SRV-DC, Services, FTP | User `cctv` terdaftar, `cisco` tidak ada | ![](docs/img/soal8-ftp-srv-dc.png) |
| 2 | Login dan unggah dari LAN CCTV | `ftp ftp.k4.smartcity.lab`, login `cctv`, `put rekaman02.txt`, `dir` | `Transfer complete`, file muncul di `dir` | ![](docs/img/soal8-ftp-upload-cctv.png) |
| 3 | Unduh dari wilayah lain | `ftp ftp.k4.smartcity.lab`, login `cctv`, `get rekaman02.txt` | File berhasil diunduh | ![](docs/img/soal8-ftp-download.png) |

### d. Email

| No | Pengujian | Cara | Hasil yang diharapkan | Screenshot |
|---|---|---|---|---|
| 1 | Layanan dan akun email | SRV-BALAIKOTA, Services, EMAIL | Domain `k4.smartcity.lab`, dua akun terdaftar | ![](docs/img/soal8-email-server.png) |
| 2 | Kirim dari Balai Kota | Compose ke `pegawai.pel@k4.smartcity.lab` | Email terkirim | ![](docs/img/soal8-email-kirim-bk.png) |
| 3 | Diterima di Pelabuhan | Receive di PC-KANTOR-PELABUHAN | Email muncul | ![](docs/img/soal8-email-terima-pel.png) |
| 4 | Balasan dari Pelabuhan | Compose ke `pegawai.bk@k4.smartcity.lab`, lalu Receive di Balai Kota | Balasan muncul | ![](docs/img/soal8-email-balasan.png) |

