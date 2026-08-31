# LAB 4 — OSPF Load Balance dan Failover

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

> ### REVISI — Penambahan jalur kedua
>
> Modul asli meminta murid mengamati pembagian beban, padahal topologinya berupa rantai lurus sehingga hanya ada satu jalur menuju setiap tujuan. Contoh keluaran pada modul menampilkan jalur melalui 16.16.16.6, alamat yang tidak pernah ada pada topologi tersebut.
>
> Pada berkas ini ditambahkan jalur kedua antara Jakarta dan Semarang memakai jaringan 16.16.16.0/24. Dengan dua jalur yang biayanya sama, pembagian beban benar-benar terjadi dan dapat diamati.

## Tujuan

- Membuktikan OSPF membagi beban ketika tersedia dua jalur dengan cost sama.
- Mengubah cost sebuah antarmuka dan mengamati perpindahan jalur.
- Menjelaskan perbedaan antara pembagian beban dan jalur cadangan.

## Persiapan perangkat keras

1. Matikan Router Jakarta melalui tab **Physical**, pasang modul NM-1FE-TX pada slot kosong, lalu nyalakan kembali.
2. Lakukan hal yang sama pada Router Semarang.
3. Hubungkan FastEthernet1/0 Jakarta ke FastEthernet1/0 Semarang.
4. Pastikan kedua router kini memiliki dua tautan menuju satu sama lain.

```text
        Jakarta                          Semarang
          |  fa0/0  12.12.12.0/24  fa0/0  |
          +-------------------------------+
          |  fa1/0  16.16.16.0/24  fa1/0  |
          +-------------------------------+
             dua jalur, cost sama besar
```

## Langkah kerja

**Router Jakarta**

```cisco
enable
configure terminal
interface FastEthernet1/0
 ip address 16.16.16.1 255.255.255.0
 no shutdown
 exit
router ospf 10
 network 16.16.16.0 0.0.0.255 area 0
 exit
end
write memory
```

**Router Semarang**

```cisco
enable
configure terminal
interface FastEthernet1/0
 ip address 16.16.16.2 255.255.255.0
 no shutdown
 exit
router ospf 10
 network 16.16.16.0 0.0.0.255 area 0
 exit
end
write memory
```

## Verifikasi pembagian beban

```cisco
Jakarta# show ip route 4.4.4.4
```

Keluaran yang diharapkan memuat dua baris jalur, keduanya bermetric sama:

```text
Routing entry for 4.4.4.4/32
  Known via "ospf 10", distance 110, metric 4, type inter area
  Routing Descriptor Blocks:
  * 12.12.12.2, from 4.4.4.4, via FastEthernet0/0
      Route metric is 4, traffic share count is 1
    16.16.16.2, from 4.4.4.4, via FastEthernet1/0
      Route metric is 4, traffic share count is 1
```

Kemunculan dua alamat berikutnya dengan traffic share count yang sama menandakan beban dibagi rata di antara kedua jalur.

## Percobaan lanjutan: mengubah jalur

Setelah pembagian beban terbukti, ubah cost salah satu jalur agar OSPF memilih satu jalur saja. Jalur yang tersisa menjadi cadangan yang langsung dipakai bila jalur utama padam.

**Router Jakarta, menaikkan cost jalur kedua**

```cisco
enable
configure terminal
interface FastEthernet1/0
 ip ospf cost 100
 exit
end
```

1. Jalankan kembali `show ip route 4.4.4.4`. Kini hanya tersisa satu jalur, yaitu melalui 12.12.12.2.
2. Matikan jalur utama dengan masuk ke `interface FastEthernet0/0` lalu ketik `shutdown`.
3. Jalankan lagi `show ip route 4.4.4.4` setelah beberapa detik. Jalur berpindah ke 16.16.16.2.
4. Nyalakan kembali FastEthernet0/0 dengan `no shutdown`, lalu amati jalur kembali seperti semula.
5. Kembalikan cost ke keadaan semula dengan `no ip ospf cost` pada FastEthernet1/0.

| Keadaan | Cost fa1/0 | Jalur yang dipakai | Perilaku |
|---|---|---|---|
| Awal | 1 (bawaan) | Keduanya | Pembagian beban |
| Setelah diubah | 100 | fa0/0 saja | fa1/0 menjadi cadangan |
| fa0/0 dimatikan | 100 | fa1/0 | Peralihan otomatis |
| fa0/0 dinyalakan | 100 | fa0/0 | Kembali ke jalur utama |
