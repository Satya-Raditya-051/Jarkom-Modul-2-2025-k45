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

```

```

Konfigurasi master dns server pada node Tirion (ns1/master) dg menambahkan `/etc/bind/named.conf.local`:

```

```

Menetapkan forwarders ke `192.168.122.1` dg menulis `/etc/bind/named.conf.options` pada Tirion (ns1/master)..

```

```

Konfigurasi slave dns pada Valmar (ns2/slave) dg menulis zonefile..

```

```

Menetapkan `/etc/resolv.conf` seperti berikut pada setiap node..

```

```

## No.5
**SOAL:** “Nama memberi arah,” kata Eonwe. Namai semua tokoh (hostname) sesuai glosarium, eonwe, earendil, elwing, cirdan, elrond, maglor, sirion, tirion, valmar, lindon, vingilot, dan verifikasi bahwa setiap host mengenali dan menggunakan hostname tersebut secara system-wide. Buat setiap domain untuk masing masing node sesuai dengan namanya (contoh: eru.<xxxx>.com) dan assign IP masing-masing juga. Lakukan pengecualian untuk node yang bertanggung jawab atas ns1 dan ns2.

**PENGERJAAN:** Tetapkan A record pada setiap client dengan subdomain yang sesuai dengan nama tiap client (eg. lindon dengan lindon.k58.com). Tambahkan ini pada zonefile milik Tirion (ns1/master):

```
...

```

