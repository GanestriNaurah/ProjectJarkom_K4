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

