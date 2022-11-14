# Jarkom-Modul-3-E05

Kelompok E05

Anggota:  
* Maula Izza Azizi - 5025201104
* Selfira Ayu Sheehan - 5025201174
* Brian Akbar Wicaksana - 5025201207

## 1
> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server (1)

- `WISE` sebagai DNS Server : 
```
apt-get update
apt-get install bind9 -y
```

- `Westalis` Sebagai DCHP
```
apt-get update
apt-get install isc-dhcp-server -y
```

kemudian kita mengedit dhcpd.conf dengan menggunakan 
```
nano /etc/dhcp/dhcpd.conf
```
```
subnet 10.24.1.0 netmask 255.255.255.0 {
    option routers 10.24.1.1;
}
```
kemudian tambahkan `eth0` pada interfaces 
```
nano /etc/default/isc-dhcp-server
```
<img width="749" alt="Screen Shot 2022-11-13 at 19 29 08" src="https://user-images.githubusercontent.com/72302421/201521688-735acbd6-152f-4acb-95a7-e6279d0ece4b.png">

kemudian yang terakhir bisa melakukan restart 
```
service isc-dhcp-server restart
```
<img width="830" alt="Screen Shot 2022-11-13 at 19 32 08" src="https://user-images.githubusercontent.com/72302421/201521801-ed3f32b4-aca9-4f9d-9dd9-8d5bc8c4463a.png">

- `Berlin` sebagai Proxy Server
```
apt-get update
apt-get install squid -y
```
## 2
> dan Ostania sebagai DHCP Relay (2). Loid dan Franky menyusun peta tersebut dengan hati-hati dan teliti.

- `Ostania` Sebagai DHCP Relay
```
apt-get update
apt-get install dhcp-helper -y
dhcp-helper -s 10.24.2.4
```

melakukan forward DHCP Server dari IP `10.24.2.4` 
```
bash .bashrc
```


## 3
> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
ubah semua Client menjadi 
```
auto eth0
iface eth0 inet dhcp
```

> Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155 (3)

```
subnet 10.24.1.0 netmask 255.255.255.0 {
    range 10.24.1.50 10.24.1.88;
    range 10.24.1.120 10.24.1.155;
    option routers 10.24.1.1;
    option broadcast-address 10.24.1.255;
    option domain-name-servers 10.24.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}
```

## 4
> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85 (4)

## 5
> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut. (5)

## 6
> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit. (6)

## 7
> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13 (7).

- Buka dan edit file `/etc/dhcp/dhcpd.conf` di Westalis
```
host Eden {
    hardware ethernet 9e:56:8b:3a:2a:9b;
    fixed-address 10.24.3.13;
}
```
- Kita dapat mencari hardware ethernet dengan menggunakan `ip a` pada Eden dan melihat di baris link/ether.
- Taruh hardware ethernet tersebut pada konfigurasi network Eden.
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 9e:56:8b:3a:2a:9b
```
- Restart service isc-dhcp-server pada Westalis

## Setting Proxy Server

- Jalankan script berikut di Berlint
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get install squid -y
```

## 1
> Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

## 2
> Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

- Untuk menjawab no 1 dan 2, buat file `/etc/squid/allowed-workinghour-sites.acl` yang isinya
```
loid-work.com
franky-work.com
```
- Lalu, buka dan edit file `/etc/squid/squid.conf` di Berlint. Tambahkan:
```
http_port 8080
visible_hostname Berlint

acl WORKING_HOUR time MTWHF 08:00-17:00
acl HOLIDAY time SA
acl WORKING_HOUR_SITES dstdomain "/etc/squid/allowed-workinghour-sites.acl"

http_access allow WORKING_HOUR_SITES WORKING_HOUR
http_access allow !WORKING_HOUR_SITES !WORKING_HOUR
http_access allow !WORKING_HOUR_SITES HOLIDAY
http_access deny all
```
- Jalankan `service squid restart`.

# 3
> Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

- Buka dan edit file `/etc/squid/squid.conf` di Berlint. Tambahkan:
```
acl HTTPS_PORT port 443

http_access deny !HTTPS_PORT
```
- http_access diatas diletakkan sebelum semua http_access lainnya.
- Jalankan `service squid restart`.


# 4
> Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

- Buat file `/etc/squid/acl-bandwidth.conf` yang isinya
```
delay_pools 1
delay_class 1 2
delay_access 1 allow all
delay_parameters 1 none 16000/16000
```
- Lalu, buka dan edit file `/etc/squid/squid.conf` di Berlint. Tambahkan:
```
include /etc/squid/acl-bandwidth.conf
```
- Jalankan `service squid restart`.

# 5
> Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan
kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

- Buka dan edit `/etc/squid/acl-bandwidth.conf`. Tambahkan:
```
acl HOLIDAY time SA

delay_access 1 allow HOLIDAY
delay_access 1 deny all
```
- delay_access yang sebelumnya dihapus dan digantikan dengan delay_access diatas.
- Jalankan `service squid restart`.


