# JARKOM-ETS

## Member

| Nama                      | NRP        | Pembagian Kerja |
| ------------------------- | ---------- | --------------- |
| Yovi Prayudya Rizky Ramadhani | 5027251107 | No 1, 2, dan 4 point 1 |
| ......................... | .......... | ............... |
| ......................... | .......... | ............... |

## Laporan

1. ![image](./assets/soal/soal-1.png)

Kurang lebih topologinya seperti berikut:

![image](./assets/jawaban/topologi-soal-1-1.png)
![image](./assets/jawaban/topologi-soal-1-2.png)

Nah disini kelompok kami menggunakan rancangan topologi skema CIDR agar bisa memetakan antar router dengan mudah
dan juga dari CIDR sendiri memiliki fleksibilitas untuk jaringannya. Konfigurasi beberapa (source: https://cidrcalculator.org/id/ entar ku sederhanain lagi):

R-ISP
![image](./assets/jawaban/config/R-ISP-config.png)

Router diberi nama `R-ISP` lewat `hostname`, lalu tiap interface diberi alamat IP beserta subnet mask-nya dan diaktifkan dengan `no shutdown`:
 
- `Gi0/0` dengan `203.0.113.6/30` ke R-PUSAT.
- `Gi0/1` dengan `8.8.8.1/24` ke Server 8.8.8.8, yang menjadi gateway server tersebut.
Konfigurasi disimpan dengan `write memory`. R-ISP tidak diberi route ke alamat privat internal, sesuai ketentuan Soal 5.

R-PUSAT

![image](./assets/jawaban/config/R-PUSAT-config.png)

Nah disini sama seperti R-ISP untuk command ini dan menyimpannya melalui `write memory`. Setelah itu

![image](./assets/jawaban/config/R-PUSAT-config-1.png)
![image](./assets/jawaban/config/R-PUSAT-config-2.png)

Konfigurasi R-PUSAT terdiri dari tiga bagian:
 
1. **Interface routed**: `Gi0/0` (`203.0.113.5/30`) ke R-ISP dan `Gi0/1` (`10.65.208.1/30`) ke R-DC.
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
| Gi0/0 | R-ISP | - | 203.0.113.5/30 |
| Gi0/1 | R-DC | L1 | 10.65.208.1/30 |
| Gi0/2 | SW-BALAIKOTA | - | sub-interface `.10` dan `.20` |
| Fa0/0/0 | R-BARAT | L5 | VLAN 15, 10.65.208.17/30 |
| Fa0/0/1 | R-UTARA | L2 | VLAN 12, 10.65.208.5/30 |
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
 
Port uplink (trunk) ke router tiap switch: SW-BALAIKOTA `Fa0/1`, SW-DC `Fa0/3`, dan SW-UTARA, SW-PELABUHAN, SW-SELATAN, SW-TIMUR, SW-BARAT di `Fa0/1`. Port lain adalah access port ke VLAN 10 dan VLAN 20.
 
2. ![image](./assets/soal/soal-2.png)

Untuk soal 2 berikut di CSVnya dengan alasan kesalahannya dibawah tabelnya

[Revisi Tabel Jarkom ETS.xlsx](./Revisi%20Tabel%20Jarkom%20ETS.xlsx)

4. ![image](./assets/soal/soal-4.png)

4.1 Interface router dan server memakai IP statis
 
Seluruh interface router dikonfigurasi dengan IP statis lewat CLI (`ip address` pada interface atau sub-interface). Seluruh server dikonfigurasi statis lewat tab **Desktop, IP Configuration**, dengan pilihan **Static**. Perangkat pengguna (PC, access point, dan webcam) tidak diberi IP statis karena akan mendapat IP dinamis dari DHCP.
 
### Alamat interface router
 
| Router | Interface | Alamat | Tujuan |
|---|---|---|---|
| R-ISP | Gi0/0 | 203.0.113.6/30 | R-PUSAT |
| | Gi0/1 | 8.8.8.1/24 | Server 8.8.8.8 |
| R-PUSAT | Gi0/0 | 203.0.113.5/30 | R-ISP |
| | Gi0/1 | 10.65.208.1/30 | L1 |
| | Gi0/2.10 | 10.65.192.1/21 | Pegawai Balai Kota |
| | Gi0/2.20 | 10.65.200.1/27 | Server Balai Kota |
| | Vlan12 | 10.65.208.5/30 | L2 |
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