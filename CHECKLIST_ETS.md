# Checklist ETS Jaringan Komputer — Kelompok 8

Ditinjau ulang 13 November 2026 terhadap soal 1, 2, dan 4 yang tersedia di
`assets/soal`, serta ketentuan soal 3, 5, 6, 7, dan 8 yang dirangkum di README.

## Status per soal

| No | Ketentuan | Status | Bukti / catatan |
|---|---|---|---|
| 1 | Topologi CIDR sesuai Gambar 1, L1–L6, R-ISP, Server 8.8.8.8 | Selesai | `project/K08_CIDR.pkt`, `project/K08_CIDR_OSPF.pkt` |
| 1 | Desain tiap lokasi (switch, VLAN, media, penambahan port) dijelaskan | Selesai | README bagian "Desain tiap lokasi", "Alasan pemilihan perangkat", "Penambahan port pada R-PUSAT" |
| 1 | Nama perangkat diganti (R-PUSAT, SW-BALAIKOTA, PC-CCTV-KOTA, dst.) | Selesai | Terlihat pada tangkapan layar topologi |
| 1 | Label tiap LAN berisi nama subnet, Network ID/prefix, jumlah host | Selesai, perlu cek visual akhir | Label terverifikasi: Pegawai Balai Kota, Server Balai Kota, Server & Storage DC, Puskesmas & Sekolah Barat, WiFi Publik Timur, PJU Selatan, WAN. Sisanya di luar viewport saat audit. |
| 2 | Tabel VLSM: subnet, network ID, prefix, host range, broadcast, gateway | Selesai | Sheet `Revisi VLSM`, 20 subnet termasuk L1–L6 |
| 2 | Tabel CIDR (summary per router) | Selesai | Sheet `Revisi CIDR` |
| 2 | Tabel revisi: jawaban UTS, perbaikan, alasan | Selesai | Baris "Kesalahan/Alasan" pada ketiga sheet |
| 2 | Mask `/25`, `/27`, `/30` valid (bukan angka rusak) | Selesai | Diperbaiki menjadi `255.255.255.128`, `255.255.255.224`, `255.255.255.252` |
| 3 | Default route hierarkis (cabang → pusat → ISP) | Selesai | README 3.1–3.2; R-PUSAT default ke `203.0.113.34` |
| 3 | Static route downstream R-PUSAT dan R-UTARA | Selesai | README 3.2 |
| 3 | Konektivitas antar-LAN | Selesai | Ping 0% loss pada bukti CIDR |
| 4 | Interface router dan server IP statis | Selesai | README 4.1 |
| 4 | Perangkat pengguna DHCP lengkap gateway + DNS | Selesai | 7 pool router + lease terverifikasi |
| 4 | CCTV Kota DHCP dari server LAN Storage DC, bukan router | Selesai | `ip helper-address 10.64.160.2` di R-DC `Gi0/1.10`; lease `10.64.128.3/19` |
| 5 | NAT Overload (PAT) ke `203.0.113.33` | Selesai | `ip nat inside source list 1 interface Gi0/0 overload` |
| 5 | R-ISP tidak memiliki rute privat internal | Selesai (diperbaiki 13 Nov) | Rute `10.64.0.0/10` dihapus; tabel rute ISP kini hanya `8.8.8.0/24` dan `203.0.113.32/30`; ping internet tetap 0% loss |
| 6 | Tunnel GRE `192.168.8.0/30` R-PUSAT–R-PELABUHAN | Selesai | `Tunnel0 192.168.8.1/30` status up/up di R-PUSAT |
| 6 | Trafik Balai Kota–Pelabuhan lewat tunnel | Selesai | `tracert` hop `192.168.8.2` (README 6.3) |
| 7 | File OSPF terpisah | Selesai | `project/K08_CIDR_OSPF.pkt` |
| 7 | OSPFv2 single area, passive interface LAN | Selesai | README 7.3 |
| 7 | Default route disebarkan `default-information originate` | Selesai | `O*E2 0.0.0.0/0` di router cabang |
| 7 | Link cadangan L7 Serial R-DC–R-UTARA | Selesai | `10.65.208.24/30`, adjacency FULL |
| 7 | Uji failover L1 | Selesai | README 7.5, ping 0% loss saat L1 shutdown |
| 8 | DNS + HTTP di SRV-BALAIKOTA | Selesai | `www.k8.local` berhasil |
| 8 | DHCP CCTV + FTP + Email di SRV-STORAGE-DC | Selesai | FTP `admin-k8`, email `k8.com` |
| 8 | Bukti layanan | Selesai | `assets/bukti/CIDR`, `assets/bukti/OSPF` |

## Yang belum selesai / risiko

1. **Video demonstrasi** belum direkam dan belum diunggah sebagai YouTube *unlisted*.
   Ini wajib diselesaikan pemilik akun.
2. **Teks asli soal 3, 5, 6, 7, dan 8 tidak ada di repo** (hanya soal 1, 2, 4).
   Domain `k8.local`/`k8.com`, akun FTP/email, dan pembagian layanan mengikuti
   topologi + README. Cocokkan sekali lagi dengan lembar soal before submit.
3. **Label LAN** perlu dicek visual terakhir untuk 7 subnet yang tidak sempat
   masuk viewport audit (CCTV, WiFi Utara, Sensor Utara, WiFi Selatan,
   WiFi Barat, Sensor Banjir, SCADA/Kantor Pelabuhan).
4. **File asli** `project/Networking-ETS-K-08.pkt` dan
   `project/Networking-ETS-K-08-OSPF.pkt` sengaja tidak diubah. File final
   ditambahkan sebagai `K08_CIDR.pkt` dan `K08_CIDR_OSPF.pkt`.
5. **ZIP** di `Jarkom-ETS-WORK` adalah paket submit; isinya perlu dibuat ulang
   setiap kali README/video berubah.

## Bukti pengujian

- CIDR: `assets/bukti/CIDR` (10 gambar)
- OSPF: `assets/bukti/OSPF` (13 gambar)

## Perbaikan audit 13 November 2026

- Menghapus `ip route 10.64.0.0 255.192.0.0 203.0.113.33` dari R-ISP pada
  file OSPF karena melanggar ketentuan Soal 5. Pengujian ulang
  `ping 8.8.8.8` dari PC Balai Kota berhasil 4/4 (0% loss), membuktikan NAT
  pada R-PUSAT tetap bekerja.
- Menyelaraskan label WAN OSPF menjadi `203.0.113.32/30`.
- Memperbaiki record DNS/HTTP OSPF dan mask pool PJU menjadi `/25`.

## Audit terhadap lembar soal asli (13 November 2026)

Lembar soal lengkap sudah dicek. Ada empat hal yang belum sesuai dan harus
dibereskan sebelum deadline 15 November 2026, 23.59 WIB.

| No | Temuan | Dampak | Status |
|---|---|---|---|
| 1 | Domain layanan masih `k8.local` / `k8.com`, padahal soal mewajibkan `k8.smartcity.lab` | Poin Soal 8 (15) bisa hilang | Belum diperbaiki |
| 2 | Halaman web berjudul "Pemerintah Kota - Kelompok 8", belum "Portal Smart City Surabaya" | Tidak sesuai kalimat soal | Belum diperbaiki |
| 3 | Video demo 30 menit (bagian 0-9) belum ada | Syarat output + 25 poin demo | Belum |
| 4 | Nama ZIP masih memakai NRP lengkap; format wajib 3 angka terakhir tiap NRP | Berkas bisa ditolak | Belum diperbaiki |

Yang sudah sesuai dengan lembar soal:

- WAN `203.0.113.32/30` (K=8, `203.0.113.(4x8)`), R-PUSAT `.33`, R-ISP `.34`.
- Server internet `8.8.8.0/24`, gateway `8.8.8.1` di R-ISP.
- Tunnel `192.168.8.0/30`, R-PUSAT `.1`, R-PELABUHAN `.2`.
- Gateway tiap LAN memakai host pertama (`.1`).
- Static route R-PUSAT ke seluruh LAN R-DC, R-UTARA, R-PELABUHAN,
  R-SELATAN, R-TIMUR, R-BARAT; R-UTARA ke kedua LAN R-PELABUHAN.
- Default route hanya mengarah ke router di atasnya.
- Perangkat pengguna DHCP lengkap gateway + DNS; CCTV lewat DHCP relay ke
  SRV-STORAGE-DC.
- NAT overload ke `203.0.113.33`; R-ISP tanpa rute privat.
- GRE membawa trafik Balai Kota-Pelabuhan; `Tunnel0` up/up.
- OSPF Area 0, L7 cadangan, passive interface LAN, default originate.
- Failover L1 terbukti (rute pindah ke Serial L7, ping 5/5) dan recovery
  kembali `FULL`.
- Spreadsheet berisi tabel kebutuhan, revisi VLSM, dan supernet per router.

Catatan tambahan: Soal 8 menyebut "DNS di LAN Server dan Storage DC".
Saat ini DNS hanya aktif di SRV-BALAIKOTA. SRV-STORAGE-DC memakai
`10.65.200.2` sebagai DNS client. Kalau dosen menafsirkan DNS harus ada di
kedua server, SRV-STORAGE-DC perlu ditambah layanan DNS dengan record yang
sama.
