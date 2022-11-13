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
```

melakukan forward DHCP Server dari IP `10.24.2.4` 
```
dhcp-helper -s 10.24.2.4
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
