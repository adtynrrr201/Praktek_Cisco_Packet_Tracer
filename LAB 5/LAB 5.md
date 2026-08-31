# LAB 5 — Routing Dinamis BGP Antar Autonomous System

> **Alokasi waktu:** 3 jam pelajaran · **Bentuk:** praktik terbimbing lalu mandiri

## Topologi utama dan rencana pengalamatan

Empat router disusun berderet. Setiap router diberi antarmuka loopback sebagai penanda identitas yang akan diumumkan oleh protokol routing.

```text
    Jakarta            Semarang            Surabaya            Bali
   lo0 1.1.1.1        lo0 2.2.2.2         lo0 3.3.3.3        lo0 4.4.4.4
       |                 |    |               |    |             |
     fa0/0  12.12.12.0  fa0/0 fa0/1 23.23.23.0 fa0/0 fa0/1 34.34.34.0 fa0/0
       +-----------------+    +---------------+    +-------------+
           .1      .2            .2      .3           .3      .4
```

| Router | Antarmuka | Alamat IP | Terhubung ke |
|---|---|---|---|
| Jakarta | FastEthernet0/0 | 12.12.12.1/24 | Semarang |
| Jakarta | Loopback0 | 1.1.1.1/32 | — |
| Semarang | FastEthernet0/0 | 12.12.12.2/24 | Jakarta |
| Semarang | FastEthernet0/1 | 23.23.23.2/24 | Surabaya |
| Semarang | Loopback0 | 2.2.2.2/32 | — |
| Surabaya | FastEthernet0/0 | 23.23.23.3/24 | Semarang |
| Surabaya | FastEthernet0/1 | 34.34.34.3/24 | Bali |
| Surabaya | Loopback0 | 3.3.3.3/32 | — |
| Bali | FastEthernet0/0 | 34.34.34.4/24 | Surabaya |
| Bali | Loopback0 | 4.4.4.4/32 | — |

## Persiapan: konfigurasi alamat dasar

Kerjakan bagian ini sekali di awal. Seluruh lab berikutnya bertumpu padanya.

**Router Jakarta**

```cisco
enable
configure terminal
hostname Jakarta
interface FastEthernet0/0
 ip address 12.12.12.1 255.255.255.0
 no shutdown
 exit
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 exit
end
write memory
```

**Router Semarang**

```cisco
enable
configure terminal
hostname Semarang
interface FastEthernet0/0
 ip address 12.12.12.2 255.255.255.0
 no shutdown
 exit
interface FastEthernet0/1
 ip address 23.23.23.2 255.255.255.0
 no shutdown
 exit
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
 exit
end
write memory
```

**Router Surabaya**

```cisco
enable
configure terminal
hostname Surabaya
interface FastEthernet0/0
 ip address 23.23.23.3 255.255.255.0
 no shutdown
 exit
interface FastEthernet0/1
 ip address 34.34.34.3 255.255.255.0
 no shutdown
 exit
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 exit
end
write memory
```

**Router Bali**

```cisco
enable
configure terminal
hostname Bali
interface FastEthernet0/0
 ip address 34.34.34.4 255.255.255.0
 no shutdown
 exit
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
 exit
end
write memory
```

> ### REVISI — Penambahan antarmuka LAN yang sebelumnya tidak ada
>
> Modul asli meminta Jakarta mengumumkan 192.168.10.0 dan Bali mengumumkan 192.168.20.0, padahal kedua jaringan itu tidak pernah dibuat pada bagian persiapan.
>
> Perintah `network` pada BGP hanya mengumumkan jaringan yang sudah ada di tabel routing. Tanpa antarmuka yang memilikinya, kedua baris itu tidak menghasilkan apa pun.
>
> Pada berkas ini ditambahkan antarmuka LAN pada Jakarta dan Bali beserta satu PC di masing-masing sisi, sesuai gambar pada modul asli.

## Tujuan

- Mengonfigurasi BGP antara empat Autonomous System yang berbeda.
- Membaca tabel BGP dan mengenali jalur terbaik beserta daftar AS yang dilewatinya.
- Menjelaskan perbedaan peran BGP dengan protokol routing di dalam satu AS.

## Persiapan tambahan

1. Hubungkan satu PC ke FastEthernet0/1 Router Jakarta.
2. Hubungkan satu PC ke FastEthernet0/1 Router Bali.
3. Beri alamat pada kedua PC sesuai tabel di bawah melalui tab **Desktop** lalu **IP Configuration**.

| Perangkat | Antarmuka | Alamat IP | Gateway | Nomor AS |
|---|---|---|---|---|
| Jakarta | Fa0/1 | 192.168.10.1/24 | — | 1 |
| PC-Jakarta | — | 192.168.10.2/24 | 192.168.10.1 | — |
| Semarang | — | — | — | 2 |
| Surabaya | — | — | — | 3 |
| Bali | Fa0/1 | 192.168.20.1/24 | — | 4 |
| PC-Bali | — | 192.168.20.2/24 | 192.168.20.1 | — |

## Langkah kerja

**Router Jakarta, AS 1**

```cisco
enable
configure terminal
interface FastEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
router bgp 1
 neighbor 12.12.12.2 remote-as 2
 network 1.1.1.1 mask 255.255.255.255
 network 12.12.12.0 mask 255.255.255.0
 network 192.168.10.0 mask 255.255.255.0
 exit
end
write memory
```

**Router Semarang, AS 2**

```cisco
enable
configure terminal
router bgp 2
 neighbor 12.12.12.1 remote-as 1
 neighbor 23.23.23.3 remote-as 3
 network 2.2.2.2 mask 255.255.255.255
 network 12.12.12.0 mask 255.255.255.0
 network 23.23.23.0 mask 255.255.255.0
 exit
end
write memory
```

**Router Surabaya, AS 3**

```cisco
enable
configure terminal
router bgp 3
 neighbor 23.23.23.2 remote-as 2
 neighbor 34.34.34.4 remote-as 4
 network 3.3.3.3 mask 255.255.255.255
 network 23.23.23.0 mask 255.255.255.0
 network 34.34.34.0 mask 255.255.255.0
 exit
end
write memory
```

**Router Bali, AS 4**

```cisco
enable
configure terminal
interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
router bgp 4
 neighbor 34.34.34.3 remote-as 3
 network 4.4.4.4 mask 255.255.255.255
 network 34.34.34.0 mask 255.255.255.0
 network 192.168.20.0 mask 255.255.255.0
 exit
end
write memory
```

## Verifikasi

1. Jalankan `show ip bgp summary` pada tiap router. Kolom State harus menunjukkan angka, bukan tulisan Idle atau Active.
2. Jalankan `show ip bgp` pada Semarang untuk melihat isi tabel BGP.
3. Jalankan `show ip route bgp` pada Semarang. Rute dari BGP berawalan huruf `B`.
4. Lakukan ping dari PC-Jakarta ke 192.168.20.2. Inilah bukti akhir bahwa BGP berhasil.

```text
Semarang# show ip bgp
   Network            Next Hop        Metric LocPrf Weight Path
*> 1.1.1.1/32         12.12.12.1           0      0      0 1 i
*> 2.2.2.2/32         0.0.0.0              0      0  32768 i
*> 3.3.3.3/32         23.23.23.3           0      0      0 3 i
*> 4.4.4.4/32         23.23.23.3           0      0      0 3 4 i
*> 12.12.12.0/24      0.0.0.0              0      0  32768 i
*> 23.23.23.0/24      0.0.0.0              0      0  32768 i
*> 34.34.34.0/24      23.23.23.3           0      0      0 3 i
*> 192.168.10.0/24    12.12.12.1           0      0      0 1 i
*> 192.168.20.0/24    23.23.23.3           0      0      0 3 4 i
```

| Bagian Keluaran | Arti |
|---|---|
| Tanda bintang | Rute sah dan dapat dipakai |
| Tanda lebih besar | Rute terbaik yang dipilih dan dimasukkan ke tabel routing |
| Kolom Path berisi `3 4` | Untuk mencapai tujuan itu, paket melewati AS 3 lalu AS 4 |
| `0.0.0.0` pada Next Hop | Jaringan itu milik router ini sendiri |
| Huruf `i` di ujung | Rute berasal dari perintah `network`, bukan dari redistribusi |

> ### CATATAN — Mengapa daftar AS penting
>
> Daftar pada kolom Path adalah dasar BGP mencegah lingkaran. Bila sebuah router menerima rute yang daftarnya sudah memuat nomor AS miliknya sendiri, rute itu ditolak. Mekanisme ini jauh lebih sederhana daripada perhitungan topologi yang dipakai OSPF, dan itulah yang memungkinkan BGP bekerja pada skala internet.
