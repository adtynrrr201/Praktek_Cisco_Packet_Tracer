# Panduan Membuat Berkas .pkt Tiap Lab

Berkas `LAB 2.pkt` sampai `LAB 6.pkt` saat ini isinya identik satu sama lain, sehingga tidak cocok dengan modulnya masing-masing. Dokumen ini merinci apa yang harus dibangun di Cisco Packet Tracer untuk tiap lab.

## Ketentuan umum

**Router yang dipakai: Cisco 2811.** Router ini sudah memiliki FastEthernet0/0 dan FastEthernet0/1 bawaan, dan slotnya menerima modul NM-1FE-TX yang dibutuhkan Lab 4.

**Tipe kabel**

| Sambungan | Kabel |
|---|---|
| Router ↔ Router | Copper Cross-Over |
| Router ↔ PC atau Server (langsung) | Copper Cross-Over |
| Router ↔ Switch | Copper Straight-Through |

Bila ragu, pakai ikon petir **Automatically Choose Connection Type**, lalu periksa lampu tautan berubah hijau di kedua ujung.

**Urutan kerja yang dianjurkan untuk tiap lab**

1. Seret perangkat ke kanvas dan beri nama sesuai tabel.
2. Pasang kabel sesuai tabel sambungan.
3. Buka CLI tiap router, tempel blok **Persiapan: konfigurasi alamat dasar** dari modul lab tersebut.
4. Tempel blok **Langkah kerja** dari modul lab tersebut.
5. Jalankan verifikasi di bawah. Baru setelah lolos, **File → Save As** dengan nama yang benar.

> **Penting:** tiap lab harus dibangun dari kanvas kosong, bukan dari hasil lab sebelumnya. Sisa konfigurasi lab terdahulu akan mengacaukan hasil. Contoh: rute statis Lab 1 yang tertinggal di Lab 2 membuat sebagian lalu lintas tidak melewati EIGRP, sehingga tabel routing tidak seperti pada modul.

---

## LAB 1 — Static Route

Topologi dasar empat router, tanpa tambahan apa pun.

| Perangkat | Tipe | Antarmuka terpakai |
|---|---|---|
| Jakarta | Router 2811 | Fa0/0, Loopback0 |
| Semarang | Router 2811 | Fa0/0, Fa0/1, Loopback0 |
| Surabaya | Router 2811 | Fa0/0, Fa0/1, Loopback0 |
| Bali | Router 2811 | Fa0/0, Loopback0 |

| Dari | Ke | Jaringan |
|---|---|---|
| Jakarta Fa0/0 | Semarang Fa0/0 | 12.12.12.0/24 |
| Semarang Fa0/1 | Surabaya Fa0/0 | 23.23.23.0/24 |
| Surabaya Fa0/1 | Bali Fa0/0 | 34.34.34.0/24 |

Loopback tidak perlu dibuat lewat tab Physical — cukup perintah `interface Loopback0` di CLI.

**Verifikasi sebelum simpan**

- `show ip route` di Jakarta memuat dua baris berawalan `S`.
- Ping Jakarta → 23.23.23.3 dan Jakarta → 34.34.34.4 berhasil.
- Ping Jakarta → 4.4.4.4 **gagal**. Ini memang hasil yang diharapkan di lab ini.

> Berkas `LAB 1.pkt` yang ada sekarang berbeda dari kelima lainnya, jadi kemungkinan besar sudah benar. Tetap jalankan verifikasi di atas untuk memastikan.

---

## LAB 2 — EIGRP

Perangkat dan kabel **sama persis dengan Lab 1**. Perbedaannya hanya pada konfigurasi: tidak ada satu pun rute statis, diganti EIGRP.

**Verifikasi sebelum simpan**

- `show ip eigrp neighbors`: Jakarta 1 tetangga, Semarang 2, Surabaya 2, Bali 1.
- `show ip route` di Jakarta memuat lima baris berawalan `D`.
- Ping Jakarta → 4.4.4.4 **berhasil**. Inilah pembeda utamanya dari Lab 1.
- `show run | include ip route` tidak menghasilkan apa pun. Bila ada isinya, rute statis Lab 1 masih tertinggal dan harus dihapus.

---

## LAB 3 — OSPF Multi Area

Perangkat dan kabel **sama persis dengan Lab 1**. Konfigurasinya OSPF dua area, tanpa EIGRP dan tanpa rute statis.

**Verifikasi sebelum simpan**

- `show ip ospf neighbor` di tiap router: seluruh tetangga berstatus FULL.
- `show ip ospf interface brief` di Bali: Fa0/0 dan Lo0 keduanya di **area 10**. Bila Lo0 masih area 0, konfigurasinya belum mengikuti revisi pada modul.
- `show ip route ospf` di Jakarta: seluruh rute bertanda `O IA`.
- Ping Jakarta → 4.4.4.4 berhasil.

---

## LAB 4 — OSPF Load Balance dan Failover

Ini **Lab 3 ditambah satu tautan kedua**. Cara tercepat: buka `LAB 3.pkt` yang sudah jadi, lakukan penambahan di bawah, lalu **Save As** sebagai `LAB 4.pkt`.

**Penambahan perangkat keras**

1. Klik Router Jakarta → tab **Physical**.
2. Matikan router dengan menekan saklar daya pada gambar perangkat.
3. Seret modul **NM-1FE-TX** dari daftar di kiri ke slot kosong pada gambar router.
4. Nyalakan kembali saklar daya.
5. Ulangi seluruh langkah untuk Router Semarang.

Router harus dalam keadaan mati saat modul dipasang. Bila tidak, modul ditolak.

**Penambahan kabel**

| Dari | Ke | Jaringan |
|---|---|---|
| Jakarta Fa1/0 | Semarang Fa1/0 | 16.16.16.0/24 |

**Verifikasi sebelum simpan**

- `show ip route 4.4.4.4` di Jakarta menampilkan **dua** Routing Descriptor Block, satu via 12.12.12.2 dan satu via 16.16.16.2, keduanya dengan metric sama.
- Bila hanya muncul satu jalur, biasanya cost kedua antarmuka berbeda. Periksa dengan `show ip ospf interface` pada Fa0/0 dan Fa1/0.

Simpan berkas dalam keadaan **cost fa1/0 masih bawaan**, yaitu sebelum percobaan lanjutan `ip ospf cost 100` dijalankan. Percobaan itu bagian dari kegiatan murid, bukan keadaan awal berkas.

---

## LAB 5 — BGP Antar Autonomous System

Topologi empat router yang sama, **ditambah dua PC**. Tidak ada OSPF maupun EIGRP di lab ini — hanya BGP.

| Perangkat | Tipe | Keterangan |
|---|---|---|
| Jakarta, Semarang, Surabaya, Bali | Router 2811 | seperti Lab 1 |
| PC-Jakarta | PC | baru |
| PC-Bali | PC | baru |

**Penambahan kabel**

| Dari | Ke | Jaringan |
|---|---|---|
| Jakarta Fa0/1 | PC-Jakarta FastEthernet0 | 192.168.10.0/24 |
| Bali Fa0/1 | PC-Bali FastEthernet0 | 192.168.20.0/24 |

Fa0/1 pada Jakarta dan Bali memang kosong pada topologi dasar, jadi tidak ada yang perlu dilepas.

**Pengalamatan PC** — klik PC → tab **Desktop** → **IP Configuration**, pilih Static:

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC-Jakarta | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| PC-Bali | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |

**Verifikasi sebelum simpan**

- `show ip bgp summary` di tiap router: kolom State/PfxRcd berisi **angka**, bukan `Idle` atau `Active`. Sesi BGP butuh beberapa puluh detik untuk naik, jadi tunggu sebentar sebelum menyimpulkan gagal.
- `show ip bgp` di Semarang menampilkan sembilan baris seperti pada modul.
- Ping PC-Jakarta → 192.168.20.2 berhasil. Ini bukti akhir lab.

---

## LAB 6 — Access List Standar dan Extended

Lab ini memakai **topologi yang sama sekali berbeda**. Bangun dari kanvas kosong, jangan dari lab mana pun.

| Perangkat | Tipe | Antarmuka terpakai |
|---|---|---|
| R1 | Router 2811 | Fa0/0, Fa0/1 |
| R2 | Router 2811 | Fa0/0, Fa0/1 |
| PC-Klien | PC | FastEthernet0 |
| Server | Server | FastEthernet0 |

| Dari | Ke | Jaringan |
|---|---|---|
| PC-Klien FastEthernet0 | R1 Fa0/0 | 10.10.10.0/24 |
| R1 Fa0/1 | R2 Fa0/0 | 192.168.99.0/24 |
| R2 Fa0/1 | Server FastEthernet0 | 20.20.20.0/24 |

**Pengalamatan PC dan Server** — tab **Desktop** → **IP Configuration**, pilih Static:

| Perangkat | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC-Klien | 10.10.10.10 | 255.255.255.0 | 10.10.10.1 |
| Server | 20.20.20.2 | 255.255.255.0 | 20.20.20.1 |

**Menyalakan layanan HTTP** — klik Server → tab **Services** → **HTTP** → pastikan HTTP berstatus **On**. Tanpa ini, pengujian access list extended pada Tahap 3 tidak bermakna, karena halaman gagal terbuka bukan sebab diblokir.

**Verifikasi sebelum simpan**

- Ping PC-Klien → 20.20.20.2 berhasil.
- Dari PC-Klien, tab **Desktop** → **Web Browser** → ketik `20.20.20.2` → Go. Halaman terbuka.

Simpan berkas **hanya sampai Tahap 1 selesai**, yaitu sebelum access list mana pun dipasang. Tahap 2 dan 3 adalah pekerjaan murid. Bila access list ikut tersimpan, murid membuka berkas yang sudah memblokir lalu lintas dan tidak dapat mengamati keadaan sebelum-sesudah.

---

## Ringkasan singkat

| Lab | Dasar | Yang ditambahkan | Simpan dalam keadaan |
|---|---|---|---|
| 1 | kanvas kosong | — | rute statis terpasang |
| 2 | kanvas kosong | — | EIGRP jalan, tanpa rute statis |
| 3 | kanvas kosong | — | OSPF dua area, seluruh tetangga FULL |
| 4 | salinan Lab 3 | NM-1FE-TX + tautan 16.16.16.0/24 | cost fa1/0 masih bawaan |
| 5 | kanvas kosong | 2 PC di Jakarta dan Bali | BGP jalan, tanpa OSPF/EIGRP |
| 6 | kanvas kosong | topologi tersendiri | Tahap 1 selesai, tanpa access list |
