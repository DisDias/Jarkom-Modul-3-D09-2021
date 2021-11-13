# Jarkom-Modul-3-D09-2021

Nama Anggota | NRP
------------------- | --------------		
Dias Tri Kurniasari | 05111940000035
Nazhwa Ameera H | 05111940000133
Nur Moh. Ihsanuddien | 05111940000142

## Pengaturan Awal 

### Pengaturan Topologi
<img src="img/M03-0.1.png">

### Edit Konfigurasi Network

#### Foosha (DHCP Relay)
``` 
    auto eth0
    iface eth0 inet dhcp

    auto eth1
    iface eth1 inet static
        address 192.196.1.1
        netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
        address 192.196.2.1
        netmask 255.255.255.0

    auto eth3
    iface eth3 inet static
        address 192.196.3.1
        netmask 255.255.255.0  
```

#### EniesLobby (DNS Server)
```
    auto eth0
    iface eth0 inet static
        address 192.196.2.2
        netmask 255.255.255.0
        gateway 192.196.2.1
```

#### Water7 (Proxy Server)
```
   auto eth0
    iface eth0 inet static
        address 192.196.2.3
        netmask 255.255.255.0
        gateway 192.196.2.1 
```

#### Jipangu (DHCP Server)
```
    auto eth0
    iface eth0 inet static
        address 192.196.2.4
        netmask 255.255.255.0
        gateway 192.196.2.1  
```

#### Loguetown (Client)
```
    auto eth0
    iface eth0 inet static
        address 192.196.1.2
        netmask 255.255.255.0
        gateway 192.196.1.1   
```

#### Alabasta (Client)
```
    auto eth0
    iface eth0 inet static
        address 192.196.1.3
        netmask 255.255.255.0
        gateway 192.196.1.1
```

#### TottoLand (Client)
```
    auto eth0
    iface eth0 inet static
        address 192.196.3.2
        netmask 255.255.255.0
        gateway 192.196.3.1
```

#### Skypie (Client)
```
    auto eth0
    iface eth0 inet static
        address 192.196.3.3
        netmask 255.255.255.0
        gateway 192.196.3.1
```

## No 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

Pertama, pada router `Foosha` jalankan perintah `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.196.0.0/16` agar dapat terhubung dengan jaringan luar. Kemudian, pada EniesLobby, Water7, dan Jipanggu jalankan perintah `echo "nameserver 192.196..." > /etc/resolv.conf` yang berguna untuk setting IP DNS agar terhubung dengan jaringan luar.

Setelah itu, melakukan instalasi:
- EnniesLobby
```
    apt-get update
    apt-get install bind9
```

- Jipangu
```
    apt-get update
    apt-get install isc-dhcp-server
```

- Water7
```
apt-get update
apt-get install squid
```

Kemudian, pada `Jipangu` edit file `/etc/default/isc-dhcp-server` dengan menambahkan `eth0` pada `INTERFACES`

<img src="img/M03-1.png">


## No 2
Foosha sebagai DHCP Relay

Pertama lakukan instalasi DHCP Relay dengan penjalankan perintah :
```
apt-get update
apt-get install isc-dhcp-relay
```

Setelah itu, edit file `/etc/default/isc-dhcp-relay` dengan menambahkan IP `Jipangu` pada `SERVER` dan `eth1 eth2 eth3` pada `INTERFACES`.

<img src="img/M03-2.png">


## No 3
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

Pada `Jipangu` tambahkan isi file `/etc/dhcp/dhcpd.conf` dengan perintah:
```
    subnet 192.196.1.0 netmask 255.255.255.0 {
        range 192.196.1.20 192.196.1.99;
        range 192.196.1.150 192.196.1.169;
        option routers 192.196.1.1;
        option broadcast-address 192.196.1.255;
        option domain-name-servers 192.196.2.2;
        default-lease-time 360;
        max-lease-time 7200;
    }
```

Kemudian lakukan perintah `service isc-dhcp-server restart` dan lakukan testing dengan menyalakan `Alabasta` atau `Loguetown` kemudian lakukan perintah `ip a`

<img src="img/M03-3.png">


## No 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

Pada `Jipangu` tambahkan isi file `/etc/dhcp/dhcpd.conf` dengan perintah:
```
    subnet 192.196.3.0 netmask 255.255.255.0 {
        range 192.196.3.30 192.196.3.50;
        option routers 192.196.3.1;
        option broadcast-address 192.196.3.255;
        option domain-name-servers 192.196.2.2;
        default-lease-time 720;
        max-lease-time 7200;
```

Kemudian lakukan perintah `service isc-dhcp-server restart` dan lakukan testing dengan menyalakan `Skypie` atau `Tottoland` kemudian lakukan perintah `ip a`

<img src="img/M03-4.png">


## No 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

Pertama, pada `EniesLobby` edit file `/etc/bind/named.conf.options` sebagai berikut :
```
forwarders {
        "IP nameserver dari Foosha";
    };

    allow-query{any;};
```
Selain itu pada file yang sama lakukan command
```
    // dnssec-validation auto;
```
Kemudian lakukan `service bind9 restart`.

Pada `Jipangu` edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan IP EniesLobby pada `option domain-name-servers` pada `subnet 192.196.1.0` dan `subnet 192.196.3.0`. Untuk testing lakukan `ping google.com` pada Client.

<img src="img/M03-5.png">


## No 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

Pada `Jipangu` edit file `/etc/dhcp/dhcpd.conf` untuk Switch1 atau `subnet 192.196.1.0` tambahkan :
```
    default-lease-time 360;
    max-lease-time 7200;
```

dan untuk Switch3 atau `subnet 192.196.3.0` tambahkan :
```
    default-lease-time 720;
    max-lease-time 7200;
```

<img src="img/M03-6.png">


## No 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

Pertama pada `Jipangu` edit file `/etc/dhcp/dhcpd.conf` tambahkan :
```
`host Skypie {
    hardware ethernet b2:0a:34:19:77:10;
    fixed-address 192.196.3.69;
}
```
Kemudian jalankan perintah `service isc-dhcp-server restart`

Setelah itu, pada Skypie edit file `/etc/network/interfaces` dengan menambahkan `hwaddress ether b2:0a:34:19:77:10`. 

Kemudian restart pada client dan lakukan perintah `ip a`.

<img src="img/M03-7.png">


## No 8
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

## No 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5  dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

## No 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

## No 11
Web server super.franky.yyy.com berada pada node Skypie

## No 12
Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

## No 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

## Error dan Kendala