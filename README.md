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

Pertama, pada router `Foosha` jalankan perintah `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.196.0.0/16` agar dapat terhubung dengan jaringan luar. Kemudian, pada EniesLobby, Water7, dan Jipanggu jalankan perintah `echo "nameserver 192.168.122.1" > /etc/resolv.conf` yang berguna untuk setting IP DNS agar terhubung dengan jaringan luar.

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
Konfigurasikan semua client agar menggunakan IP dari DHCP Server.  

Gunakan perintah `nano /etc/network/interfaces` pada setiap client node.    

Ubah konfigurasi yang ada menjadi seperti di bawah. Contoh di bawah diterapkan pada Client Node Loguetown.  
```
    #auto eth0
	#iface eth0 inet static
	#	address 192.196.1.2
	#	netmask 255.255.255.0
	#	gateway 192.196.1.1
	auto eth0
	iface eth0 inet dhcp
```  
Comment konfigurasi static di semua client node dan uncomment konfigurasi DHCP-nya.  

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

<img src="img/M03-5-1.png">
<img src="img/M03-6.png">


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

Untuk menyelesaikan soal nomor 8, yang pertama kita lakukan adalah membuat domain jualbelikapald09.com dengan port 5000 di EniesLobby.  

Konfigurasi ini pernah dilakukan di modul sebelumnya. Pertama kita mengedit konfigurasi dengan perintah `nano /etc/bind/named.conf.local`. Lalu kita menambahkan script berikut:   

```
`zone "jualbelikapal.d09.com" {
		type master;
		file "/etc/bind/jarkom/jualbelikapal.d09.com";
	};
```   

Buat folder jarkom di dalam /etc/bind dengan perintah `mkdir /etc/bind/jarkom`.   

Copy file db.local ke dalam folder yang baru dibuat `cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapal.d09.com`.  

Lakukan konfigurasi terhadap file db.local yang sudah dicopy. Masukkan konfigurasi berikut.  

```  
`$TTL    604800
@       IN      SOA     jualbelikapal.d09.com. root.jualbelikapal.d09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapal.d09.com.
@       IN      A       192.196.2.3 ; IP Water7
@       IN      AAAA    ::1
```  

Restart service dengan perintah `sevice bind9 restart`.  

Selanjutnya kita akan mengonfigurasi Water7 (Proxy Server).  

Lakukan konfigurasi dnegan perintah `nano /etc/squid/squid.conf`.  

Tambahkan `http_access allow all` pada konfigurasi.  

Restart service dengan perintah `service squid restart`.  

Aktifkan proxy di Client dengan perintah `export http_proxy="http://jualbelikapal.d09.com:5000"`.  

Testing dengan perintah `lynx https://its.ac.id`. 
![image](https://user-images.githubusercontent.com/65032157/141647814-42c7251f-8db7-449a-9e73-d5dd434e0e50.png)
(https://user-images.githubusercontent.com/68385532/141646505-6108f646-2961-4553-b743-fb6c6d23b3ac.PNG)






## No 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5  dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy  

Pada soal nomor 9, kita diminta untuk membuat 2 user dengan nama luffybelikapald09 dan zorobelikapald09 dengan password yang sudah ditentukan menggunakan autentikasi user proxy enkripsi MD51. Untuk melakukannya kita membutuhkan perintah berikut:  
```  
htpasswd -cm /etc/squid/passwd luffybelikapald09
htpasswd -m /etc/squid/passwd zorobelikapald09  
```  
![9-1](https://user-images.githubusercontent.com/68385532/141646506-a84eac8f-f142-4c94-b4f3-2dc78a7aab0d.PNG)  

Edit konfigurasi squid dengan perintah `nano /etc/squid/squid.conf`.  

Tambahkan script seperti di bawah.  
![9-2](https://user-images.githubusercontent.com/68385532/141646507-b1c77181-0246-4602-9f3d-8eb3f899696d.PNG)  

Restart squid dengan perintah `service squid restart`.  
![9-3](https://user-images.githubusercontent.com/68385532/141646508-e5be09ba-110a-4d24-b9cb-cb1a0a45c93e.PNG)  

Lakukan testing dengan perintah `lynx google.com`
![9](https://user-images.githubusercontent.com/68385532/141646505-6108f646-2961-4553-b743-fb6c6d23b3ac.PNG)

## No 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)  

Buat file baru di Water7 dengan perintah berikut `nano /etc/squid/acl.conf`.  

Isikan sesuai gambar di bawah.  
![10-1](https://user-images.githubusercontent.com/68385532/141646509-1ee801e6-a9a7-495a-8858-be0ef7c5ba12.PNG)  

Edit konfigurasi squid dengan perintah `nano /etc/squid/squid.conf`. Samakan konfigurasi dengan gambar.  
![10-2](https://user-images.githubusercontent.com/68385532/141646502-371b22fe-1bdb-469e-8802-03dee2235b0e.PNG)  

Restart squid dengan perintah `service squid restart`.  

Lakukan testing.  


## No 11
Web server super.franky.yyy.com berada pada node Skypie

1. Membuka `EniesLobby` ,kemudian melakukan konfigurasi seperti berikut
![11_1](https://user-images.githubusercontent.com/65032157/141611564-aa388f99-0a8f-4971-8d01-5cfc3176306c.png)

Kemudian membuat dan konfigurasi pada `/etc/bind/kaizoku/super.franky.d09.com`

![11_2](https://user-images.githubusercontent.com/65032157/141611618-621c017d-cca0-4539-a394-bf748f3b5f78.png)

Melakukan `service bind9 restart`

2. Membuka `Skypie` dan menginstall seperti berikut

![11_3](https://user-images.githubusercontent.com/65032157/141611633-8933ac7d-5eeb-43f7-943b-6424ee76dab0.png)

Membuat `mkdir /var/www/super.franky.d09.com`
Mendonwload dan Unzip `super.franky` dan meindahkan ke `/var/www/super.franky.d09.com`
Melakukan copy `cp /etc/bind/db.local /etc/bind/kaizoku/super.franky.d09.com.conf` dan mengisi dengan konfigurasi berikut

![image](https://user-images.githubusercontent.com/65032157/141611789-40ec9ac8-f4a2-423e-8570-53b251bf064f.png)

Melakukan `a2ensite super.franky.d09.com` kemudian `service apache2 restart`

3. Membuka Water7 dan pada file `/etc/squid/squid.conf` menambahkan konfigurasi:
![11_6](https://user-images.githubusercontent.com/65032157/141612063-a1050349-2df4-4f8d-826d-83fd737cac86.png)

menjadi 
![image](https://user-images.githubusercontent.com/65032157/141611992-a7fb13aa-a300-427b-be61-1b41678ad614.png)

Kemudian melakukan `Service squid restart` 

### Testing

Melakukan `lynx google.com` pada Client `LogueTown`

Dan setelah melakukan login akan membuka halaman super.franky.d09.com
![11_9](https://user-images.githubusercontent.com/65032157/141612147-5fdef6bb-8563-45f0-a259-2cb8272b59d8.png)






## No 12
Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps


Dimana untuk mengatur bandwitch dimana agar kecepatan 10 kbps diperoleh konversi ke 1250 byte 
Pada `Water7` dilakukan membuat dan melakukan konfigurasi pada `/etc/squid/acl-bandwidth.conf` dengan menambahkan
```
acl download url_regex -i .jpg$ .png$
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd

acl luffy proxy_auth luffybelikapald09
acl zoro proxy_auth zorobelikapald09

delay_pools 

delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 allow luffy
delay_access 1 deny zoro
delay_access 1 allow download
delay_access 1 deny all
```

Kemudian memasukkan file `/etc/squid/acl-bandwidth.conf ` tersebut ke dalam `/etc/squid/squid.conf`
```
echo '
include /etc/squid/acl/bandwidth.conf
' >> /etc/squid/squid.conf
```

### Testing
![Screenshot (849)](https://user-images.githubusercontent.com/65032157/141643593-e1d4a540-929b-4dd9-abbd-9115d1b28e88.png)


## No 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

Seperti pada `Soal 12` , melakukan penambahan pada file `/etc/squid/acl-bandwidth.conf ` dengan konfigurasi
```
delay_class 2 1
delay_parameters 2 none
delay_access 2 allow zoro
delay_access 2 deny luffy
delay_access 2 deny all 
```
Dimana akan didapatkan akses Zoro tidak mendapat limit bandwitch sehingga ketika melakukan donwload akan langsung berhasil 
### Testing
![Screenshot (848)](https://user-images.githubusercontent.com/65032157/141643641-380c1619-9f5a-40f3-9fd0-9716a374ac34.png)



## Error dan Kendala
