# **LAPORAN RESMI PRAKTIKUM KOMUNIKASI DATA & JARINGAN KOMPUTER MODUL 2**

|Nama                 |NRP               |                                   
|---------------------|----------|
|Agas Ananta Wijaya |5027221004|
---
Berikut adalah Laporan Resmi Praktikum Komunikasi Data & Jaringan Komputer Modul 2 oleh Kelompok IT33.

---

### **SOAL 1**

Untuk membantu pertempuran di Erangel, kamu ditugaskan untuk membuat jaringan komputer yang akan digunakan sebagai alat komunikasi. Sesuaikan rancangan Topologi dengan rancangan dan pembagian yang berada di link yang telah disediakan, dengan ketentuan nodenya sebagai berikut :
DNS Master akan diberi nama Pochinki, sesuai dengan kota tempat dibuatnya server tersebut. Karena ada kemungkinan musuh akan mencoba menyerang Server Utama, maka buatlah DNS Slave Georgopol yang mengarah ke Pochinki. Markas pusat juga meminta dibuatkan tiga Web Server yaitu Severny, Stalber, dan Lipovka. Sedangkan Mylta akan bertindak sebagai Load Balancer untuk server-server tersebut.

**Jawaban Nomor 1**

Diperlukan topologi untuk pengerjaan soal tersebut. Berikut ini adalah topologi yang telah saya buat.



Agar semua node dapat terkonfigurasi internet, maka diperlukan konfigurasi network sebagai berikut:

- Erangel

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.80.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.80.2.1
	netmask 255.255.255.0
```

- Pochinki

```
auto eth0
iface eth0 inet static
	address 10.80.1.2
	netmask 255.255.255.0
	gateway 10.80.1.1
```

- Georgopol

```
auto eth0
iface eth0 inet static
	address 10.80.1.3
	netmask 255.255.255.0
	gateway 10.80.1.1
```

- Stalber

```
auto eth0
iface eth0 inet static
	address 10.80.2.2
	netmask 255.255.255.0
	gateway 10.80.2.1
```
  
- Mylta
  
```
auto eth0
iface eth0 inet static
address 10.80.2.3
netmask 255.255.255.0
gateway 10.80.2.1
```

- Severny

```
auto eth0
iface eth0 inet static
address 10.80.2.4
netmask 255.255.255.0
gateway 10.80.2.1
```

- Lipovka

```
auto eth0
iface eth0 inet static
address 10.80.2.5
netmask 255.255.255.0
gateway 10.80.2.1
```

- GatkaTrenches

```
auto eth0
iface eth0 inet static
address 10.80.1.4
netmask 255.255.255.0
gateway 10.80.1.1
```

- GatkaRadio

```
auto eth0
iface eth0 inet static
	address 10.80.1.5
	netmask 255.255.255.0
	gateway 10.80.1.1
```

Kemudian agar semua node dapat terkoneksi internet dengan tepat, maka harus dijalankan command berikut pada setiap node:
 ```
 echo 'nameserver 192.168.122.1' > /etc/resolv.conf
 ```

Tidak lupa pada node erangel juga harus dijalankan command berikut:
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.80.0.0/16
```

Berikut ini adalah contoh dari salah satu node yang sudah berhasil terkoneksi ke internet:
```
ping google.com
```



### **SOAL 2**

Karena para pasukan membutuhkan koordinasi untuk mengambil airdrop, maka buatlah sebuah domain yang mengarah ke Stalber dengan alamat airdrop.xxxx.com dengan alias www.airdrop.xxxx.com dimana xxxx merupakan kode kelompok. Contoh : airdrop.it01.com

**Jawaban Nomor 2**

Meskipun sebelumnya saya tidak mencoba menggunakan bash, namun untuk kali ini saya mencoba menggunakan bash script agar lebih efektif. 

```bash
#!/bin/bash

apt-get update
apt-get install bind9 -y

mkdir /etc/bind/jarkom

cat <<EOL >> /etc/bind/named.conf.local
zone "airdrop.it33.com" {
    type master;
    file "/etc/bind/jarkom/airdrop.it33.com";
};
EOL

cp /etc/bind/db.local /etc/bind/jarkom/airdrop.it33.com

cat <<EOL > /etc/bind/jarkom/airdrop.it33.com
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     airdrop.it33.com. airdrop.it33.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      airdrop.it33.com.
@       IN      A       10.80.2.2    ; IP Stalber
www     IN      CNAME   airdrop.it33.com.
EOL

service bind9 restart
```
Lalu jalankan command `chmod +x soal2.sh` dan `./soal2.sh`

### **SOAL 3**

Para pasukan juga perlu mengetahui mana titik yang sedang di bombardir artileri, sehingga dibutuhkan domain lain yaitu redzone.xxxx.com dengan alias www.redzone.xxxx.com yang mengarah ke Severny

**Jawaban Nomor 3**

Dengan metode yang kurang lebih sama seperti nomor 2, saya membuka console Pochinki lalu membuat bash seperti berikut:

```bash
cat <<EOL >> /etc/bind/named.conf.local
zone "redzone.it33.com" {
    type master;
    file "/etc/bind/jarkom/redzone.it33.com";
};
EOL

cp /etc/bind/db.local /etc/bind/jarkom/redzone.it33.com

cat <<EOL > /etc/bind/jarkom/redzone.it33.com
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     redzone.it33.com. redzone.it33.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      redzone.it33.com.
@       IN      A       10.80.2.4     ; IP Severny
www     IN      CNAME   redzone.it33.com.
EOL

service bind9 restart
```

### **SOAL 4**
Markas pusat meminta dibuatnya domain khusus untuk menaruh informasi persenjataan dan suplai yang tersebar. Informasi persenjataan dan suplai tersebut mengarah ke Mylta dan domain yang ingin digunakan adalah loot.xxxx.com dengan alias www.loot.xxxx.com

**Jawaban Nomor 4**
Metode nya masih kurang lebih sama seperti sebelumnya, membuka console Pochinki lalu membuat nano soal4.sh

```bash
cat <<EOL >> /etc/bind/named.conf.local
zone "loot.it33.com" {
    type master;
    file "/etc/bind/jarkom/loot.it33.com";
};
EOL

cp /etc/bind/db.local /etc/bind/jarkom/loot.it33.com

cat <<EOL > /etc/bind/jarkom/loot.it33.com
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     loot.it33.com. loot.it33.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loot.it33.com.
@       IN      A       10.80.2.3     ; IP Mylta
www     IN      CNAME   loot.it33.com.
EOL

service bind9 restart
```

###  **SOAL 5**
Pastikan domain-domain tersebut dapat diakses oleh seluruh komputer (client) yang berada di Erangel

**Jawaban Nomor 5**

Untuk memastikan semua client dapat mengakses domain, maka harus dilakukan deklarasi nameserver IP Pochinki dan IP Georgopol pada node client (GatkaTrenches dan GatkaRadio). Kita bisa melakukannya dengan cara berikut:

```
nano /etc/resolv.conf
```

```
nameserver 10.80.1.2  #IP Pochinki
nameserver 10.80.1.3  #IP Georgopol
```

Berikut ini adalah bukti dari keberhasilan `ping airdrop.it33.com` `ping redzone.it33.com` `ping loot.it33.com` pada GatkaTrenches (sebelah kiri) dan GatkaRadio (sebelah kanan):

### **SOAL 6**
Beberapa daerah memiliki keterbatasan yang menyebabkan hanya dapat mengakses domain secara langsung melalui alamat IP domain tersebut. Karena daerah tersebut tidak diketahui secara spesifik, pastikan semua komputer (client) dapat mengakses domain redzone.xxxx.com melalui alamat IP Severny (Notes : menggunakan pointer record)

**Jawaban Nomor 6**

Untuk pengerjaan soal ini, kita akan mencoba menerapkan 
