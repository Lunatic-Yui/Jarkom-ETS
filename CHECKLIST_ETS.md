# Checklist ETS Jaringan Komputer - Kelompok 8

## Status per soal

| No | Ketentuan | Status | Bukti / catatan |
|---|---|---|---|
| 1 | Topologi CIDR sesuai Gambar 1, L1-L6, R-ISP, dan Server 8.8.8.8 | Selesai | [K08_CIDR.pkt](./project/K08_CIDR.pkt), [K08_CIDR_OSPF.pkt](./project/K08_CIDR_OSPF.pkt), dan [topologi CIDR](./assets/jawaban/topologi-cidr-full.png) |
| 1 | Desain tiap lokasi, VLAN, media, dan penambahan port dijelaskan | Selesai | [Desain tiap lokasi](./README.md#desain-tiap-lokasi), [alasan pemilihan perangkat](./README.md#alasan-pemilihan-perangkat-dan-media), dan [penambahan port R-PUSAT](./README.md#penambahan-port-pada-r-pusat) |
| 1 | Nama perangkat diganti sesuai tempat dan fungsi | Selesai | [Topologi CIDR](./assets/jawaban/topologi-cidr-full.png) dan [topologi OSPF](./assets/jawaban/topologi-ospf-full.png) |
| 1 | Label tiap LAN memuat nama subnet, Network ID/prefix, dan jumlah host | Selesai | [Topologi CIDR](./assets/jawaban/topologi-cidr-full.png) dan [topologi OSPF](./assets/jawaban/topologi-ospf-full.png) |
| 2 | Tabel VLSM berisi subnet, network ID, prefix, host range, broadcast, dan gateway | Selesai | [Screenshot VLSM](./assets/jawaban/tabel/tabel-revisi-vlsm.png) dan sheet `Revisi VLSM` pada [file Excel](./Revisi%20Tabel%20Jarkom%20ETS.xlsx) |
| 2 | Tabel CIDR per router | Selesai | [Screenshot CIDR](./assets/jawaban/tabel/tabel-revisi-cidr.png) dan sheet `Revisi CIDR` pada [file Excel](./Revisi%20Tabel%20Jarkom%20ETS.xlsx) |
| 2 | Jawaban UTS, perbaikan, dan alasan dicatat | Selesai | [Revisi Tabel Jarkom ETS.xlsx](./Revisi%20Tabel%20Jarkom%20ETS.xlsx) |
| 2 | Mask `/25`, `/27`, dan `/30` sudah benar | Selesai | [Tabel subnetting](./README.md#soal-2-tabel-vlsm-cidr-dan-revisi) |
| 3 | Default route cabang ke pusat dan pusat ke ISP | Selesai | [Aturan static routing](./README.md#31-aturan-static-routing) |
| 3 | Static route downstream pada R-PUSAT dan R-UTARA | Selesai | [Tabel static route](./README.md#32-tabel-konfigurasi-static-route) dan [bukti R-PUSAT](./assets/bukti/Keisya/03-static-route-r-pusat.png) |
| 3 | Koneksi antar-LAN | Selesai | [Uji koneksi](./README.md#33-uji-konektivitas-antar-lan) dan [ping CIDR](./assets/bukti/CIDR/04-ping-antarkota-pju-selatan.png) |
| 4 | Interface router dan server memakai IP statis | Selesai | [IP statis](./README.md#41-ip-statis) |
| 4 | Perangkat pengguna mendapat IP, gateway, dan DNS dari DHCP | Selesai | [Cek DHCP](./README.md#43-cek-dhcp) dan [ipconfig PC Balai Kota](./assets/bukti/CIDR/02-dhcp-pc-balai-ipconfig.png) |
| 4 | CCTV Kota mendapat DHCP dari SRV-STORAGE-DC | Selesai | [DHCP CCTV](./assets/bukti/OSPF/02-dhcp-cctv-helper.png) |
| 5 | NAT Overload ke `203.0.113.33` | Selesai | [Konfigurasi NAT R-PUSAT](./assets/bukti/Keisya/05-nat-pat-r-pusat.png) |
| 5 | R-ISP tidak memiliki rute privat internal | Selesai | [Tabel rute R-ISP](./assets/bukti/Keisya/05-rute-router-isp.png) dan [ping internet](./assets/bukti/Keisya/05-ping-internet.png) |
| 6 | Tunnel GRE `192.168.8.0/30` dari R-PUSAT ke R-PELABUHAN | Selesai | [Status Tunnel0](./assets/bukti/Keisya/06-tunnel-gre-r-pusat.png) |
| 6 | Trafik Balai Kota-Pelabuhan melewati tunnel | Selesai | [Uji traceroute](./README.md#63-uji-traceroute) |
| 7 | OSPF dikerjakan pada file terpisah | Selesai | [K08_CIDR_OSPF.pkt](./project/K08_CIDR_OSPF.pkt) |
| 7 | OSPFv2 single area dan passive interface LAN | Selesai | [OSPF Area 0](./README.md#73-ospf-area-0) dan [neighbor FULL](./assets/bukti/OSPF/09-ospf-neighbors-full.png) |
| 7 | Default route dibagikan dengan `default-information originate` | Selesai | [Default route OSPF](./assets/bukti/Keisya/07-default-route-ospf.png) |
| 7 | Link cadangan L7 Serial R-DC-R-UTARA | Selesai | [Topologi OSPF](./assets/jawaban/topologi-ospf-full.png) dan [neighbor R-PUSAT](./assets/bukti/Keisya/07-ospf-neighbor-r-pusat.png) |
| 7 | Failover saat L1 dimatikan | Selesai | [Bukti failover L7](./assets/bukti/Keisya/07-ospf-failover-l7.png) dan [pemulihan L1](./assets/bukti/Keisya/07-ospf-recovery.png) |
| 8 | DNS dan HTTP aktif di SRV-BALAIKOTA | Selesai | [Record DNS](./assets/bukti/Keisya/11-dns-records-k8-smartcity-lab.png) dan [portal web](./assets/bukti/Keisya/08-web-portal-smart-city.png) |
| 8 | DHCP CCTV, FTP, dan email aktif di SRV-STORAGE-DC | Selesai | [DHCP CCTV](./assets/bukti/OSPF/02-dhcp-cctv-helper.png), [FTP](./assets/bukti/CIDR/05-ftp-login-dir-success.png), [kirim email](./assets/bukti/Keisya/09-email-send-k8-smartcity-lab.png), dan [terima email](./assets/bukti/Keisya/10-email-receive-k8-smartcity-lab.png) |

## Bukti pengujian

### `assets/bukti/CIDR/`

- `01-web-dns-www-k8-local.png`
- `02-dhcp-pc-balai-ipconfig.png`
- `03-ping-dns-storage-wan.png`
- `04-ping-antarkota-pju-selatan.png`
- `05-ftp-login-dir-success.png`
- `06-email-petugas-to-admin-success.png`
- `07-dns-records.png`
- `08-pool-pju-mask-25.png`
- `09-dhcp-pju-mask-25.png`
- `10-ping-pju-timur-zero-loss.png`

### `assets/bukti/OSPF/`

- `01-dhcp-pju-25.png`
- `02-dhcp-cctv-helper.png`
- `03-web-dns-success.png`
- `04-email-success.png`
- `05-ftp-success.png`
- `06-ping-dns-storage.png`
- `07-ping-antarkota.png`
- `08-ping-internet.png`
- `09-ospf-neighbors-full.png`
- `10-ospf-routes.png`
- `11-default-route.png`
- `12-isp-return-route.png`
- `13-wan-label.png`
- `14-ping-internet-setelah-fix.png`

### `assets/bukti/Keisya/`

- `03-static-route-r-pusat.png`
- `05-nat-pat-r-pusat.png`
- `05-ping-internet.png`
- `05-rute-router-isp.png`
- `06-tunnel-gre-r-pusat.png`
- `07-default-route-ospf.png`
- `07-ospf-failover-l7.png`
- `07-ospf-neighbor-r-pusat.png`
- `07-ospf-recovery.png`
- `07-rute-ospf.png`
- `08-web-portal-smart-city.png`
- `09-email-send-k8-smartcity-lab.png`
- `10-email-receive-k8-smartcity-lab.png`
- `11-dns-records-k8-smartcity-lab.png`

## Pembagian A / B / C

| Anggota | Peran | Pekerjaan |
|---|---|---|
| Yovi (5027251107) | A - Topologi dan subnetting | Topologi CPT, VLSM/CIDR, tabel revisi, penamaan, label, IP dasar, serta video bagian 0-2 |
| Askhabul (5027251049) | B - Routing dan DHCP | Static routing, DHCP PC/CCTV, NAT, GRE, serta video bagian 3-6 |
| Keisya (5027251068) | C - OSPF dan layanan | OSPF, L7, failover, DNS, web, FTP, email, serta video bagian 7-9 |

README, laporan, bukti, dan persiapan video dikerjakan bersama. Tiap anggota menulis dan mengecek bagian yang menjadi tugasnya.
