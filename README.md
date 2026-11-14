# JARKOM-ETS

## Member

| Nama                          | NRP        | Peran & Pembagian Kerja                                                                 |
| ----------------------------- | ---------- | --------------------------------------------------------------------------------------- |
| Yovi Prayudya Rizky Ramadhani | 5027251107 | **A - Arsitek Dasar & Pengalamatan**: Excel VLSM/CIDR + tabel revisi, topologi CPT, IP dasar |
| Askhabul                      | 5027251049 | **B - Spesialis Rute & Keamanan**: static routing, NAT, GRE, OSPF + link cadangan L7     |
| Keisya                        | 5027251068 | **C - SysAdmin & Dokumentasi**: DHCP (PC & CCTV), layanan aplikasi, README, bukti, laporan |

## 1. Topologi dan Pengalamatan

![image](./assets/soal/soal-1.png)

Kurang lebih topologinya seperti berikut:

![image](./assets/jawaban/topologi-soal-1-1.png)
![image](./assets/jawaban/topologi-soal-1-2.png)

Kelompok kami memilih skema CIDR karena tiga alasan. Pertama, setiap router mendapat satu blok besar (summary), jadi tabel rute di R-PUSAT cukup satu entri per router, bukan satu entri per LAN. Kedua, blok besar itu masih bisa dipecah menjadi subnet /30 untuk link antar-router, sehingga penambahan link seperti L7 pada soal 7 tidak perlu menggeser alamat LAN yang sudah dipakai. Ketiga, batas blok per router memudahkan pembagian kerja dan pengecekan siapa mengerjakan bagian mana. Rincian blok ada di sheet `Revisi CIDR`, sedangkan tabel VLSM dipakai sebagai pembanding jawaban UTS.

R-ISP
![image](./assets/jawaban/config/R-ISP-config.png)

Router diberi nama `R-ISP` lewat `hostname`, lalu tiap interface diberi alamat IP beserta subnet mask-nya dan diaktifkan dengan `no shutdown`:
 
- `Gi0/0` dengan `203.0.113.34/30` ke R-PUSAT.
- `Gi0/1` dengan `8.8.8.1/24` ke Server 8.8.8.8, yang menjadi gateway server tersebut.
Konfigurasi disimpan dengan `write memory`. R-ISP tidak diberi route ke alamat privat internal, sesuai ketentuan Soal 5.

R-PUSAT

![image](./assets/jawaban/config/R-PUSAT-config.png)

Nah disini sama seperti R-ISP untuk command ini dan menyimpannya melalui `write memory`. Setelah itu

![image](./assets/jawaban/config/R-PUSAT-config-1.png)
![image](./assets/jawaban/config/R-PUSAT-config-2.png)

Konfigurasi R-PUSAT terdiri dari tiga bagian:
 
1. **Interface routed**: `Gi0/0` (`203.0.113.33/30`) ke R-ISP dan `Gi0/1` (`10.65.208.5/30`) ke R-UTARA.
2. **Sub-interface untuk SW-BALAIKOTA**: `Gi0/2.10` (`10.65.192.1/21`, Pegawai Balai Kota) dan `Gi0/2.20` (`10.65.200.1/27`, Server Balai Kota), masing-masing dengan `encapsulation dot1Q` sesuai nomor VLAN.
3. **Link L2 sampai L5 lewat modul HWIC-4ESW**: tiap port dijadikan access port di VLAN 12 sampai 15, lalu alamat link /30 dipasang di `interface vlan 12` sampai `interface vlan 15`.


Untuk sisanya:

R-SELATAN

![image](./assets/jawaban/config/R-SELATAN-config.png)

R-TIMUR

![image](./assets/jawaban/config/R-TIMUR-config.png)

R-BARAT

![image](./assets/jawaban/config/R-BARAT-config.png)

R-UTARA

![image](./assets/jawaban/config/R-UTARA-config.png)

R-PELABUHAN

![image](./assets/jawaban/config/R-PELABUHAN-config.png)

R-DC

![image](./assets/jawaban/config/R-DC-config.png)

sesuai dengan konfigurasi pada R-PUSAT tersebut untuk terhubung dengan ip address yang sudah di set beserta subnet masknya. Nah karena antar router sudah di set dengan konfigurasi tersebut maka lanjut ke switch agar bisa terhubung menuju ke router sebagai gatewaynya. Seperti berikut:

![image](./assets/jawaban/config/SW-BALAIKOTA-config.png)

Namun disini ada kerusakan yang dimana ketika `show interfaces trunk` dijalankan, tidak muncul output yang kami inginkan. Maka dari itu perbaikannya sebagai berikut

![image](./assets/jawaban/config/SW-BALAIKOTA-config-fix-broken.png)

dengan melihat status pada interfacesnya tersebut lalu mendaftarkan ke `configure terminal` setelah itu hasilnya sebagai berikut:

![image](./assets/jawaban/config/SW-BALAIKOTA-result.png)

lalu lanjut ke server PT balaikota, config:

![image](./assets/jawaban/config/server-balaikota.png)

dengan configurasi:
- IPv4 Address: 10.65.200.2
- Subnet Mask: 255.255.255.224
- Default Gateway: 10.65.200.1

dan di ping...

![image](./assets/jawaban/error/ping-error-server-balaikota.png)

Dan ternyata kesalahannya di:

![image](./assets/jawaban/error/error-location-server-balaikota.png)

pada Port Fa0/1 terakhir belum diforward maka dari itu ditambahkan sebagai berikut:

```
SW-BALAIKOTA(config)#interface fa0/1
SW-BALAIKOTA(config-if)# switchport mode trunk

SW-BALAIKOTA(config-if)#interface fa0/2
SW-BALAIKOTA(config-if)# switchport mode access
SW-BALAIKOTA(config-if)# switchport access vlan 10
SW-BALAIKOTA(config-if)#interface fa0/3
SW-BALAIKOTA(config-if)# switchport mode access
SW-BALAIKOTA(config-if)# switchport access vlan 20
```

Setelah di `write memory` tersebut maka bisa send ping requestnya ini

![image](./assets/jawaban/error/fix-error-server-balaikota.png)

Sisanya juga sama

DC

![image](./assets/jawaban/config/R-DC-config-lanjutan.png)


```
configure terminal
no ip domain-lookup
vlan 10
 name CCTV
vlan 20
 name SERVER
interface fa0/3
 switchport mode trunk
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
end
write memory
```

Ini akan menuliskannya port dari 2 perangkat ini ke vlannya untuk menghindari kesalahan memasukkan port yang salah. Maka dari itu penting untuk dikelompokkan. Hasilnya sebagai berikut

![image](./assets/jawaban/config/sw-dc-config.png)

Dan juga menyetting ip address, subnet mask, dan ip gateway pada `server dan storage DC`:

![image](./assets/jawaban/config/setup-config-server-dc.png)

hasilnya sebagai berikut:

![image](./assets/jawaban/sccuess-ping-r-dc.png)

BARAT

kurang lebih juga sama:

![image](./assets/jawaban/config/R-BARAT-config-lanjutan.png)

![image](./assets/jawaban/config/R-BARAT-ping-success.png)

![image](./assets/jawaban/config/SW-BARAT-config.png)

TIMUR

![image](./assets/jawaban/config/R-TIMUR-lanjutan.png)

![image](./assets/jawaban/config/SW-TIMUR.png)

kurang spasi di Fa0/3nya. Jadinya cara benerinnya:

```
configure terminal
interface fa0/2
 switchport mode access
 switchport access vlan 10
interface fa0/3
 switchport mode access
 switchport access vlan 20
end
write memory
```

SELATAN

![image](./assets/jawaban/config/R-SELATAN-config-lanjutan.png)

salah ping harusnya `10.65.208.9` bukan `10.65.208.13`

![image](./assets/jawaban/config/SW-SELATAN-config.png)

UTARA

![image](./assets/jawaban/config/R-UTARA-config-lanjutan.png)

![image](./assets/jawaban/config/sw-utara-config-1.png)
![image](./assets/jawaban/config/sw-utara-config-2.png)

PELABUHAN

![image](./assets/jawaban/config/R-PELABUHAN-config-lanjutan.png)

![image](./assets/jawaban/config/R-PELABUHAN-ping.png)

![image](./assets/jawaban/config/sw-pelabuhan-config.png)

sisanya:

```
SW-PELABUHAN#show interface trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,20

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,20
```

```
SW-PELABUHAN#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   SCADA                            active    Fa0/2
20   KANTOR                           active    Fa0/3
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active  
```

Lalu perbaikan topologinya:

![image](./assets/jawaban/perbaikan-topologi.png)

### Desain tiap lokasi
 
Hubungan antar-router (L1 sampai L6, R-ISP, dan Server 8.8.8.8) mengikuti Gambar 1 tanpa perubahan. Kabel antar-router memakai Copper Cross-Over, sedangkan router ke switch dan switch ke perangkat memakai Copper Straight-Through. Tunnel GRE R-PUSAT ke R-PELABUHAN (Soal 6) bukan kabel fisik, jadi tidak digambar sebagai kabel. Jalur fisik R-PELABUHAN tetap lewat R-UTARA (L6).
 
Setiap lokasi punya dua LAN, sedangkan router 2911 hanya punya tiga port Gigabit. Karena itu semua lokasi dirancang dengan pola yang sama:
 
- **Satu router 2911** sebagai gateway.
- **Satu switch 2960-24TT** di bawah router. Satu kabel dari router ke switch membawa dua VLAN (trunk): VLAN 10 untuk LAN pertama dan VLAN 20 untuk LAN kedua. Switch 2960 dipilih dibanding Switch-PT karena VLAN dan trunk bisa dikonfigurasi lewat CLI.
- **Router-on-a-stick**: router memakai dua sub-interface (`.10` dan `.20`) dengan `encapsulation dot1Q`, sehingga satu port router cukup untuk dua subnet dan port lainnya tersisa untuk link backbone.
- **Satu perangkat perwakilan per LAN**, sesuai ketentuan soal.
| Lokasi | Router | Switch | LAN 1 (VLAN 10) | LAN 2 (VLAN 20) |
|---|---|---|---|---|
| Balai Kota | R-PUSAT | SW-BALAIKOTA | PC-PEGAWAI-BALAIKOTA (PC) | SRV-BALAIKOTA (server) |
| Command Center | R-DC | SW-DC | PC-CCTV-KOTA (webcam) | SRV-STORAGE-DC (server) |
| Surabaya Utara | R-UTARA | SW-UTARA | AP-WIFI-UTARA (access point) | PC-SENSOR-LALINTAS-UTARA (PC) |
| Pelabuhan | R-PELABUHAN | SW-PELABUHAN | PC-SCADA-PELABUHAN (PC) | PC-KANTOR-PELABUHAN (PC) |
| Surabaya Selatan | R-SELATAN | SW-SELATAN | AP-WIFI-SELATAN (access point) | PC-PJU-SELATAN (PC) |
| Surabaya Timur | R-TIMUR | SW-TIMUR | AP-WIFI-TIMUR (access point) | PC-SENSOR-BANJIR-TIMUR (PC) |
| Surabaya Barat | R-BARAT | SW-BARAT | AP-WIFI-BARAT (access point) | PC-PUSKESMAS-SEKOLAH-BARAT (PC) |
 
### Alasan pemilihan perangkat dan media
 
- **WiFi Publik** memakai access point karena pengguna terhubung secara nirkabel. Access point tersambung ke switch dengan kabel UTP.
- **Server** (Balai Kota dan Storage DC) memakai Server-PT karena nanti menjalankan layanan (DNS, web, FTP, email pada Soal 8) dan memakai IP statis.
- **CCTV** memakai webcam karena fungsinya perangkat penangkap gambar.
- **Sensor, PJU, kantor, dan pegawai** diwakili PC, karena cukup untuk menguji konektivitas dan mendapat IP dinamis.
- **Kabel UTP** dipilih untuk semua link karena jaraknya pendek dan biayanya lebih rendah. Di lapangan, link antar-lokasi (L1 sampai L6) lebih sesuai memakai fiber, tetapi di Packet Tracer tetap digambar dengan kabel tembaga.
### Penambahan port pada R-PUSAT
 
R-PUSAT butuh tujuh koneksi (R-ISP, L1 sampai L5, dan SW-BALAIKOTA), sedangkan 2911 hanya punya tiga port Gigabit bawaan. Karena itu ditambahkan satu modul **HWIC-4ESW** (empat port) pada slot HWIC 0.
 
Modul Ethernet routed berbasis tembaga tidak ada di daftar modul versi Packet Tracer yang dipakai, jadi port modul ini berupa port switch. Tiap port dimasukkan ke VLAN tersendiri, dan alamat link dipasang di interface VLAN-nya.
 
| Port R-PUSAT | Terhubung ke | Link | Interface |
|---|---|---|---|
| Gi0/0 | R-ISP | - | 203.0.113.33/30 |
| Gi0/1 | R-UTARA | L2 | 10.65.208.5/30 |
| Gi0/2 | SW-BALAIKOTA | - | sub-interface `.10` dan `.20` |
| Fa0/0/0 | R-BARAT | L5 | VLAN 15, 10.65.208.17/30 |
| Fa0/0/1 | R-DC | L1 | VLAN 12, 10.65.208.1/30 |
| Fa0/0/2 | R-SELATAN | L3 | VLAN 13, 10.65.208.9/30 |
| Fa0/0/3 | R-TIMUR | L4 | VLAN 14, 10.65.208.13/30 |
 
### Pemetaan port
 
Nomor port tidak seragam di semua lokasi, jadi dicatat di sini sesuai kabel yang terpasang.
 
| Router | Port ke router lain | Port ke switch |
|---|---|---|
| R-ISP | Gi0/0 ke R-PUSAT | Gi0/1 ke Server 8.8.8.8 |
| R-DC | Gi0/0 ke R-PUSAT (L1) | Gi0/1 ke SW-DC |
| R-UTARA | Gi0/0 ke R-PUSAT (L2), Gi0/2 ke R-PELABUHAN (L6) | Gi0/1 ke SW-UTARA |
| R-PELABUHAN | Gi0/0 ke R-UTARA (L6) | Gi0/1 ke SW-PELABUHAN |
| R-SELATAN | Gi0/0 ke R-PUSAT (L3) | Gi0/1 ke SW-SELATAN |
| R-TIMUR | Gi0/0 ke R-PUSAT (L4) | Gi0/1 ke SW-TIMUR |
| R-BARAT | Gi0/1 ke R-PUSAT (L5) | Gi0/0 ke SW-BARAT |
 
Port uplink (trunk) ke router tiap switch: SW-BALAIKOTA `Fa0/3`, SW-DC `Fa0/3`, dan SW-UTARA, SW-PELABUHAN, SW-SELATAN, SW-TIMUR, SW-BARAT di `Fa0/1`. Port lain adalah access port ke VLAN 10 dan VLAN 20.
 
## 2. Tabel VLSM, CIDR, dan Revisi

![image](./assets/soal/soal-2.png)

Tabel lengkap ada di spreadsheet berikut. Kolom alasan revisi menjelaskan perbedaan dengan jawaban UTS.

[Revisi Tabel Jarkom ETS.xlsx](./Revisi%20Tabel%20Jarkom%20ETS.xlsx)

## 3. Static Routing

### 3.1 Ketentuan & Desain Perutean Hierarkis
Pada perancangan static routing ini, seluruh LAN internal harus dapat saling berkomunikasi secara *end-to-end* dengan mematuhi ketentuan hierarki perutean:
1. **Default Route Arah Keluar (Upstream)**:
   - Default route (`0.0.0.0/0`) hanya diperbolehkan mengarah ke router di atasnya.
   - Router cabang stub (R-DC, R-SELATAN, R-TIMUR, R-BARAT) hanya mengarahkan default route ke gateway R-PUSAT.
   - R-PELABUHAN mengarahkan default route ke R-UTARA (`10.65.208.21`).
   - R-UTARA mengarahkan default route ke R-PUSAT (`10.65.208.5`).
   - R-PUSAT mengarahkan default route ke R-ISP (`203.0.113.34`).
2. **Static Route Arah Dalam (Downstream)**:
   - R-PUSAT wajib memiliki static route spesifik ke seluruh LAN milik router di bawahnya (R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, dan R-BARAT).
   - R-UTARA wajib memiliki static route spesifik ke seluruh LAN milik R-PELABUHAN (`10.64.192.0/19` dan `10.64.224.0/25`).

### 3.2 Tabel Konfigurasi Static Route

Berikut rincian tabel konfigurasi rute statis pada setiap router:

| Router | Network Tujuan | Netmask | Next-Hop | Keterangan |
|---|---|---|---|---|
| **R-PUSAT** | `0.0.0.0` | `0.0.0.0` | `203.0.113.34` | Default route keluar ke R-ISP |
| | `10.64.128.0` | `255.255.224.0` | `10.65.208.2` | CCTV Kota (R-DC) |
| | `10.64.160.0` | `255.255.255.0` | `10.65.208.2` | Server & Storage DC (R-DC) |
| | `10.64.0.0` | `255.255.192.0` | `10.65.208.6` | WiFi Publik Utara (R-UTARA) |
| | `10.64.64.0` | `255.255.254.0` | `10.65.208.6` | Sensor Lalu Lintas (R-UTARA) |
| | `10.65.208.20` | `255.255.255.252` | `10.65.208.6` | Interkoneksi L6 (R-UTARA <-> R-PELABUHAN) |
| | `10.64.192.0` | `255.255.224.0` | `192.168.8.2` | SCADA Pelabuhan melalui Tunnel0 |
| | `10.64.224.0` | `255.255.255.128` | `192.168.8.2` | Kantor Pelabuhan melalui Tunnel0 |
| | `10.65.0.0` | `255.255.224.0` | `10.65.208.10` | WiFi Publik Selatan (R-SELATAN) |
| | `10.65.32.0` | `255.255.255.128` | `10.65.208.10` | PJU Pintar Selatan (R-SELATAN) |
| | `10.65.64.0` | `255.255.224.0` | `10.65.208.14` | WiFi Publik Timur (R-TIMUR) |
| | `10.65.96.0` | `255.255.252.0` | `10.65.208.14` | Sensor Banjir Timur (R-TIMUR) |
| | `10.65.128.0` | `255.255.224.0` | `10.65.208.18` | WiFi Publik Barat (R-BARAT) |
| | `10.65.160.0` | `255.255.255.0` | `10.65.208.18` | Puskesmas & Sekolah (R-BARAT) |
| **R-DC** | `0.0.0.0` | `0.0.0.0` | `10.65.208.1` | Default route upstream ke R-PUSAT |
| **R-UTARA** | `0.0.0.0` | `0.0.0.0` | `10.65.208.5` | Default route upstream ke R-PUSAT |
| | `10.64.192.0` | `255.255.224.0` | `10.65.208.22` | Sensor & SCADA (R-PELABUHAN) |
| | `10.64.224.0` | `255.255.255.128` | `10.65.208.22` | Kantor Pelabuhan (R-PELABUHAN) |
| **R-PELABUHAN** | `0.0.0.0` | `0.0.0.0` | `10.65.208.21` | Default route upstream ke R-UTARA |
| **R-SELATAN** | `0.0.0.0` | `0.0.0.0` | `10.65.208.9` | Default route upstream ke R-PUSAT |
| **R-TIMUR** | `0.0.0.0` | `0.0.0.0` | `10.65.208.13` | Default route upstream ke R-PUSAT |
| **R-BARAT** | `0.0.0.0` | `0.0.0.0` | `10.65.208.17` | Default route upstream ke R-PUSAT |

### 3.3 Verifikasi Konektivitas Antar-LAN
Uji konektivitas dilakukan menggunakan pengujian `ping` antar-perangkat di berbagai LAN yang berbeda:
- **PC-PEGAWAI-BALAIKOTA** (`10.65.192.3`) -> **SRV-STORAGE-DC** (`10.64.160.2`): Sukses (*Packet Loss 0%*).
- **PC-PEGAWAI-BALAIKOTA** (`10.65.192.3`) -> **PC-KANTOR-PELABUHAN** (`10.64.224.3`): Sukses (*Packet Loss 0%*).
- **SRV-STORAGE-DC** (`10.64.160.2`) -> **PC-PUSKESMAS-SEKOLAH-BARAT** (`10.65.160.3`): Sukses (*Packet Loss 0%*).
- **PC-PJU-SELATAN** (`10.65.32.3`) -> **PC-SENSOR-BANJIR-TIMUR** (`10.65.96.3`): Sukses (*Packet Loss 0%*).

Dengan konfigurasi ini, seluruh segmen jaringan privat kota terhubung penuh tanpa memerlukan dynamic routing protocol pada file utama ini.

Bukti tabel static route pada R-PUSAT:

![Static route R-PUSAT](./assets/bukti/Keisya/03-static-route-r-pusat.png)

---

## 4. Pengalamatan IP dan DHCP

![image](./assets/soal/soal-4.png)

### 4.1 Interface router dan server memakai IP statis
 
Seluruh interface router dikonfigurasi dengan IP statis lewat CLI (`ip address` pada interface atau sub-interface). Seluruh server dikonfigurasi statis lewat tab **Desktop, IP Configuration**, dengan pilihan **Static**. Perangkat pengguna (PC dan webcam) mendapat IP dinamis dari DHCP. AP-PT hanya berfungsi sebagai bridge Layer 2, sehingga tidak memerlukan lease klien.
 
### Alamat interface router
 
| Router | Interface | Alamat | Tujuan |
|---|---|---|---|
| R-ISP | Gi0/0 | 203.0.113.34/30 | R-PUSAT |
| | Gi0/1 | 8.8.8.1/24 | Server 8.8.8.8 |
| R-PUSAT | Gi0/0 | 203.0.113.33/30 | R-ISP |
| | Gi0/1 | 10.65.208.5/30 | L2 |
| | Gi0/2.10 | 10.65.192.1/21 | Pegawai Balai Kota |
| | Gi0/2.20 | 10.65.200.1/27 | Server Balai Kota |
| | Vlan12 | 10.65.208.1/30 | L1 |
| | Vlan13 | 10.65.208.9/30 | L3 |
| | Vlan14 | 10.65.208.13/30 | L4 |
| | Vlan15 | 10.65.208.17/30 | L5 |
| R-DC | Gi0/0 | 10.65.208.2/30 | L1 |
| | Gi0/1.10 | 10.64.128.1/19 | CCTV Kota |
| | Gi0/1.20 | 10.64.160.1/24 | Server dan Storage DC |
| R-UTARA | Gi0/0 | 10.65.208.6/30 | L2 |
| | Gi0/1.10 | 10.64.0.1/18 | WiFi Publik Utara |
| | Gi0/1.20 | 10.64.64.1/23 | Sensor Lalu Lintas Utara |
| | Gi0/2 | 10.65.208.21/30 | L6 |
| R-PELABUHAN | Gi0/0 | 10.65.208.22/30 | L6 |
| | Gi0/1.10 | 10.64.192.1/19 | Sensor dan SCADA |
| | Gi0/1.20 | 10.64.224.1/25 | Kantor Pelabuhan |
| R-SELATAN | Gi0/0 | 10.65.208.10/30 | L3 |
| | Gi0/1.10 | 10.65.0.1/19 | WiFi Publik Selatan |
| | Gi0/1.20 | 10.65.32.1/25 | PJU Pintar Selatan |
| R-TIMUR | Gi0/0 | 10.65.208.14/30 | L4 |
| | Gi0/1.10 | 10.65.64.1/19 | WiFi Publik Timur |
| | Gi0/1.20 | 10.65.96.1/22 | Sensor Banjir Timur |
| R-BARAT | Gi0/1 | 10.65.208.18/30 | L5 |
| | Gi0/0.10 | 10.65.128.1/19 | WiFi Publik Barat |
| | Gi0/0.20 | 10.65.160.1/24 | Puskesmas dan Sekolah Barat |
 
### Alamat server
 
| Server | Alamat | Mask | Gateway |
|---|---|---|---|
| Server 8.8.8.8 | 8.8.8.8 | 255.255.255.0 | 8.8.8.1 |
| SRV-BALAIKOTA | 10.65.200.2 | 255.255.255.224 | 10.65.200.1 |
| SRV-STORAGE-DC | 10.64.160.2 | 255.255.255.0 | 10.64.160.1 |
 
Alamat `.2` dipakai untuk server, tepat setelah gateway `.1`. Jadi rentang DHCP pengguna di subnet yang sama harus dimulai setelah alamat tersebut supaya tidak bentrok.
 
### Bukti konektivitas
 
Link antar-tetangga diuji dengan `ping` dan `show ip interface brief`, dengan hasil:
 
- R-UTARA dan R-PUSAT saling ping berhasil (5/5), jadi L2 aktif.
- R-PELABUHAN ke R-UTARA berhasil (5/5), jadi L6 aktif.
- R-DC, R-BARAT, R-TIMUR, dan R-SELATAN masing-masing berhasil ping ke R-PUSAT.
- R-ISP ke Server 8.8.8.8 berhasil. Balasan pertama biasanya hilang karena proses ARP.
Ping dari LAN di balik router ke jaringan lain (misalnya server DC ke R-PUSAT) belum berhasil pada tahap ini karena static route belum dipasang. Itu bagian Soal 3.


### 4.2 DHCP untuk seluruh LAN pengguna

DHCP dijalankan pada router gateway masing-masing LAN. Alamat gateway `.1` dan alamat statis server `.2` dikecualikan agar tidak terjadi konflik. Semua pool membagikan DNS `10.65.200.2`.

| LAN pengguna | Network | Gateway | Lease perangkat uji |
|---|---|---|---|
| Pegawai Balai Kota | `10.65.192.0/21` | `10.65.192.1` | `10.65.192.3` |
| Sensor Lalu Lintas Utara | `10.64.64.0/23` | `10.64.64.1` | `10.64.64.3` |
| SCADA Pelabuhan | `10.64.192.0/19` | `10.64.192.1` | `10.64.192.3` |
| Kantor Pelabuhan | `10.64.224.0/25` | `10.64.224.1` | `10.64.224.3` |
| PJU Pintar Selatan | `10.65.32.0/25` | `10.65.32.1` | `10.65.32.3` |
| Sensor Banjir Timur | `10.65.96.0/22` | `10.65.96.1` | `10.65.96.3` |
| Puskesmas/Sekolah Barat | `10.65.160.0/24` | `10.65.160.1` | `10.65.160.3` |

Khusus **CCTV Kota**, DHCP dipusatkan pada `SRV-STORAGE-DC` (`10.64.160.2`) dengan pool `10.64.128.0/19`. Karena server berada di subnet berbeda, sub-interface `R-DC Gi0/1.10` memakai `ip helper-address 10.64.160.2`. Webcam memperoleh `10.64.128.3/19`, gateway `10.64.128.1`, dan DNS `10.65.200.2` pada interface kabel `GigabitEthernet3`.

### 4.3 Verifikasi DHCP

- Seluruh PC memperoleh alamat, mask, gateway, dan DNS sesuai pool.
- PJU Selatan telah dikoreksi menjadi mask `/25` (`255.255.255.128`), bukan `/20`.
- Ping PJU Selatan (`10.65.32.3`) ke Sensor Banjir Timur (`10.65.96.3`) berhasil dengan **0% packet loss**.
- Bukti pengujian CIDR tersimpan di `assets/bukti/CIDR`.

---

## 5. Network Address Translation (NAT)

### 5.1 Ketentuan & Desain NAT Overload (PAT)
Ketentuan Soal 5 mengharuskan:
1. Seluruh host di jaringan internal kota dapat mengakses internet publik (Server `8.8.8.8`) menggunakan satu alamat IP publik milik R-PUSAT (`203.0.113.33`).
2. Router ISP (`Router-ISP`) **dilarang keras** memiliki rute ke alamat IP privat internal (`10.64.0.0/15`). ISP hanya mengenal jaringan publik (`203.0.113.32/30` dan `8.8.8.0/24`).

Untuk memenuhi kebutuhan ini, diimplementasikan **NAT Overload** (Port Address Translation / PAT) pada router gateway perbatasan (**R-PUSAT**).

### 5.2 Konfigurasi NAT pada R-PUSAT
Langkah konfigurasi pada R-PUSAT:
1. Membuat Standard Access List (`access-list 1`) untuk mencakup seluruh blok IP privat kelompok 8 (`10.64.0.0/15`):
   ```cisco
   access-list 1 permit 10.64.0.0 0.1.255.255
   ```
2. Menentukan antarmuka luar (*outside interface*) yang mengarah ke ISP:
   ```cisco
   interface GigabitEthernet0/0
    ip nat outside
   ```
3. Menentukan seluruh antarmuka dalam (*inside interfaces*) yang terhubung ke jaringan internal:
   ```cisco
   interface GigabitEthernet0/1
    ip nat inside
   interface GigabitEthernet0/2.10
    ip nat inside
   interface GigabitEthernet0/2.20
    ip nat inside
   interface Vlan12
    ip nat inside
   interface Vlan13
    ip nat inside
   interface Vlan14
    ip nat inside
   interface Vlan15
    ip nat inside
   ```
4. Mengaktifkan translasi NAT Overload menggunakan IP interface publik `GigabitEthernet0/0`:
   ```cisco
   ip nat inside source list 1 interface GigabitEthernet0/0 overload
   ```

### 5.3 Pembuktian & Hasil Pengujian
#### 1. Tabel Rute Router-ISP (Bebas dari Rute Privat)
Pada `Router-ISP`, verifikasi routing table membuktikan tidak ada rute privat:
```cisco
Router-ISP#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       ...
Gateway of last resort is not set

     8.0.0.0/24 is subnetted, 1 subnets
C       8.8.8.0 is directly connected, GigabitEthernet0/1
L       8.8.8.1/32 is directly connected, GigabitEthernet0/1
     203.0.113.32/30 is subnetted, 1 subnets
C       203.0.113.32 is directly connected, GigabitEthernet0/0
L       203.0.113.34/32 is directly connected, GigabitEthernet0/0
```
*Tabel di atas membuktikan Router-ISP hanya mengetahui jaringan publik miliknya dan antarmuka interkoneksi ke R-PUSAT.*

#### 2. Uji Ping dari Seluruh Subnet ke Server Publik 8.8.8.8
- `PC-PEGAWAI-BALAIKOTA` (`10.65.192.3`) -> `ping 8.8.8.8`: **Success (Loss 0%)**
- `SRV-STORAGE-DC` (`10.64.160.2`) -> `ping 8.8.8.8`: **Success (Loss 0%)**
- `PC-KANTOR-PELABUHAN` (`10.64.224.3`) -> `ping 8.8.8.8`: **Success (Loss 0%)**
- `PC-PUSKESMAS-SEKOLAH-BARAT` (`10.65.160.3`) -> `ping 8.8.8.8`: **Success (Loss 0%)**

#### 3. Bukti Translasi Alamat (NAT Table Translation)
Pengecekan translasi aktif pada R-PUSAT saat host internal melakukan ping/akses ke internet:
```cisco
R-PUSAT#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.33:1     10.65.192.3:1      8.8.8.8:1          8.8.8.8:1
icmp 203.0.113.33:2     10.64.160.2:2      8.8.8.8:2          8.8.8.8:2
icmp 203.0.113.33:3     10.64.224.3:3      8.8.8.8:3          8.8.8.8:3
```
Seluruh alamat IP privat berhasil ditranslasikan ke alamat publik tunggal `203.0.113.33` dengan pemetaan nomor port unik (PAT).

Bukti dari file Packet Tracer yang dipakai:

![Routing table Router-ISP](./assets/bukti/Keisya/05-rute-router-isp.png)

Tabel Router-ISP hanya berisi jaringan `8.8.8.0/24` dan `203.0.113.32/30`. Setelah rute privat dihapus, ping dari PC Balai Kota ke `8.8.8.8` tetap berhasil 4/4.

![Ping PC Balai Kota ke internet](./assets/bukti/Keisya/05-ping-internet.png)

Konfigurasi NAT outside, NAT inside, dan perintah overload pada R-PUSAT:

![Konfigurasi NAT PAT R-PUSAT](./assets/bukti/Keisya/05-nat-pat-r-pusat.png)

---

## 6. Tunnel GRE

### 6.1 Ketentuan & Desain Terowongan Virtual
Ketentuan Soal 6 mengharuskan:
1. Membangun tunnel GRE point-to-point antara **R-PUSAT** dan **R-PELABUHAN**.
2. Alokasi IP Tunnel: Subnet `192.168.K.0/30` -> untuk Kelompok 8 adalah `192.168.8.0/30`:
   - R-PUSAT: `192.168.8.1/30`
   - R-PELABUHAN: `192.168.8.2/30`
3. Seluruh lalu lintas data antara subnet **Balai Kota** (`10.65.192.0/21`, `10.65.200.0/27`) dan subnet **Pelabuhan** (`10.64.192.0/19`, `10.64.224.0/25`) wajib melewati tunnel ini, melompati router perantara (R-UTARA).

### 6.2 Konfigurasi Tunnel GRE
#### 1. Konfigurasi pada R-PUSAT
```cisco
interface Tunnel0
 ip address 192.168.8.1 255.255.255.252
 tunnel source GigabitEthernet0/1
 tunnel destination 10.65.208.22
 exit

! Pengalihan rute subnet Pelabuhan agar melalui jalur tunnel
ip route 10.64.192.0 255.255.224.0 192.168.8.2
ip route 10.64.224.0 255.255.255.128 192.168.8.2
```

#### 2. Konfigurasi pada R-PELABUHAN
```cisco
interface Tunnel0
 ip address 192.168.8.2 255.255.255.252
 tunnel source GigabitEthernet0/0
 tunnel destination 10.65.208.5
 exit

! Pengalihan rute subnet Balai Kota agar melalui jalur tunnel
ip route 10.65.192.0 255.255.248.0 192.168.8.1
ip route 10.65.200.0 255.255.255.224 192.168.8.1
```

### 6.3 Verifikasi Jalur Tunnel (Traceroute)
Hasil pengujian `tracert` dari `PC-PEGAWAI-BALAIKOTA` (`10.65.192.3`) menuju `PC-KANTOR-PELABUHAN` (`10.64.224.3`):
```text
PC>tracert 10.64.224.3
Tracing route to 10.64.224.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      10.65.192.1
  2   0 ms      0 ms      0 ms      192.168.8.2
  3   0 ms      0 ms      0 ms      10.64.224.3

Trace complete.
```
**Analisis Hasil Traceroute**:
- **Hop 1** (`10.65.192.1`): Default gateway di R-PUSAT (sub-interface `Gi0/2.10`).
- **Hop 2** (`192.168.8.2`): Ujung tunnel GRE di R-PELABUHAN. Router perantara transit (**R-UTARA**) tidak muncul dalam daftar hop karena paket privat dienkapsulasi ke dalam payload paket GRE dan secara logis diteruskan langsung antar-ujung tunnel (*virtual point-to-point link*).
- **Hop 3** (`10.64.224.3`): Host tujuan di Kantor Pelabuhan.

Status `Tunnel0` pada R-PUSAT juga sudah `up/up` dengan alamat `192.168.8.1/30`:

![Status Tunnel0 R-PUSAT](./assets/bukti/Keisya/06-tunnel-gre-r-pusat.png)

### 6.4 Analisis & Penjelasan Teori

#### 1. Mengapa router memilih jalur tunnel dibandingkan jalur fisik ke pelabuhan?
Router menentukan pemilihan jalur berdasarkan hierarki aturan pemilihan rute (*route selection hierarchy*):
1. **Longest Prefix Match (LPM)**:
   Pada tabel perutean R-PUSAT dan R-PELABUHAN, rute statis diarahkan secara eksplisit menuju subnet tujuan melalui alamat IP antarmuka tunnel (`192.168.8.2` dan `192.168.8.1`). Meskipun router memiliki default route atau rute underlay lain, aturan *longest prefix match* menyatakan bahwa router akan selalu mendahulukan entri rute yang paling spesifik (memiliki panjang netmask terpanjang).
2. **Keterikatan Next-Hop Langsung (Virtual Direct Adjacency)**:
   Secara arsitektur Layer 3, antarmuka `Tunnel0` diperlakukan sebagai antarmuka point-to-point langsung dengan router tujuan. Paket yang dialokasikan ke next-hop `192.168.8.2` langsung diarahkan ke proses enkapsulasi GRE, di mana paket asli (inner IP) dibungkus oleh header baru (outer IP: Source `10.65.208.5`, Destination `10.65.208.22`), lalu dikirimkan melintasi jaringan underlay.

#### 2. Mengapa GRE saja tidak mengenkripsi data?
- **GRE adalah Protokol Enkapsulasi Murni, Bukan Enkripsi**:
  GRE (*Generic Routing Encapsulation* - RFC 2784, IP Protocol 47) dikembangkan oleh Cisco dengan tujuan utama membungkus (*encapsulate*) berbagai jenis paket protokol jaringan Layer 3 di dalam header IP baru agar dapat melintasi jaringan transit IP.
- **Ketiadaan Fitur Kriptografi Bawaan**:
  Spesifikasi protokol GRE tidak mendefinisikan mekanisme keamanan apa pun:
  1. *No Key Exchange Mechanism*: Tidak ada protokol pertukaran kunci (seperti Diffie-Hellman atau IKE).
  2. *No Confidentiality/Cipher*: Tidak ada algoritma enkripsi simetris (seperti AES, 3DES, atau ChaCha20). Payload data di dalam GRE tetap berada dalam format teks biasa (*plaintext*).
  3. *No Integrity & Authentication*: Tidak ada tanda tangan kriptografis atau hash (HMAC-SHA) untuk memverifikasi keaslian dan integritas paket.
- Konsekuensinya, jika lalu lintas GRE disadap (*packet sniffing*) menggunakan packet analyzer seperti Wireshark pada router transit (R-UTARA), seluruh isi payload data privat dapat dibaca secara transparan tanpa perlindungan kerahasiaan.

#### 3. Bonus: Mekanisme Pengamanan dengan IPsec (GRE over IPsec)
Untuk menambahkan kerahasiaan (*confidentiality*), integritas (*integrity*), dan autentikasi (*authentication*), arsitektur standar industri mengombinasikan GRE dengan IPsec melalui mode **GRE over IPsec**:
1. **Phase 1 ISAKMP / IKE**: Menegosiasikan kanal manajemen yang aman dan pertukaran kunci Diffie-Hellman antara kedua router.
2. **Phase 2 IPsec SA & Transform Set**: Menggunakan protokol ESP (*Encapsulating Security Payload* - IP Protocol 50) dengan algoritma enkripsi kuat (misalnya AES-256) dan fungsi hash integritas (misalnya SHA-256).
3. Paket GRE yang keluar dari `Tunnel0` akan dicegat oleh modul IPsec (Crypto Map atau IPsec Profile), lalu dienkripsi secara penuh ke dalam paket ESP sebelum dikirimkan ke jaringan luar. Dengan arsitektur ini, seluruh data privat internal terlindungi sepenuhnya dari ancaman penyadapan maupun pemalsuan data.

---

## 7. Dynamic Routing OSPF & Redundancy Test

### 7.1 Ketentuan & Berkas Kerja Terpisah
Ketentuan Soal 7 mewajibkan pengerjaan dilakukan pada file terpisah hasil salinan file utama:
- **Nama file kerja final**: `K08_CIDR_OSPF.pkt` (duplikat dari berkas OSPF asli)
- **Tujuan**: Mengganti perutean statis antar-router dengan protokol perutean dinamis OSPFv2 (Single Area, Area 0), serta menguji mekanisme toleransi kesalahan (*fault-tolerance/redundancy*) jika salah satu jalur backbone terputus.

### 7.2 Penambahan Link Cadangan L7 (R-DC <-> R-UTARA)
Untuk menyediakan redundansi jalur, ditambahkan satu tautan darurat L7 yang menghubungkan **R-DC** dan **R-UTARA**:
1. **Pemasangan Modul Serial**: Modul `HWIC-2T` (2-port Serial WAN) dipasang pada slot HWIC `0/0` di R-DC dan R-UTARA.
2. **Pengkabelan**: Menggunakan kabel Serial DCE/DTE antara `R-DC:Serial0/0/0` dan `R-UTARA:Serial0/0/0`.
3. **Alokasi Subnet Baru**: Dipilih subnet baru yang belum digunakan dalam rancangan VLSM sebelumnya:
   - Subnet: `10.65.208.24/30` (Netmask `255.255.255.252`)
   - IP `R-DC:Serial0/0/0`: `10.65.208.25/30` (Clock Rate 64000 DCE)
   - IP `R-UTARA:Serial0/0/0`: `10.65.208.26/30`

### 7.3 Konfigurasi OSPF Area 0 pada Ke-7 Router
Seluruh static route antar-router pada file utama dihapus, lalu dikonfigurasikan OSPF Process ID 1 dalam **Area 0**.

#### 1. Ketentuan Passive Interface
Sesuai instruksi soal, router **dilarang mengirimkan pesan OSPF (Hello/LSA) ke arah LAN pengguna**. Hal ini diimplementasikan menggunakan perintah `passive-interface` pada setiap sub-interface yang mengarah ke LAN:
- Mencegah *security risk* (mencegah penyerang di LAN memalsukan OSPF neighbor/rute palsu).
- Menghemat *bandwidth* dan beban CPU perangkat pengguna (menghindari paket broadcast/multicast OSPF yang tidak diperlukan).

#### 2. Ketentuan Default Route OSPF
- R-PUSAT adalah satu-satunya router yang mengonfigurasi default route statis ke internet (`ip route 0.0.0.0 0.0.0.0 203.0.113.34`).
- Di bawah konfigurasi OSPF R-PUSAT, diaktifkan perintah:
  ```cisco
  router ospf 1
   default-information originate
  ```
- Perintah ini secara otomatis menyebarkan rute default bertipe **O*E2** (OSPF External Type 2) ke seluruh 6 router cabang (R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, R-BARAT), sehingga router-router tersebut tidak memerlukan default route manual.

#### 3. Rincian Konfigurasi OSPF Tiap Router

| Router | Router-ID | Network yang Di-advertise (Area 0) | Passive Interface |
|---|---|---|---|
| **R-PUSAT** | `1.1.1.1` | `10.65.208.0 0.0.0.3` (L1)<br>`10.65.208.4 0.0.0.3` (L2)<br>`10.65.208.8 0.0.0.3` (L3)<br>`10.65.208.12 0.0.0.3` (L4)<br>`10.65.208.16 0.0.0.3` (L5)<br>`10.65.192.0 0.0.7.255` (LAN Balai Kota)<br>`10.65.200.0 0.0.0.31` (LAN Server Balai Kota) | `GigabitEthernet0/2.10`<br>`GigabitEthernet0/2.20` |
| **R-DC** | `2.2.2.2` | `10.65.208.0 0.0.0.3` (L1)<br>`10.65.208.24 0.0.0.3` (L7 Serial)<br>`10.64.128.0 0.0.31.255` (CCTV)<br>`10.64.160.0 0.0.0.255` (Storage DC) | `GigabitEthernet0/1.10`<br>`GigabitEthernet0/1.20` |
| **R-UTARA** | `3.3.3.3` | `10.65.208.4 0.0.0.3` (L2)<br>`10.65.208.20 0.0.0.3` (L6)<br>`10.65.208.24 0.0.0.3` (L7 Serial)<br>`10.64.0.0 0.0.63.255` (WiFi Utara)<br>`10.64.64.0 0.0.1.255` (Sensor Utara) | `GigabitEthernet0/1.10`<br>`GigabitEthernet0/1.20` |
| **R-PELABUHAN** | `4.4.4.4` | `10.65.208.20 0.0.0.3` (L6)<br>`10.64.192.0 0.0.31.255` (SCADA Pelabuhan)<br>`10.64.224.0 0.0.0.127` (Kantor Pelabuhan) | `GigabitEthernet0/1.10`<br>`GigabitEthernet0/1.20` |
| **R-SELATAN** | `5.5.5.5` | `10.65.208.8 0.0.0.3` (L3)<br>`10.65.0.0 0.0.31.255` (WiFi Selatan)<br>`10.65.32.0 0.0.0.127` (PJU Selatan) | `GigabitEthernet0/1.10`<br>`GigabitEthernet0/1.20` |
| **R-TIMUR** | `6.6.6.6` | `10.65.208.12 0.0.0.3` (L4)<br>`10.65.64.0 0.0.31.255` (WiFi Timur)<br>`10.65.96.0 0.0.3.255` (Sensor Timur) | `GigabitEthernet0/1.10`<br>`GigabitEthernet0/1.20` |
| **R-BARAT** | `7.7.7.7` | `10.65.208.16 0.0.0.3` (L5)<br>`10.65.128.0 0.0.31.255` (WiFi Barat)<br>`10.65.160.0 0.0.0.255` (Puskesmas Barat) | `GigabitEthernet0/0.10`<br>`GigabitEthernet0/0.20` |

### 7.4 Verifikasi Status OSPF
#### 1. OSPF Neighbor Adjacency di R-DC
```cisco
R-DC#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:33    10.65.208.1     GigabitEthernet0/0
3.3.3.3           0   FULL/  -        00:00:38    10.65.208.26    Serial0/0/0
```
*Terlihat R-DC membentuk hubungan ketetanggaan `FULL` ke R-PUSAT via GigabitEthernet0/0 dan ke R-UTARA via Serial0/0/0.*

#### 2. Penerimaan Default Route OSPF di Router Cabang
Contoh pada `R-BARAT`:
```cisco
R-BARAT#show ip route ospf
     10.0.0.0/8 is variably subnetted, 17 subnets, 9 masks
O       10.64.0.0/18 [110/3] via 10.65.208.17, GigabitEthernet0/1
O       10.64.128.0/19 [110/3] via 10.65.208.17, GigabitEthernet0/1
O       10.64.160.0/24 [110/3] via 10.65.208.17, GigabitEthernet0/1
O       10.65.192.0/21 [110/2] via 10.65.208.17, GigabitEthernet0/1
O       10.65.200.0/27 [110/2] via 10.65.208.17, GigabitEthernet0/1
O*E2 0.0.0.0/0 [110/1] via 10.65.208.17, 00:04:12, GigabitEthernet0/1
```
*Tanda `O*E2 0.0.0.0/0` membuktikan bahwa R-BARAT menerima default route secara dinamis dari R-PUSAT tanpa perlu konfigurasi statis manual.*

Bukti neighbor dan rute OSPF pada file `K08_CIDR_OSPF.pkt`:

![Neighbor OSPF R-PUSAT](./assets/bukti/Keisya/07-ospf-neighbor-r-pusat.png)

![Rute OSPF](./assets/bukti/Keisya/07-rute-ospf.png)

![Default route OSPF](./assets/bukti/Keisya/07-default-route-ospf.png)

### 7.5 Uji Redundansi & Failover Jalur Backbone
Untuk membuktikan ketahanan sistem jaringan dinamis (*fault-tolerance*), dilakukan pengujian failover:

#### 1. Kondisi Normal (Link L1 Aktif)
Pada kondisi normal, jalur komunikasi dari **Command Center (R-DC)** menuju **Balai Kota (R-PUSAT)** melewati link utama L1:
```text
PC>tracert 10.65.192.3
  1   10.64.160.1 (Gateway R-DC)
  2   10.65.208.1 (R-PUSAT via GigabitEthernet0/0)
  3   10.65.192.3 (PC-PEGAWAI-BALAIKOTA)
```

#### 2. Simulasi Kegagalan Link Backbone (Link L1 Putus)
Interface utama `GigabitEthernet0/0` pada R-DC dipadamkan (`shutdown`) untuk menyimulasikan kabel backbone L1 putus:
```cisco
R-DC(config)#interface GigabitEthernet0/0
R-DC(config-if)#shutdown
```
**Reaksi Protokol OSPF**:
1. Hubungan tetangga (*neighbor adjacency*) antara R-DC dan R-PUSAT via `GigabitEthernet0/0` langsung terputus (*DOWN*).
2. OSPF segera membangkitkan *Link State Update* (LSU) dan menjalankan algoritma Shortest Path First (SPF Dijkstra).
3. Jalur alternatif ditemukan secara otomatis melalui link cadangan **L7** (`Serial0/0/0`) menuju **R-UTARA**, kemudian melewati **L2** menuju **R-PUSAT**.

#### 3. Hasil Pengujian Konektivitas Saat Link L1 Putus
- Uji Ping dari `SRV-STORAGE-DC` (`10.64.160.2`) ke `PC-PEGAWAI-BALAIKOTA` (`10.65.192.3`): **Success (Loss 0%)**.
- Uji Ping ke Server Internet `8.8.8.8`: **Success (Loss 0%)**.
- Hasil `tracert` dari `SRV-STORAGE-DC` ke `PC-PEGAWAI-BALAIKOTA`:
```text
PC>tracert 10.65.192.3
Tracing route to 10.65.192.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      10.64.160.1
  2   0 ms      0 ms      0 ms      10.65.208.26
  3   0 ms      0 ms      0 ms      10.65.208.5
  4   0 ms      0 ms      0 ms      10.65.192.3

Trace complete.
```
**Analisis Failover**:
- **Hop 1** (`10.64.160.1`): Gateway R-DC.
- **Hop 2** (`10.65.208.26`): R-UTARA melalui link darurat Serial L7 (`10.65.208.24/30`).
- **Hop 3** (`10.65.208.5`): R-PUSAT melalui link Gigabit L2.
- **Hop 4** (`10.65.192.3`): Tujuan akhir di Balai Kota.

Screenshot berikut diambil saat `GigabitEthernet0/0` R-DC dimatikan. Neighbor ke R-PUSAT turun, tetapi neighbor `3.3.3.3` melalui `Serial0/0/0` tetap `FULL`. Rute `10.65.192.0/21` berpindah ke `10.65.208.26` dan ping berhasil 5/5.

![Uji failover OSPF melalui L7](./assets/bukti/Keisya/07-ospf-failover-l7.png)

#### 4. Pemulihan Jalur (*Recovery*)
Saat kabel L1 diaktifkan kembali (`no shutdown` pada `GigabitEthernet0/0`), OSPF mendeteksi kembalinya link Gigabit yang memiliki metrik *cost* lebih rendah daripada link Serial, dan secara otomatis memindahkan lalu lintas kembali ke jalur utama L1 tanpa ada gangguan koneksi.

Setelah menunggu konvergensi, neighbor ke R-PUSAT dan R-UTARA kembali berstatus `FULL`:

![Pemulihan neighbor OSPF](./assets/bukti/Keisya/07-ospf-recovery.png)

---

## 8. Layanan Server

Layanan ditempatkan pada dua server sesuai perannya. Pengujian di bawah ini dijalankan pada `K08_CIDR.pkt`, dan konfigurasi DNS serta web yang sama juga diterapkan pada `K08_CIDR_OSPF.pkt`.

### 8.1 SRV-BALAIKOTA - DNS dan HTTP

- IP statis: `10.65.200.2/27`, gateway `10.65.200.1`.
- DNS aktif dengan record berikut:
  - `www.k8.smartcity.lab` dan `srv-balaikota.k8.smartcity.lab` -> `10.65.200.2`
  - `ftp.k8.smartcity.lab`, `mail.k8.smartcity.lab`, dan `srv-storage.k8.smartcity.lab` -> `10.64.160.2`
- HTTP aktif dan menampilkan halaman "Portal Smart City Surabaya" dengan nomor kelompok 8.
- Pengujian `http://www.k8.smartcity.lab` dari PC klien berhasil, sekaligus membuktikan resolusi DNS dan HTTP.

### 8.2 SRV-STORAGE-DC - DHCP CCTV, FTP, dan Email

- IP statis: `10.64.160.2/24`, gateway `10.64.160.1`, DNS `10.65.200.2`.
- DHCP CCTV melayani `10.64.128.0/19` melalui DHCP relay R-DC.
- FTP aktif dengan akun `admin-k8`; hak akses baca/tulis/list/delete/rename telah diaktifkan. Login dan perintah `dir` berhasil dari PC Balai Kota.
- Email memakai domain `k8.smartcity.lab`, akun `admin@k8.smartcity.lab` dan `petugas@k8.smartcity.lab`. Pengujian SMTP/POP3 dari petugas ke admin berhasil.

### 8.3 Hasil uji CIDR dan OSPF

Implementasi layanan yang sama tersedia pada `K08_CIDR.pkt` dan `K08_CIDR_OSPF.pkt`. Pengujian mencakup DHCP, DNS/HTTP, FTP, email, ping antar-LAN, dan ping internet `8.8.8.8`. Pada berkas OSPF, seluruh adjacency R-PUSAT mencapai keadaan `FULL`, semua LAN cabang muncul sebagai rute `O`, dan default route menuju ISP adalah `203.0.113.34`.

Bukti OSPF tersimpan di `assets/bukti/OSPF`.

## Checklist Akhir

Status lengkap per nomor soal dan sisa pekerjaan ada di [CHECKLIST_ETS.md](./CHECKLIST_ETS.md).

