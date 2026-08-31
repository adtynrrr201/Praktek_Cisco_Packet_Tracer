# LAB 2 — Routing Dinamis EIGRP

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

- Mengonfigurasi EIGRP pada empat router dalam satu Autonomous System.
- Membaca tabel routing dan mengenali rute yang dipelajari EIGRP.
- Menjelaskan asal usul nilai metric EIGRP.

> ### REVISI — Penulisan network memakai wildcard
>
> Modul asli menuliskan `network 1.1.1.1` tanpa wildcard. EIGRP akan memperlakukannya sebagai 1.0.0.0 sehingga cakupannya lebih luas daripada yang dimaksud. Penulisan yang dipakai di sini menyertakan wildcard `0.0.0.0` agar hanya alamat itu yang diumumkan. Keduanya berfungsi, tetapi bentuk dengan wildcard yang dipakai pada ujian sertifikasi.

## Langkah kerja

**Router Jakarta**

```cisco
enable
configure terminal
router eigrp 10
 network 12.12.12.0 0.0.0.255
 network 1.1.1.1 0.0.0.0
 no auto-summary
 exit
end
write memory
```

**Router Semarang**

```cisco
enable
configure terminal
router eigrp 10
 network 12.12.12.0 0.0.0.255
 network 23.23.23.0 0.0.0.255
 network 2.2.2.2 0.0.0.0
 no auto-summary
 exit
end
write memory
```

**Router Surabaya**

```cisco
enable
configure terminal
router eigrp 10
 network 23.23.23.0 0.0.0.255
 network 34.34.34.0 0.0.0.255
 network 3.3.3.3 0.0.0.0
 no auto-summary
 exit
end
write memory
```

**Router Bali**

```cisco
enable
configure terminal
router eigrp 10
 network 34.34.34.0 0.0.0.255
 network 4.4.4.4 0.0.0.0
 no auto-summary
 exit
end
write memory
```

## Verifikasi

1. Jalankan `show ip eigrp neighbors` pada tiap router. Jakarta harus memiliki satu tetangga, Semarang dan Surabaya masing-masing dua, dan Bali satu.
2. Jalankan `show ip route` pada Jakarta. Rute yang dipelajari EIGRP berawalan huruf `D`.
3. Lakukan ping dari Jakarta ke 4.4.4.4. Kini harus berhasil karena loopback ikut diumumkan.
4. Jalankan `show ip route 4.4.4.4` untuk melihat rincian metric dan jalurnya.

```text
Jakarta# show ip route
D    2.2.2.2/32 [90/130816] via 12.12.12.2, FastEthernet0/0
D    3.3.3.3/32 [90/158976] via 12.12.12.2, FastEthernet0/0
D    4.4.4.4/32 [90/161280] via 12.12.12.2, FastEthernet0/0
D    23.23.23.0/24 [90/30720] via 12.12.12.2, FastEthernet0/0
D    34.34.34.0/24 [90/33280] via 12.12.12.2, FastEthernet0/0
```

Angka 90 adalah administrative distance EIGRP. Angka di sebelahnya adalah metric, yang dihitung dari lebar pita paling kecil sepanjang jalur dan jumlah penundaan seluruh antarmuka yang dilewati.

## Nilai bawaan lebar pita dan penundaan

Tabel berikut dikembalikan dari modul asli. Nilai inilah yang dipakai EIGRP untuk menghitung metric.

| Jenis Antarmuka | Lebar Pita (kb/s) | Penundaan (mikrodetik) |
|---|---|---|
| Loopback | 8.000.000 | 5.000 |
| FastEthernet | 100.000 | 100 |
| Ethernet | 10.000 | 1.000 |
| FDDI | 100.000 | 100 |
| Token Ring | 16.000 | 630 |
| Serial | 1.544 | 20.000 |
| ISDN BRI dan PRI | 64 | 20.000 |
| Dialer | 56 | 20.000 |
