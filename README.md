# Jarkom-Modul-2-2025-K45

| Nama                            | NRP        |
| ------------------------------- | ---------- |
| I Dewa Made Satya Raditya       | 5027231051 |
| Made Gede Khrisna Wangsa        | 5027201047 | 

## No.1
**SOAL:** 

Di tepi Beleriand yang porak-poranda, Eonwe merentangkan tiga jalur: Barat untuk Earendil dan Elwing, Timur untuk Círdan, Elrond, Maglor, serta pelabuhan DMZ bagi Sirion, Tirion, Valmar, Lindon, Vingilot. Tetapkan alamat dan default gateway tiap tokoh sesuai glosarium yang sudah diberikan.

**PENGERJAAN:** 

Buat topologi LAN seperti yang diminta soal
<img width="766" height="689" alt="Screenshot 2025-10-13 173507" src="https://github.com/user-attachments/assets/10516a75-cd12-4a0e-aa7b-10917f1eacce" />






## No.2
**SOAL:** Angin dari luar mulai berhembus ketika Eonwe membuka jalan ke awan NAT. Pastikan jalur WAN di router aktif dan NAT meneruskan trafik keluar bagi seluruh alamat internal sehingga host di dalam dapat mencapai layanan di luar menggunakan IP address.

**PENGERJAAN:** Hubungkan router dengan node NAT: 

Berikan router sebuah IP berdasarkan DHCP server pada iface `eth0`; 
```
# DHCP config for eth0
auto eth0  
iface eth0 inet dhcp
#	hostname ervn-debi-new-1
```

setelan router untuk tiap koneksi ke switch
```
# Static config for eth1 
auto eth1
iface eth1 inet static
	address 10.86.1.1
	netmask 255.255.255.0

# Static config for eth2 
auto eth2
iface eth2 inet static
	address 10.86.2.1
	netmask 255.255.255.0

# Static config for eth3
auto eth3
iface eth3 inet static
	address 10.86.3.1
	netmask 255.255.255.0


```

Jalankan iptables agar router sebagai NAT bagi client-clientnya: `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.86.0.0/16`

## No.3
**SOAL:** Kabar dari Barat menyapa Timur. Pastikan kelima klien dapat saling berkomunikasi lintas jalur (routing internal via Eonwe berfungsi), lalu pastikan setiap host non-router menambahkan resolver 192.168.122.1 saat interfacenya aktif agar akses paket dari internet tersedia sejak awal.

**PENGERJAAN:** Tambahkan konfigurasi pada setiap node client berupa: 

```
auto eth0
iface eth0 inet static
	address <prefix>.x.x
	netmask 255.255.255.0
	gateway <prefix>.x.x
	up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

jadi disini, prefix mengikuti setup:
- zona barat menggunakan prefix: 10.86.1.2 (earendil), 10.86.1.3 (elwing) dengan gateway 10.86.1.1 (eonwe)
- zona timur menggunakan prefix: 10.86.2.2 (cirdan), 10.86.2.3 (elrong), 10.86.2.4 (malgor) dengan gateway 10.86.2.1 (eonwe)
- area DMZ menggunakan prefix 10.86.3.2 (sirion), 10.86.3.3 (tirion0, 10.86.3.4 (valmar), 10.86.3.5 (lindon), 10.86.3.6 (vingot) dengan gateway 10.86.3.1 (eonwe)

## No.4
**SOAL:** Para penjaga nama naik ke menara, di Tirion (ns1/master) bangun zona <xxxx>.com sebagai authoritative dengan SOA yang menunjuk ke ns1.<xxxx>.com dan catatan NS untuk ns1.<xxxx>.com dan ns2.<xxxx>.com. Buat A record untuk ns1.<xxxx>.com dan ns2.<xxxx>.com yang mengarah ke alamat Tirion dan Valmar sesuai glosarium, serta A record apex <xxxx>.com yang mengarah ke alamat Sirion (front door), aktifkan notify dan allow-transfer ke Valmar, set forwarders ke 192.168.122.1. Di Valmar (ns2/slave) tarik zona <xxxx>.com dari Tirion dan pastikan menjawab authoritative pada seluruh host non-router ubah urutan resolver menjadi IP dari ns1.<xxxx>.com → ns2.<xxxx>.com → 192.168.122.1. Verifikasi query ke apex dan hostname layanan dalam zona dijawab melalui ns1/ns2.

**PENGERJAAN:** 

dimulai dengan melakukan instalasi bind9 dengan command `apt install bind9 -y`

1. Konfigurasi master dns server node Tirion (ns1/master):

- menambahkan setup sebagai master di zone dalam `/etc/bind/named.conf.local`
```
  zone "k45.com" {
    type master;
    file "/etc/bind/db.k45.com"; 
    // Izinkan transfer dan notifikasi ke Valmar (ns2) 
    allow-transfer { 10.86.3.4; }; // IP Valmar
    also-notify { 10.86.3.4; };    // IP Valmar
};
```
- menambahkan setup forwarders di `/etc/bind/named.conf.options`:
```
options {
    directory "/var/cache/bind";
    forwarders {
        192.168.122.1;
    };
    recursion yes;
    allow-query { any; };
};
```
- menambahkan setup A record di `nano /etc/bind/db.k45.com`
```
$TTL    604800
@       IN      SOA     ns1.k45.com. root.k45.com. (
                              2         ; Serial (Mulai dari 1 atau 2)
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;

@       IN      NS      ns1.k45.com.
@       IN      NS      ns2.k45.com.


@       IN      A       10.86.3.2		; IP Sirion sbg apex

; Record untuk Name Server
ns1     IN      A       10.86.3.3       ; IP Tirion
ns2     IN      A       10.86.3.4       ; IP Valmar
```

- restart bind9 dengan `service named restart`


2. Konfigurasi slave dns pada Valmar
   
- deklarasi valmar sebagai budak (ns2/slave) dg menulis zonefile `nano /etc/bind/named.conf.local`

```
zone "k45.com" {
    type slave;
    file "db.k45.com"; 
    masters { 10.86.3.3; }; // IP Tirion (ns1)
};
```
- restart bind9 valmar dengan `service named restart`

3. tambahkan nameserver baru dengan ip ns1 & ns2 pada konfigurasi setiap node dengan `nano /etc/resolv.conf`
   ```
	nameserver 10.86.3.3 // IP Tirion (ns1)
	nameserver 10.86.3.4 // IP Valmar (ns2)
   ```

## No.5
**SOAL:** “Nama memberi arah,” kata Eonwe. Namai semua tokoh (hostname) sesuai glosarium, eonwe, earendil, elwing, cirdan, elrond, maglor, sirion, tirion, valmar, lindon, vingilot, dan verifikasi bahwa setiap host mengenali dan menggunakan hostname tersebut secara system-wide. Buat setiap domain untuk masing masing node sesuai dengan namanya (contoh: eru.<xxxx>.com) dan assign IP masing-masing juga. Lakukan pengecualian untuk node yang bertanggung jawab atas ns1 dan ns2.

**PENGERJAAN:** 

- Tambahkan A record pada setiap client dengan subdomain yang sesuai dengan nama tiap client, ini dilakukan di Tirion dengan `nano /etc/bind/db.k45.com`

```
earendil   IN   A   10.86.1.2
elwing     IN   A   10.86.1.3
cirdan     IN   A   10.86.2.2
elrond     IN   A   10.86.2.3
maglor     IN   A   10.86.1.4
eonwe      IN   A   10.86.3.1 
```
- restart bind9
- test ping ke host lain dengan nama (misal: ping elrond.k45.com)

## No.6
**SOAL:** "Lonceng Valmar berdentang mengikuti irama Tirion. Pastikan zone transfer berjalan, 
Pastikan Valmar (ns2) telah menerima salinan zona terbaru dari Tirion (ns1). Nilai 
serial SOA di keduanya harus sama"

**PENGERJAAN**:
Pastikan serialnya sama dengan command `dig @localhost k45.com SOA` pada Tirion & Valmar, perhatikan hasil dari `;; ANSWER SECTION:` nya, apabila nomornya sama, maka sudah benar. apabila belum, maka perlu konfigurasi ulang.

## No. 7
**SOAL**: "Peta kota dan pelabuhan dilukis. Sirion sebagai gerbang, Lindon sebagai web statis, Vingilot sebagai web dinamis. Tambahkan pada zona <xxxx>.com A record untuk sirion.<xxxx>.com (IP Sirion), lindon.<xxxx>.com (IP Lindon), dan vingilot.<xxxx>.com (IP Vingilot). Tetapkan CNAME :
- www.<xxxx>.com → sirion.<xxxx>.com, 
- static.<xxxx>.com → lindon.<xxxx>.com, dan 
- app.<xxxx>.com → vingilot.<xxxx>.com."

**PENGERJAAN**:
Di server Tirion (10.86.3.3)
    1. edit file zone `db.k45.com` dengan `nano /etc/bind/db.k45.com`
    2. tambahkan A records dan CNAME untuk DMZ:
	```   
	; A Records untuk Server DMZ 
	sirion     IN   A   10.86.3.2       
	lindon     IN   A   10.86.3.5       
	vingilot   IN   A   10.86.3.6       
	; CNAME (Alias) untuk Layanan
	www        IN   CNAME   sirion.<xxxx>.com.   
	static     IN   CNAME   lindon.<xxxx>.com.   
	app        IN   CNAME   vingilot.<xxxx>.com. 
	```
	3. keluar dan cek file dengan `named-checkzone k45.com /etc/bind/db.k45.com` sampai muncul 
    4. Jika output keduanya OK, restart bind9 dengan `sevice named restart`
	5. masuk ke salah satu node dan ujicoba dengan command:
		- `dig www.k45.com` ---> pastikan www adalah CNAME ke sirion.k45.com, dan sirion.k45.com adalah A record ke 10.86.3.2.
		- `dig static.k45.com` ---> pastikan static adalah CNAME ke lindon.k45.com, dan lindon.k45.com adalah A record ke 10.86.3.5.
		- `dig app.k45.com` ---> pastikan app adalah CNAME ke vingilot.<xxxx>.com, dan vingilot.<xxxx>.com adalah A record ke 10.86.3.6.
## No.8 
**SOAL**: "Setiap jejak harus bisa diikuti. Di Tirion (ns1) deklarasikan satu reverse zone untuk segmen DMZ tempat Sirion, Lindon, Vingilot berada. Di Valmar (ns2) tarik reverse zone tersebut sebagai slave, isi PTR untuk ketiga hostname itu agar pencarian balik IP address mengembalikan hostname yang benar, lalu pastikan query reverse untuk alamat Sirion, Lindon, Vingilot dijawab authoritative."

**PENGERJAAN**: 
1. Di server Tirion:
	- buat deklarasi reverse zone di named.conf.local 
    	```
			zone "3.86.10.in-addr.arpa" {
    	    type master;
    		file "/etc/bind/db.10.86.3"; // Nama file untuk reverse zone
    		allow-transfer { 10.86.3.4; }; // Izinkan Valmar menyalin
    		also-notify { 10.86.3.4; };    // Beri tahu Valmar jika ada update
			};
     	```
     - buat file reverse zone `nano /etc/bind/db.10.86.3` dan isi dengan konfig berikut:
        ```
  			$TTL    604800
		@       IN      SOA     ns1.k45.com. root.k45.com. (
                              1         ; Serial (Mulai dari 1)
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
		
		;  Name Server (NS) 
		@       IN      NS      ns1.k45.com.
		@       IN      NS      ns2.k45.com.

		; PTR Records (IP -> Hostname) 
	
		2       IN      PTR     sirion.k45.com.
		5       IN      PTR     lindon.k45.com.
		6       IN      PTR     vingilot.k45.com.
        ```
	- keluar dan cek file dengan `named-checkzone 3.86.10.in-addr.arpa /etc/bind/db.10.86.3` Jika output keduanya OK, restart bind9 dengan `sevice named restart`
 
 2. Di server Valmar:
    - tambahkan deklarasi slave baru di `nano /etc/bind/named.conf.local` dengan konfig ini:
      ```
		zone "3.86.10.in-addr.arpa" {
    	type slave;
    	file "db.10.86.3"; // BIND akan otomatis membuat file ini
    	masters { 10.86.3.3; }; // Master-nya adalah Tirion
		};
      ```
	- simpan dan restart bind9 dengan `sevice named restart`

3. Verifikasi:
    - masuk ke salah satu node dan ketik command:
      - `dig -x 10.86.3.2` ----> Verifikasi IP Sirion
      - `dig -x 10.86.3.5` ----> Verifikasi IP Lindon
      - `dig -x 10.86.3.6` ----> Verifikasi IP Vingilot
    - pastikan:
       - ada flag `aa` di bagian `;; flags:`
       - di `;; ANSWER SECTION:` harus mengarah ke IP yang benar

## No.9
**SOAL**: "Lampion Lindon dinyalakan. Jalankan web statis pada hostname static.<xxxx>.com dan buka folder arsip /annals/ dengan autoindex (directory listing) sehingga isinya dapat ditelusuri. Akses harus dilakukan melalui hostname, bukan IP."

**PENGERJAAN**:
1. Install Nginx di Lindon `apt update && apt install nginx -y`
2. Buat direktori: 
		 `mkdir -p /var/www/static/annals` 
   ``` 
   #Buat file dummy di dalam /annals/ agar tidak kosong
   echo "The Annals of Aman" > /var/www/static/annals/chapter1.txt
   echo "The Annals of Beleriand" > /var/www/static/annals/chapter2.txt
   echo "<h1>Welcome to Lindon</h1>" > /var/www/static/index.html
   ```
    
3. Konfig nginx:
- Hapus konfigurasi default `rm /etc/nginx/sites-enabled/default`
- Buat file konfigurasi baru `nano /etc/nginx/sites-available/static`
- Isi file konfig:
  ```
	server {
    # Listen di port 80
    listen 80;

    # Tentukan root directory
    root /var/www/static;
    index index.html index.htm;

    # Tentukan hostname yang dilayani 
    server_name static.k45.com;

    # Konfigurasi untuk halaman utama
    location / {
        try_files $uri $uri/ =404;
    }

    # Konfigurasi untuk folder /annals/ 
    location /annals/ {
        # Aktifkan directory listing (autoindex)
        autoindex on;
    }
  }
  ```
4. Aktifkan konfigurasi baru dengan membuat symbolic link: `ln -s /etc/nginx/sites-available/static /etc/nginx/sites-enabled/`
5. Restart Nginx
6. Verifikasi dengan:
   - Tes halaman utama: `curl http://static.k45.com` ----> Output yang diharapkan: `<h1>Welcome to Lindon</h1>`
   - Tes directory listing /annals/: `curl http://static.k45.com/annals/` ----> 0utput yang diharapkan: daftar file HTML yang rapi, berisi chapter1.txt dan 	chapter2.txt, yang membuktikan autoindex berfungsi.
   - 
	
    
