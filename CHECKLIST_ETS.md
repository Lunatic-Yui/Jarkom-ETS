# Checklist ETS Jaringan Komputer - Kelompok 8

Ditinjau ulang 14 November 2026 terhadap lembar soal lengkap dari myITS
Classroom dan isi repo `Lunatic-Yui/Jarkom-ETS`.

## Status per soal

| No | Ketentuan | Status | Bukti / catatan |
|---|---|---|---|
| 1 | Topologi CIDR sesuai Gambar 1, L1-L6, R-ISP, Server 8.8.8.8 | Selesai | `project/K08_CIDR.pkt`, `project/K08_CIDR_OSPF.pkt` |
| 1 | Desain tiap lokasi (switch, VLAN, media, penambahan port) dijelaskan | Selesai | README bagian "Desain tiap lokasi", "Alasan pemilihan perangkat", "Penambahan port pada R-PUSAT" |
| 1 | Nama perangkat diganti (R-PUSAT, SW-BALAIKOTA, PC-CCTV-KOTA, dst.) | Selesai | Terlihat pada tangkapan layar topologi |
| 1 | Label tiap LAN berisi nama subnet, Network ID/prefix, jumlah host | Selesai | Seluruh label diperiksa dalam viewport atas dan bawah; bukti ada di `assets/jawaban/topologi-*-rapi.png` |
| 2 | Tabel VLSM: subnet, network ID, prefix, host range, broadcast, gateway | Selesai | Sheet `Revisi VLSM`, 20 subnet termasuk L1-L6 |
| 2 | Tabel CIDR (summary per router) | Selesai | Sheet `Revisi CIDR` |
| 2 | Tabel revisi: jawaban UTS, perbaikan, alasan | Selesai | Baris "Kesalahan/Alasan" pada ketiga sheet |
| 2 | Mask `/25`, `/27`, `/30` valid (bukan angka rusak) | Selesai | Diperbaiki menjadi `255.255.255.128`, `255.255.255.224`, `255.255.255.252` |
| 3 | Default route hierarkis (cabang -> pusat -> ISP) | Selesai | README 3.1-3.2; R-PUSAT default ke `203.0.113.34` |
| 3 | Static route downstream R-PUSAT dan R-UTARA | Selesai | README 3.2 |
| 3 | Konektivitas antar-LAN | Selesai | Ping 0% loss pada bukti CIDR |
| 4 | Interface router dan server IP statis | Selesai | README 4.1 |
| 4 | Perangkat pengguna DHCP lengkap gateway + DNS | Selesai | 7 pool router + lease terverifikasi |
| 4 | CCTV Kota DHCP dari server LAN Storage DC, bukan router | Selesai | `ip helper-address 10.64.160.2` di R-DC `Gi0/1.10`; lease `10.64.128.3/19` |
| 5 | NAT Overload (PAT) ke `203.0.113.33` | Selesai | `ip nat inside source list 1 interface Gi0/0 overload` |
| 5 | R-ISP tidak memiliki rute privat internal | Selesai (diperbaiki 13 Nov) | Rute `10.64.0.0/10` dihapus; tabel rute ISP kini hanya `8.8.8.0/24` dan `203.0.113.32/30`; ping internet tetap 0% loss |
| 6 | Tunnel GRE `192.168.8.0/30` R-PUSAT-R-PELABUHAN | Selesai | `Tunnel0 192.168.8.1/30` status up/up di R-PUSAT |
| 6 | Trafik Balai Kota-Pelabuhan lewat tunnel | Selesai | `tracert` hop `192.168.8.2` (README 6.3) |
| 7 | File OSPF terpisah | Selesai | `project/K08_CIDR_OSPF.pkt` |
| 7 | OSPFv2 single area, passive interface LAN | Selesai | README 7.3 |
| 7 | Default route disebarkan `default-information originate` | Selesai | `O*E2 0.0.0.0/0` di router cabang |
| 7 | Link cadangan L7 Serial R-DC-R-UTARA | Selesai | `10.65.208.24/30`, adjacency FULL |
| 7 | Uji failover L1 | Selesai | README 7.5, ping 0% loss saat L1 shutdown |
| 8 | DNS + HTTP di SRV-BALAIKOTA | Selesai di `K08_CIDR.pkt` | 5 record `k8.smartcity.lab` (www, srv-balaikota, ftp, mail, srv-storage); record lama `k8.local` sudah dihapus |
| 8 | Judul halaman web "Portal Smart City Surabaya" + nomor kelompok | Selesai | Tampil di `www.k8.smartcity.lab`, bukti `assets/bukti/Keisya/08-web-portal-smart-city.png` |
| 8 | DHCP CCTV + FTP + Email di SRV-STORAGE-DC | Selesai di `K08_CIDR.pkt` | FTP `admin-k8`, domain email sudah `k8.smartcity.lab` |
| 8 | Bukti layanan | Selesai | Bukti DNS, web, DHCP CCTV, FTP, kirim email, dan terima email ditampilkan langsung pada README bagian 8 |

## Yang belum selesai

1. **Domain email di `K08_CIDR_OSPF.pkt`** masih `k8.com`, sedangkan record
   DNS sudah `mail.k8.smartcity.lab`. Ubah lewat Services > EMAIL pada
   SRV-STORAGE-DC, lalu simpan ulang berkas.
2. **Video demonstrasi 30 menit** belum direkam dan belum diunggah sebagai
   YouTube *unlisted*. Wajib diselesaikan pemilik akun.
3. **ZIP final** harus dibuat ulang setiap kali README, bukti, atau link
   video berubah.
4. **Uji ulang sebelum rekaman**: DHCP 8 LAN + CCTV, matriks ping antar-LAN,
   `show ip nat translations` saat ada trafik, `tracert` GRE, neighbor OSPF
   semua router, failover `ping -t` + `tracert`, lalu recovery.

## Bukti pengujian

- CIDR: `assets/bukti/CIDR` (10 gambar)
- OSPF: `assets/bukti/OSPF` (13 gambar)
- Bukti tambahan sesi 13-14 November: `assets/bukti/Keisya` (11 gambar)

## Perbaikan 13 November 2026

- Menghapus `ip route 10.64.0.0 255.192.0.0 203.0.113.33` dari R-ISP pada
  file OSPF karena melanggar ketentuan Soal 5. Pengujian ulang
  `ping 8.8.8.8` dari PC Balai Kota berhasil 4/4 (0% loss), membuktikan NAT
  pada R-PUSAT tetap bekerja.
- Menyelaraskan label WAN OSPF menjadi `203.0.113.32/30`.
- Memperbaiki record DNS/HTTP OSPF dan mask pool PJU menjadi `/25`.
- Mengganti nama ZIP menjadi `ProjectJarkom_K8_107_049_068.ZIP` dan
  menyusun ulang isinya.

## Perbaikan dokumentasi 14 November 2026

- Menghapus seluruh karakter em dash dan en dash dari README dan checklist,
  termasuk sisa karakter rusak sisa salah encoding yang terbaca sebagai
  mojibake.
- Mengurutkan ulang README menjadi 1 Topologi, 2 Tabel VLSM/CIDR, 3 Static
  Routing, 4 IP dan DHCP, 5 NAT, 6 GRE, 7 OSPF, 8 Layanan. Sebelumnya
  bagian Soal 3 berada setelah Soal 4.
- Menyelaraskan alamat PC pada narasi pengujian dengan lease DHCP yang
  benar (`.3`, bukan `.2`). Bukti `ipconfig` menunjukkan
  PC-PEGAWAI-BALAIKOTA mendapat `10.65.192.3`.
- Menambahkan alasan pemilihan CIDR yang sebelumnya hanya satu kalimat.
- Menghapus kalimat yang menyebut teks Soal 8 tidak tersedia di repo.

## Pembagian resmi A / B / C

| Anggota | Peran | Pekerjaan |
|---|---|---|
| Yovi (5027251107) | A - Arsitek Dasar dan Pengalamatan | Excel VLSM/CIDR + tabel revisi, topologi CPT, penamaan, label, IP dasar |
| Askhabul (5027251049) | B - Spesialis Rute dan Keamanan | Static routing, NAT, Tunnel GRE, file OSPF + L7 |
| Keisya (5027251068) | C - SysAdmin dan Dokumentasi | DHCP PC dan CCTV, DNS/web/FTP/email, README, bukti, laporan |

Urutan video tetap 0-9, pembicara mengikuti pemilik pekerjaan:

| Bagian | Menit | Pembicara |
|---|---|---|
| 0 Pembukaan | 1 | A |
| 1 Topologi | 2 | A |
| 2 Subnetting | 2 | A |
| 3 Static routing | 4 | B |
| 4 DHCP | 3 | C |
| 5 NAT | 2 | B |
| 6 Tunnel GRE | 3 | B |
| 7 OSPF dan failover | 5 | B |
| 8 Layanan aplikasi | 5 | C |
| 9 Penutup | 3 | C |

Total 30 menit. Urutan bagian tidak diacak; hanya pembicara yang bergantian
karena pembagian kerja tidak berbentuk blok.

## Kendala login Packet Tracer

Packet Tracer 9.0.1 saat dibuka ulang meminta login Cisco Networking
Academy. Sesi sebelumnya sudah kedaluwarsa, sehingga jendela utama tidak
bisa dioperasikan sampai pemilik akun login. File tetap bisa dibuka dan
topologi tampil normal, jadi berkas `.pkt` tidak rusak.

Rencana perbaikan: uninstall lalu install ulang memakai
`CiscoPacketTracer_901_win_64bit.exe` di folder Downloads, setelah semua
file `.pkt` dibackup ke `Jarkom-ETS-BACKUP-20261113`. Proses ini butuh
persetujuan UAC dari pemilik laptop.

Yang belum bisa dijalankan ulang: perubahan domain, uji ping menyeluruh,
dan tangkapan layar baru. Bukti yang ada diambil dari file yang sama pada
13 November pagi/siang, jadi masih sah.

## Catatan DNS di LAN Server dan Storage DC

Soal 8 menyebut "DNS di LAN Server dan Storage DC". Saat ini DNS hanya aktif
di SRV-BALAIKOTA, sedangkan SRV-STORAGE-DC memakai `10.65.200.2` sebagai DNS
client. Kalau dosen menafsirkan DNS harus ada di kedua server,
SRV-STORAGE-DC perlu ditambah layanan DNS dengan record yang sama.
