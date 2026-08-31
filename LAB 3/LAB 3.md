# LAB 3 — Routing Dinamis OSPF Multi Area

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

## Tujuan

- Mengonfigurasi OSPF dengan lebih dari satu area.
- Menjelaskan aturan yang mengikat hubungan antararea dengan area backbone.
- Membedakan rute intra area dan rute antararea pada tabel routing.

> ### REVISI — Perbaikan penempatan area pada Router Bali
>
> Modul asli menempatkan loopback Bali 4.4.4.4 pada area 0, sedangkan satu-satunya antarmuka Bali yang lain berada di area 10. Rancangan itu melanggar aturan OSPF: setiap area harus tersambung ke area 0, dan area 0 sendiri harus menyambung tanpa terputus.
>
> Akibatnya Bali memiliki potongan area 0 yang terpisah dari area 0 di Jakarta dan Semarang. Pada Packet Tracer hal ini sering tetap tampak berjalan, tetapi pada perangkat sungguhan menuntut virtual link dan termasuk kesalahan rancangan.
>
> Pada berkas ini loopback Bali dipindahkan ke area 10.

## Pembagian area

| Router | Antarmuka | Jaringan | Area | Peran |
|---|---|---|---|---|
| Jakarta | Fa0/0 | 12.12.12.0/24 | 0 | Router internal area 0 |
| Jakarta | Lo0 | 1.1.1.1/32 | 0 | — |
| Semarang | Fa0/0 | 12.12.12.0/24 | 0 | Area Border Router |
| Semarang | Fa0/1 | 23.23.23.0/24 | 10 | — |
| Semarang | Lo0 | 2.2.2.2/32 | 10 | — |
| Surabaya | Fa0/0 | 23.23.23.0/24 | 10 | Router internal area 10 |
| Surabaya | Fa0/1 | 34.34.34.0/24 | 10 | — |
| Surabaya | Lo0 | 3.3.3.3/32 | 10 | — |
| Bali | Fa0/0 | 34.34.34.0/24 | 10 | Router internal area 10 |
| Bali | Lo0 | 4.4.4.4/32 | 10 | Diperbaiki dari area 0 |

Semarang menjadi satu-satunya Area Border Router karena hanya router itu yang memiliki antarmuka pada dua area sekaligus. Seluruh lalu lintas antararea melewatinya.

## Langkah kerja

**Router Jakarta**

```cisco
enable
configure terminal
router ospf 10
 router-id 1.1.1.1
 network 12.12.12.0 0.0.0.255 area 0
 network 1.1.1.1 0.0.0.0 area 0
 exit
end
write memory
```

**Router Semarang**

```cisco
enable
configure terminal
router ospf 10
 router-id 2.2.2.2
 network 12.12.12.0 0.0.0.255 area 0
 network 23.23.23.0 0.0.0.255 area 10
 network 2.2.2.2 0.0.0.0 area 10
 exit
end
write memory
```

**Router Surabaya**

```cisco
enable
configure terminal
router ospf 10
 router-id 3.3.3.3
 network 23.23.23.0 0.0.0.255 area 10
 network 34.34.34.0 0.0.0.255 area 10
 network 3.3.3.3 0.0.0.0 area 10
 exit
end
write memory
```

**Router Bali**

```cisco
enable
configure terminal
router ospf 10
 router-id 4.4.4.4
 network 34.34.34.0 0.0.0.255 area 10
 network 4.4.4.4 0.0.0.0 area 10
 exit
end
write memory
```

> ### CATATAN — Penambahan router-id
>
> Baris `router-id` ditambahkan pada berkas revisi ini. Tanpa penetapan manual, OSPF memilih sendiri identitasnya dari alamat antarmuka tertinggi. Bila antarmuka itu kelak padam, identitas router berubah dan hubungan tetangga yang sudah terbentuk dapat terputus. Menetapkannya secara manual membuat identitas tetap dan memudahkan pembacaan hasil verifikasi.

## Verifikasi

1. Jalankan `show ip ospf neighbor` pada tiap router. Keadaan tiap tetangga harus FULL.
2. Jalankan `show ip ospf interface` untuk memastikan tiap antarmuka berada pada area yang benar.
3. Jalankan `show ip route ospf` pada Jakarta.
4. Lakukan ping dari Jakarta ke 4.4.4.4.

```text
Jakarta# show ip route ospf
O IA    2.2.2.2/32 [110/2] via 12.12.12.2, FastEthernet0/0
O IA    3.3.3.3/32 [110/3] via 12.12.12.2, FastEthernet0/0
O IA    4.4.4.4/32 [110/4] via 12.12.12.2, FastEthernet0/0
O IA    23.23.23.0/24 [110/2] via 12.12.12.2, FastEthernet0/0
O IA    34.34.34.0/24 [110/3] via 12.12.12.2, FastEthernet0/0
```

| Penanda | Arti | Muncul di sini karena |
|---|---|---|
| `O` | Rute dari dalam area yang sama | Tidak muncul pada Jakarta, sebab seluruh tujuan berada di area 10 |
| `O IA` | Rute antararea | Seluruh tujuan dipelajari melalui Semarang sebagai Area Border Router |
| `110` | Administrative distance OSPF | Nilai tetap bagi seluruh rute OSPF |
| angka kedua | Cost, dihitung dari lebar pita | Bertambah setiap melewati satu antarmuka |
