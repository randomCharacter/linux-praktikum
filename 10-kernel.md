# 1. Kernel

## 1.1 Bildovanje kernela

### 1.1.1 Preuzimanje kernela
*Potrebni alati:* `git`

Kernel za Raspberry pi može se preuzeti sa sledećeg [repozitorijuma](https://github.com/raspberrypi/linux), sledećom komandom:  
`git clone https://github.com/raspberrypi/linux`

### 1.1.2 Primena patch-a
*Potrebni alati:* `patch`

Moguće je primeniti patch na kernel sledećom komandom (potrebno je biti u direktorijumu kernela):  
`patch -p1 < <path/to/patch-x.y.z>`, *gde je **<path/to/patch-x.y.z>** putanja do fajla u kojem se nalazi patch.*  
Ukoliko je potrebno ukoliniti primenuti patch, to se može uraditi sledećom komandom:  
`patch -R -p1 < <path/to/patch-x.y.z>`

### 1.1.3 Prevođenje kernela
*Potrebni alati:* `make`

Pre prevođenja kernela potrebno je postaviti sistemske varijable **ARCH** i **CROSS_COMPILE** u zavisnosti od toga za koju platformu se kernel prevodi.
U slučaju RPI platforme, sledeća komanda to radi:
```
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```
Kako ne bi bilo potrebe kucati ovo pre svakog pokretanja, moguće je dodati ovo u fajl **~/.bashrc**. Na ovaj način ova komanda će se pozvati svaki put kada se otvori novi shel.

### 1.1.4 Konfiguracija kernela
*Potrebni alati:* `libqt4-dev` `g++`

Prilikom prevođenja kernela koristiće se konfiguracija iz fajla **.config**, koji se nalazi u istom direktorijumu kao i **makefile**. Ukoliko ovaj fajl ne postoji koristiće se podrazumevana konfiguracija.
Moguće je izabrati koji će se moduli prevesti pomoću komande `make xconfig`, koja će otvoriti alat za konfiguraciju.

## 1.2 Pokretanje kernela na RPI platformi

### 1.2.1 Konekcija sa U-Boot-om
*Potrebni alati:* `picocom`

Nakon instaliranja picocom komande potrebno je dodati trenutnog korisnika u **dialout** grupu, kako bi imao sve potrebne privilegije za čitanje i pisanje na USB port. Sledeća konda radi to:  
`sudo adduser $USER dialout`

Potrebno je izlogovati se i logovati ponovo da bi ova komanda imala efekta.

Nakon konektovanja Pi-a na računar, komunikacija se ostvaruje komandom:  
`picocom -b 115200 /dev/ttyUSBX`*, gde **ttyUSBX** predstavlja uređaj povezan na Pi (najčešće je to **ttyUSB0**)*

### 1.2.2 TFTP server
*Potrebni alati:* `tftpd-hpa`

Nakon uspešnog bildovanja kernela potrebno je prebaciti fajlove **zImage** i **bcm2709-rpi-2-b.dtb** u **/var/lib/tftpboot**, nakon čega je potrebno restartovati tftp server, sledećom komandom:  
`sudo service tftpd-hpa restart`

Sada je potrebno povući date fajlove iz RPI U-boot-a.

Prvo je potrebno podesiti IP adrese servera i klijenta, kao i eth adresu sledećim komandama
```
setenv ipaddr <IP adresa rpi>
setenv serverip <IP adresa pc>
setenv ethaddr <eth adresa rpi>
```  
Nakon postavljanja env varijabli, moguće je sačuvati ih komandom `saveenv`, kako bi bili dostupni i nakon reboot-a.

### 1.2.3 NFS Kernel server
*Potrebni alati:* `nfs-kernel-server`

Pored kernela, potrebno je imati u root sistem datoteka, koji će kernel koristiti. Za ovo se koristi nfs-kernel-server.  
Prvo je potrebno u datoteku **/etc/exports** dodati sledeću liniju koda  
```
/putanja/do/root/direktorijuma <IP adresa rpi>(rw,no_root_squash,no_subtree_check)
```  
Nakon ovoga je potrebno restartovati server, kako bi se izmene primetile. To radi sledeća komanda:  
`sudo /etc/init.d/nfs-kernel-server restart`  

### 1.2.4 Bootovanje sistema sa Raspberry-ju

Potrebno je povući **zImage** i **bcm2709-rpi-2-b.dtb** u memoriju pre bootovnaja. Sledeće komande ih povlače i bootuju
```
tftp 0x01000000 zImage
tftp 0x02000000 bcm2709-rpi-2-b.dtb
bootz 0x01000000 - 0x02000000
```

U ovom slučaju za RFS biće korišćena SD kartica, međutim moguće je i postaviti komandu koja ovo automatski radi i pritom koristi mrežni RFS.
```
setenv bootargs "root=/dev/nfs rw ip=<IP adresa rpi> console=ttyAMA0,115200 nfsroot=<IP adresa pc>:/putanja/do/nfs/root/direktorijuma"
boot
```

Moguće je komandu sačuvati za narednu upotrebu sledećom komandom:  
`saveenv`  
U slučaju potrebe da se ovo promeni, sledeća komanda to radi:  
`editenv bootargs`
