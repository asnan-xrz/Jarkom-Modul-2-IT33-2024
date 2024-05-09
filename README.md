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

Untuk pengerjaan soal ini, kita akan mencoba menerapkan D Reverse DNS. Seperti biasa kita akan menggunakan command `nano soal6.sh` dan memasukkan command sebagai berikut:

```bash
#!/bin/bash

cat >> /etc/bind/named.conf.local <<EOL
zone "2.80.10.in-addr.arpa" {
    type master;
    file "/etc/bind/jarkom/2.80.10.in-addr.arpa";
};
EOL

cp /etc/bind/db.local /etc/bind/jarkom/2.80.10.in-addr.arpa

cat >> /etc/bind/jarkom/2.80.10.in-addr.arpa <<EOL
\$TTL    604800
@       IN      SOA     redzone.it33.com. redzone.it33.com. (
                        2024050401      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      redzone.it33.com.
@       IN      A       10.80.2.4     ; IP Severny
www     IN      CNAME   redzone.it33.com.
2.80.10.in-addr.arpa IN NS redzone.it33.com.
4       IN      PTR     redzone.it33.com.
EOL

# Restart server
service bind9 restart
```

Ketik `chmod +x soal6.sh` dan ketik `./soal6.sh`. Pada node client, tambahkan nameserver IP Severny dengan cara sebagai berikut:

```
nano /etc/resolv.conf
```

```
nameserver 10.80.1.2  #IP Pochinki
nameserver 10.80.1.3  #IP Georgopol
nameserver 10.80.2.4  # Tambahkan IP Severny
```

### **SOAL 7**

Akhir-akhir ini seringkali terjadi serangan siber ke DNS Server Utama, sebagai tindakan antisipasi kamu diperintahkan untuk membuat DNS Slave di Georgopol untuk semua domain yang sudah dibuat sebelumnya

**Jawaban Nomor 7**

Buka console Pochinki, lalu ketik `nano soal7Poc.sh` dan masukkan konfigurasi berikut:

```bash
#!/bin/bash

cat <<EOL > /etc/bind/named.conf.local
zone "airdrop.it33.com" {
    type master;
    notify yes;
    also-notify { 10.80.1.3; }; //IP Georgopol
    allow-transfer { 10.80.1.3; }; //IP Georgopol
    file "/etc/bind/jarkom/airdrop.it33.com";
};

zone "redzone.it33.com" {
    type master;
    notify yes;
    also-notify { 10.80.1.3; }; //IP Georgopol
    allow-transfer { 10.80.1.3; }; //IP Georgopol
    file "/etc/bind/jarkom/redzone.it33.com";
};

zone "loot.it33.com" {
    type master;
    notify yes;
    also-notify { 10.80.1.3; }; //IP Georgopol
    allow-transfer { 10.80.1.3; }; //IP Georgopol
    file "/etc/bind/jarkom/loot.it33.com";
};
EOL

service bind9 restart

service bind9 stop
```
Lalu ketik `chmod +x soal7Poc.sh` dan `./soal7Poc.sh`

Kemudian lanjut untuk bagian konfigurasi node Georgopol, buka console Georgopol lalu ketik `nano soal7Geo.sh`.

```bash
#!/bin/bash

# Update package lists and install BIND9
apt-get update
apt-get install bind9 -y

# Configure BIND9 zones
cat <<EOL > /etc/bind/named.conf.local
zone "airdrop.it33.com" {
    type slave;
    masters {10.80.1.2; }; // IP Pochinki
    file "/var/lib/bind/airdrop.it33.com";
};

zone "redzone.it33.com" {
    type slave;
    masters {10.80.1.2; }; // IP Pochinki
    file "/var/lib/bind/redzone.it33.com";
};

zone "loot.it33.com" {
    type slave;
    masters {10.80.1.2; }; // IP Pochinki
    file "/var/lib/bind/loot.it33.com";
};
EOL

# Restart BIND9 service
service bind9 restart

```

Lalu ketik `chmod +x soal7Geo.sh` dan `./soal7Geo.sh`. Testing hanya bisa dilakukan dengan mematikan server Pochinki `service bind9 stop`.

Ketikkan `ping airdrop.it33.com` `ping redzone.it33.com` `ping loot18.it33.com` pada node client untuk testing.

### **SOAL 8**
Kamu juga diperintahkan untuk membuat subdomain khusus melacak airdrop berisi peralatan medis dengan subdomain medkit.airdrop.xxxx.com yang mengarah ke Lipovka

**Jawaban Nomor 8**
Buka console Pochinki lalu ketik `nano soal8.sh` dan masukkan konfigurasi berikut:

```bash
cat <<EOL >/etc/bind/jarkom/airdrop.it33.com
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
@       IN      A       10.80.2.2     ; IP Stalber
www     IN      CNAME   airdrop.it33.com.
medkit  IN      A       10.80.2.5     ; IP Lipovka
@       IN      AAAA    ::1
EOL

service bind9 restart
```
Lalu ketikan `chmod +x soal8.sh` dan `./soal8.sh`

### **SOAL 9**
Terkadang red zone yang pada umumnya di bombardir artileri akan dijatuhi bom oleh pesawat tempur. Untuk melindungi warga, kita diperlukan untuk membuat sistem peringatan air raid dan memasukkannya ke subdomain siren.redzone.xxxx.com dalam folder siren dan pastikan dapat diakses secara mudah dengan menambahkan alias www.siren.redzone.xxxx.com dan mendelegasikan subdomain tersebut ke Georgopol dengan alamat IP menuju radar di Severny

**Jawaban Nomor 9**
Pada dasarnya kita hanya perlu menambahkan subdomain `siren.redzone.it33.com` pada server Pochinki `nano /etc/bind/jarkom/redzone.it33.com`. Pada Pochinki ketikkan `nano soal9Poc.sh` dan masukkan konfigurasi sebagai berikut:

```bash
#!/bin/bash

cat >> /etc/bind/jarkom/redzone.it33.com <<EOL
\$TTL    604800
@       IN      SOA     redzone.it33.com. redzone.it33.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      redzone.it33.com.
@       IN      A       10.80.1.2     ; IP Pochinki
www     IN      CNAME   redzone.it33.com.
@       IN      A       10.80.2.4     ; IP Severny
2.80.10.in-addr.arpa IN NS redzone.it33.com.
4       IN      PTR     redzone.it33.com.
ns1     IN      A       10.80.1.3     ; IP Georgopol
siren   IN      A       ns1'           
EOL

service bind9 restart
```
Lalu ketik `chmod +x soal9Poc.sh` dan `./soal9Poc.sh`

Buka `named.conf.options` dengan command `/etc/bind/named.conf.options` pada Pochinki. Kemudian comment dnssec-validation auto lalu tambahkan baris berikut pada /etc/bind/named.conf.options. `  allow-query{any;};`

Dan seperti biasa lakukan restart bind9 `service bind9 restart`

Tidak lupa pula tambahkan konfigurasi pada DNS Slave Georgopol dengan menambahkan subdomain `siren.redzone.it33.com`. Ketik `nano soal9Geo.sh` dan masukkan konfigurasi sebagai berikut:

```bash
#!/bin/bash

cat <<EOL >> /etc/bind/named.conf.local
zone "siren.redzone.it33.com" {
    type master;
    file "/etc/bind/delegasi/siren.redzone.it33.com";
};
EOL

mkdir -p /etc/bind/delegasi/

cp /etc/bind/db.local /etc/bind/delegasi/siren.redzone.it33.com

cat <<EOL > /etc/bind/delegasi/siren.redzone.it33.com
\$TTL 604800
@       IN      SOA     siren.redzone.it33.com. root.siren.redzone.it33.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
@       IN      NS      siren.redzone.it33.com.
@       IN      A       10.80.2.4	; IP Severny
www     IN      CNAME   siren.redzone.it33.com.
EOL

service bind9 restart

```

Ketik command `chmod +x soal9Geo.sh` dan `./soal9Geo.sh`

Layaknya sebelumnya, buka `named.conf.options` dengan command `/etc/bind/named.conf.options` pada Georgopol. Kemudian comment dnssec-validation auto; dan tambahkan baris berikut pada /etc/bind/named.conf.options. `  allow-query{any;};`
