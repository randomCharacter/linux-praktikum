# 2. Busybox

## 2.1 Bildovanje busybox-a
*Potrebni alati:* `libncurses-dev`

*NAPOMENA:* potrebno je postaviti promenljive **ARCH** i **CROSS_COMPILE**  
Nakon kopiranja željene konfiguracije (opciono), potrebno je izabrati koji će se moduli prevesti. Sledeća komanda to omogućava:  
`make menuconfig`  
Nakon toga, potrebno je pokrenuti komandu za bildovanje:  
`make install`  
nakon čega će se izgenerisati rfs.

## 2.2 Dodavanje applet-a

- U fajl **include/applets.src.h** dodati:
```
IF_<IME_APPLETA>(APPLET(<ime_appleta>, BB_DIR_USR_BIN, BB_SUID_DROP))
```

- U fajl **include/usage.src.h** dodati opise:
```
#define <ime_appleta>_trivial_usage \
"[param1] [param2] ...“
#define <ime_appleta>_full_usage \
"Opis programa"
```

- U fajl **miscutils/Config.src** dodati konfiguraciju za buildovanje:
```
config <IME_APPLETA>
  bool “<ime_appleta>”
  default n
  depends on LFS
  help
    Opis appleta
    koji se ispisuje
    u meniju
```

- U fajl **miscutils/Kbuild.src** dodati izgenerisani fajl:
```
lib-$(<IME_APPLETA>) += <ime_appleta>.o
```

- Konačno, potrebno je napisati kod appleta, koji će se prevesti i vezati prilikom bildovanja:
```c
#include "busybox.h"

int <ime_appleta>_main(int argc, char **argv)
{
  if (<neispunjen_uslov_pozivanja) {
    bb_show_usage(); // Prikazuje poruku o tome kako se poziva program
  }
  
  /*
    Kod appleta
  */
}
```
Nakon ovoga potrebno je otkačiti applet u menuconfig-u, kako bi se preveo (`make menuconfig`).
Applet će se nalaziti **miscutils** kategoriji.
Postupak je isti i za neku drugu.

## 2.3 Dodavanje potrebnih uređaja


Prvo je potrebno napraviti **dev** folder, sledećom komandom:  
`mkdir <putanja do rfs-a>/dev/`  

### tty uređaji
```
mknod <putanja do rfs-a>/dev/tty1 c 4 1
mknod <putanja do rfs-a>/dev/tty2 c 4 2
mknod <putanja do rfs-a>/dev/tty3 c 4 3
mknod <putanja do rfs-a>/dev/tty4 c 4 4
```

### ttyAMA0
`mknod <putanja do rfs-a>/dev/ttyAMA0 c 204 64`

### console
`mknod <putanja do rfs-a>/dev/console c 5 1`

## 2.4 proc i sys direktorijumi

### 2.4.1 Mountovanje
Za mountovanje **proc** i **sys** direktorijuma koriste se sledeće komande:
```
mount -t sysfs sys sys
mount -t proc proc proc
```

### 2.4.2 Automatizacija procesa
Ovaj proces može se automatizovati, dodavanjem ovih komandi u skriptu koja se pokreće pri pokretanju.

- Kreirati foldere **etc** i **init.d** sledećim komandama:
```
mkdir <putanja do rfs-a>/etc/
mkdir <putanja do rfs-a>/etc/init.d/
```

- u folderu **<putanja do rfs-a>/etc/init.d/** kreirati fajl **rcS** i u njega upisati komande koje je potrebno izvršiti prilikom pokretanja

- kreirati fajl **<putanja do rfs-a>/etc/inittab** i upisati sledeće u njega:
```
::sysinit:/etc/init.d/rcS
ttyAMA0::askfirst:/bin/sh
```
### 2.5 Initramfs
*Potrebni alati:* `libqt4-dev`

Moguće je ovako izgenerisan fajl sistem ubaciti u zImage.

- U folderu sa kodom kernela pokrenuti komandu `make xconfig`.
- Naći opciju **CONFIG_INITRAMFS_SOURCE** i postaviti ga na lokaciju u kojoj je izgenerisan rfs.
- Pokrenuti build proces komandom `make -j4`

*NAPOMENA:* Iz bootargs-a je potrebno izbaciti sve komande koje nisu **rw** i **console**
