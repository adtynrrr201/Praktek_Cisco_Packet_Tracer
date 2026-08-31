# LAB 1 — Static Route

> **Alokasi waktu:** 2 jam pelajaran · **Bentuk:** praktik terbimbing lalu mandiri

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

- Menuliskan rute statis agar seluruh jaringan pada topologi saling terhubung.
- Membaca tabel routing dan mengenali rute yang berasal dari konfigurasi manual.
- Menjelaskan mengapa tiap router membutuhkan jumlah rute yang berbeda.

## Dasar penulisan

```cisco
ip route [jaringan tujuan] [subnet mask] [alamat router berikutnya]
```

Setiap router hanya perlu diberi rute menuju jaringan yang tidak tersambung langsung kepadanya. Jakarta dan Bali berada di ujung sehingga membutuhkan dua rute. Semarang dan Surabaya berada di tengah sehingga masing-masing hanya membutuhkan satu.

## Langkah kerja

**Router Jakarta**

```cisco
enable
configure terminal
ip route 23.23.23.0 255.255.255.0 12.12.12.2
ip route 34.34.34.0 255.255.255.0 12.12.12.2
end
write memory
```

**Router Semarang**

```cisco
enable
configure terminal
ip route 34.34.34.0 255.255.255.0 23.23.23.3
end
write memory
```

**Router Surabaya**

```cisco
enable
configure terminal
ip route 12.12.12.0 255.255.255.0 23.23.23.2
end
write memory
```

**Router Bali**

```cisco
enable
configure terminal
ip route 23.23.23.0 255.255.255.0 34.34.34.3
ip route 12.12.12.0 255.255.255.0 34.34.34.3
end
write memory
```

## Verifikasi

```cisco
Jakarta# show ip route
```

Keluaran yang diharapkan memuat dua baris berawalan huruf `S`:

```text
C    12.12.12.0/24 is directly connected, FastEthernet0/0
S    23.23.23.0/24 [1/0] via 12.12.12.2
S    34.34.34.0/24 [1/0] via 12.12.12.2
```

| Pengujian | Dari | Ke | Hasil yang diharapkan |
|---|---|---|---|
| 1 | Jakarta | 23.23.23.3 | Berhasil |
| 2 | Jakarta | 34.34.34.4 | Berhasil |
| 3 | Bali | 12.12.12.1 | Berhasil |
| 4 | Jakarta | 4.4.4.4 | Gagal, dan ini wajar |

> ### CATATAN — Mengapa ping ke loopback masih gagal
>
> Rute yang dituliskan pada lab ini hanya mencakup jaringan penghubung antarrouter, yaitu 12.12.12.0, 23.23.23.0, dan 34.34.34.0. Alamat loopback belum dikenalkan kepada router lain. Bila ingin loopback saling terjangkau, tambahkan rute untuk tiap loopback, misalnya pada Jakarta: `ip route 4.4.4.4 255.255.255.255 12.12.12.2`. Pada lab routing dinamis berikutnya, loopback akan diumumkan otomatis sehingga penambahan manual tidak lagi diperlukan.
