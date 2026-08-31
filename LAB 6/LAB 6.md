TOPOLOGI UTAMA DAN RENCANA PENGALAMATAN
Empat router disusun berderet. Setiap router diberi antarmuka loopback sebagai penanda identitas yang akan diumumkan oleh protokol routing.
    Jakarta            Semarang            Surabaya            Bali
   lo0 1.1.1.1        lo0 2.2.2.2         lo0 3.3.3.3        lo0 4.4.4.4
       |                 |    |               |    |             |
     fa0/0  12.12.12.0  fa0/0 fa0/1 23.23.23.0 fa0/0 fa0/1 34.34.34.0 fa0/0
       +-----------------+    +---------------+    +-------------+
           .1      .2            .2      .3           .3      .4

Router	Antarmuka	Alamat IP	Terhubung ke
Jakarta	FastEthernet0/0	12.12.12.1/24	Semarang
Jakarta	Loopback0	1.1.1.1/32	—
Semarang	FastEthernet0/0	12.12.12.2/24	Jakarta
Semarang	FastEthernet0/1	23.23.23.2/24	Surabaya
Semarang	Loopback0	2.2.2.2/32	—
Surabaya	FastEthernet0/0	23.23.23.3/24	Semarang
Surabaya	FastEthernet0/1	34.34.34.3/24	Bali
Surabaya	Loopback0	3.3.3.3/32	—
Bali	FastEthernet0/0	34.34.34.4/24	Surabaya
Bali	Loopback0	4.4.4.4/32	—

PERSIAPAN: KONFIGURASI ALAMAT DASAR
Kerjakan bagian ini sekali di awal. Seluruh lab berikutnya bertumpu padanya.
Router Jakarta
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

Router Semarang
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

Router Surabaya
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

Router Bali
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

LAB 6
ACCESS LIST STANDAR DAN EXTENDED
Alokasi waktu: 3 jam Pelajaran	Bentuk: praktik terbimbing lalu mandiri


REVISI —  Penambahan topologi yang sebelumnya tidak disediakan
Modul asli memakai topologi yang sama sekali berbeda dari empat lab sebelumnya, yaitu dua router dengan PC dan server, tetapi tidak menjelaskan cara membangunnya. Murid diberi perintah access list tanpa mengetahui perangkat apa yang harus disiapkan.

Pada berkas ini topologi tersebut dibangun dari awal, lengkap dengan pengalamatan dan rute statis agar kedua sisi saling terhubung sebelum access list diterapkan.

Tujuan
•   Membangun topologi dua router dengan PC dan server.
•   Menerapkan access list standar untuk memblokir seluruh lalu lintas dari satu jaringan.
•   Menerapkan access list extended untuk memblokir satu layanan saja.
•   Menjelaskan mengapa keduanya dipasang pada tempat yang berbeda.
Topologi
   PC-Klien              R1                    R2              Server
 10.10.10.10/24    fa0/0      fa0/1      fa0/0      fa0/1   20.20.20.2/24
      |            .1           .1        .2          .1         |
      +------------+   10.10.10.0  192.168.99.0  20.20.20.0 -----+
                          /24          /24           /24

Perangkat	Antarmuka	Alamat IP	Gateway
R1	Fa0/0	10.10.10.1/24	—
R1	Fa0/1	192.168.99.1/24	—
R2	Fa0/0	192.168.99.2/24	—
R2	Fa0/1	20.20.20.1/24	—
PC-Klien	—	10.10.10.10/24	10.10.10.1
Server	—	20.20.20.2/24	20.20.20.1

Tahap 1: Membangun konektivitas
Access list hanya dapat diuji bila kedua sisi sudah saling terhubung. Kerjakan tahap ini sampai tuntas dan buktikan berhasil, baru lanjutkan ke tahap berikutnya.
Router R1
Enable
configure terminal
hostname R1
interface FastEthernet0/0
 ip address 10.10.10.1 255.255.255.0
 no shutdown
 exit
interface FastEthernet0/1
 ip address 192.168.99.1 255.255.255.0
 no shutdown
 exit
ip route 20.20.20.0 255.255.255.0 192.168.99.2
end
write memory

Router R2
Enable
configure terminal
hostname R2
interface FastEthernet0/0
 ip address 192.168.99.2 255.255.255.0
 no shutdown
 exit
interface FastEthernet0/1
 ip address 20.20.20.1 255.255.255.0
 no shutdown
 exit
ip route 10.10.10.0 255.255.255.0 192.168.99.1
end
write memory

1.   Atur alamat PC-Klien dan Server melalui tab Desktop lalu IP Configuration sesuai tabel.
2.   Pada Server, buka tab Services lalu HTTP dan pastikan layanannya berstatus On.
3.   Dari PC-Klien lakukan ping 20.20.20.2. Harus berhasil.
4.   Dari PC-Klien buka tab Desktop lalu Web Browser, ketik 20.20.20.2, lalu tekan Go. Halaman harus terbuka.

PERHATIAN —  Jangan lanjut sebelum tahap ini berhasil
Bila ping atau halaman web belum berhasil, access list yang dipasang kemudian tidak dapat diuji karena tidak diketahui apakah yang memblokir adalah access list atau kesalahan konektivitas.

Tahap 2: Access list standar
Access list standar hanya dapat menyaring berdasarkan alamat sumber. Karena itu ia dipasang sedekat mungkin dengan tujuan. Bila dipasang dekat sumber, seluruh lalu lintas dari jaringan itu akan terblokir ke mana pun, bukan hanya ke server.
Router R2, dipasang pada antarmuka ke arah Server
Enable
configure terminal
access-list 1 deny 10.10.10.0 0.0.0.255
access-list 1 permit any
interface FastEthernet0/1
 ip access-group 1 out
 exit
end
write memory

1.   Dari PC-Klien lakukan ping 20.20.20.2. Kini harus gagal.
2.   Jalankan show access-lists pada R2 dan amati bertambahnya angka pencocokan pada baris deny.
3.   Jalankan show ip interface FastEthernet0/1 pada R2 untuk memastikan access list benar terpasang.
R2# show access-lists
Standard IP access list 1
    10 deny 10.10.10.0 0.0.0.255 (8 match(es))
    20 permit any


PERHATIAN —  Hapus sebelum lanjut ke tahap 3
configure terminal
 interface FastEthernet0/1
  no ip access-group 1 out
  exit
 no access-list 1
 end

Pastikan ping dari PC-Klien ke server kembali berhasil sebelum melanjutkan.

Tahap 3: Access list extended
Access list extended dapat menyaring berdasarkan sumber, tujuan, protokol, dan nomor port. Karena sasarannya sudah tepat, ia dipasang sedekat mungkin dengan sumber agar lalu lintas yang akan diblokir tidak perlu menempuh perjalanan lebih dulu.
Router R1, dipasang pada antarmuka ke arah PC
Enable
configure terminal
access-list 100 deny tcp 10.10.10.0 0.0.0.255 host 20.20.20.2 eq 80
access-list 100 permit ip any any
interface FastEthernet0/0
 ip access-group 100 in
 exit
end
write memory

Pengujian	Dari PC-Klien	Hasil yang diharapkan	Alasan
1	ping 20.20.20.2	Berhasil	ICMP tidak termasuk yang diblokir
2	Buka 20.20.20.2 di peramban	Gagal	TCP port 80 diblokir oleh baris pertama
3	ping 192.168.99.2	Berhasil	Bukan tujuan yang disebut pada aturan


CATATAN —  Mengapa baris permit di akhir wajib ada
Setiap access list memiliki aturan tersembunyi berupa penolakan terhadap segala sesuatu yang tidak disebutkan. Bila baris permit ip any any dihapus, seluruh lalu lintas dari mana pun akan terblokir, bukan hanya port 80 dari jaringan klien.

Cobalah sendiri: hapus baris permit itu, lalu uji ping ke 192.168.99.2. Hasilnya akan gagal. Kembalikan setelah dicoba.

Ringkasan perbandingan
Pembanding	Access List Standar	Access List Extended
Nomor	1 sampai 99	100 sampai 199
Dasar penyaringan	Alamat sumber saja	Sumber, tujuan, protokol, dan port
Tempat pemasangan	Sedekat mungkin dengan tujuan	Sedekat mungkin dengan sumber
Alasan penempatan	Agar tidak memblokir lalu lintas ke tujuan lain	Sasarannya sudah tepat sehingga dapat diblokir sedini mungkin
Contoh pemakaian	Memblokir satu jaringan dari satu server	Memblokir satu layanan pada satu server

 
