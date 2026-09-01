# Praktek Cisco Packet Tracer

Kumpulan enam modul praktikum jaringan menggunakan Cisco Packet Tracer, disusun berurutan dari routing statis sampai access list. Tiap lab berisi langkah konfigurasi lengkap, perintah verifikasi, contoh keluaran yang diharapkan, dan catatan penjelas.

## Topologi utama

Lab 1 sampai 5 memakai satu topologi yang sama: empat router berderet, masing-masing dengan satu antarmuka loopback sebagai penanda identitas.

```text
    Jakarta            Semarang            Surabaya            Bali
   lo0 1.1.1.1        lo0 2.2.2.2         lo0 3.3.3.3        lo0 4.4.4.4
       |                 |    |               |    |             |
     fa0/0  12.12.12.0  fa0/0 fa0/1 23.23.23.0 fa0/0 fa0/1 34.34.34.0 fa0/0
       +-----------------+    +---------------+    +-------------+
           .1      .2            .2      .3           .3      .4
```

Lab 6 memakai topologi tersendiri berupa dua router dengan satu PC dan satu server.

## Daftar lab

| No | Lab | Materi | Alokasi waktu |
|---|---|---|---|
| 1 | [Static Route](LAB%201/LAB%201.md) | Rute statis, membaca tabel routing | 2 jam pelajaran |
| 2 | [Routing Dinamis EIGRP](LAB%202/LAB%202.md) | EIGRP satu AS, perhitungan metric | 2 jam pelajaran |
| 3 | [Routing Dinamis OSPF Multi Area](LAB%203/LAB%203.md) | OSPF dua area, Area Border Router | 3 jam pelajaran |
| 4 | [OSPF Load Balance dan Failover](LAB%204/LAB%204.md) | Dua jalur cost sama, perpindahan jalur | 3 jam pelajaran |
| 5 | [Routing Dinamis BGP](LAB%205/LAB%205.md) | BGP antar Autonomous System, AS Path | 3 jam pelajaran |
| 6 | [Access List Standar dan Extended](LAB%206/LAB%206.md) | Penyaringan lalu lintas dan penempatannya | 3 jam pelajaran |

## Isi tiap folder

Setiap folder `LAB n` memuat dua berkas:

- `LAB n.md` — modul praktikum: topologi, tabel pengalamatan, langkah konfigurasi, dan verifikasi.
- `LAB n.pkt` — berkas Cisco Packet Tracer untuk lab tersebut.

Modul Lab 1 sampai 5 mengulang bagian **Topologi utama** dan **Persiapan: konfigurasi alamat dasar** di bagian atas. Pengulangan ini disengaja agar tiap lab dapat dikerjakan mandiri tanpa membuka berkas lain.

> **Perhatian — berkas .pkt belum sesuai modul.** `LAB 2.pkt` sampai `LAB 6.pkt` saat ini isinya identik satu sama lain, padahal Lab 4, 5, dan 6 menuntut topologi yang berbeda. Langkah membangun ulang tiap berkas, lengkap dengan daftar perangkat, port, dan tipe kabel, ada di **[PANDUAN-BUAT-PKT.md](PANDUAN-BUAT-PKT.md)**.

## Cara memakai

1. Buka `LAB n.pkt` dengan Cisco Packet Tracer (versi 8.x atau lebih baru).
2. Ikuti `LAB n.md` dari atas: kerjakan bagian persiapan lebih dulu, baru langkah kerja lab.
3. Jalankan perintah pada bagian **Verifikasi** dan bandingkan hasilnya dengan contoh keluaran di modul.

## Catatan revisi

Beberapa lab memuat blok **REVISI** yang menandai perbaikan terhadap modul asli, antara lain:

- **Lab 2** — penulisan `network` dilengkapi wildcard mask.
- **Lab 3** — loopback Bali dipindahkan dari area 0 ke area 10 agar tidak melanggar aturan backbone OSPF; ditambahkan `router-id` manual.
- **Lab 4** — ditambahkan jalur kedua 16.16.16.0/24 agar pembagian beban benar-benar dapat diamati.
- **Lab 5** — ditambahkan antarmuka LAN pada Jakarta dan Bali yang sebelumnya tidak pernah dibuat.
- **Lab 6** — topologi dua router dibangun dari awal, lengkap dengan rute statis.
