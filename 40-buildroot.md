# 4. Buildroot

## 4.1 Preuzimanje koda
*Potrebni alati:* `git`

Kod buildroot-a moguće je preuzeti sledećom komandom:  
`git clone git://git.buildroot.net/buildroot`  

## 4.2 Konfigurisanje
*Potrebni alati:* `make` `libqt4-dev`

Podrazumevana minimalistčka podešavanja moguće je postaviti sledećom komandom:  
`make allnoconfig`  
Podrazumevana konfiguracija za RPI2 može se dobiti sledećom komandom:  
`make raspberrypi2_defconfig`  
Nakon toga moguće je ovu konfiguraciju promeniti komandom:  
`make menuconfig` ili `make xconfig`  
Nakon završenog podešvanja potrebno je pokrenuti build proces komandom:  
`make`

Rezultat build-a nalaziće se u folderu **output**

## 4.3 Primena patch-a
*Potrebni alati:* `git`

Ukoliko je potrebno primeniti patch, to se može uraditi sledećom komandom:  
`git apply <putanja do patch-a>`

## 4.4 Dodavanje u PATH

Ovako izgenerisan toolchain ne nalazi se u PATH-u, te ga je potrebno dodati na početak PATH-a, kako bi se mogao prepisati  
**NAPOMENA: Nikada ne prepisivati PATH, već nove lokacije dodati na njegov početak**  
`export PATH=<putanja do toolchain-a>/bin:$PATH`  
Istu stvar treba uraditi i za deljene biblioteke:  
`export LD_LIBRARY_PATH=<putanja do toolchain-a>/lib:$LD_LIBRARY_PATH`
