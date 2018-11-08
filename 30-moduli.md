# 3. Moduli

## 3.1 Komande za rad sa modulima

### 3.1.1 Sistemske komande

- Ispis kernel log-a:  
`dmesg`
- Ispis kernel log-a sa promenama u realnom vremenu:  
`dmesg -wH`
- Listanje svih učitanih modula:  
`lsmod`
- Učitavanje novog modula u kernel:  
`insmod <ime_modula.ko>`
- Brisanje učitanog modula
`rmsmod <ime_modula>`
- Kreiranje fajla za komunikaciju sa modulom:  
`mknod <putanja/do/fajla> <tip> <major broj> <minor broj>`  
tip uređaja može biti **c** ili **b**

*Na većini operativnih sistema neke od ovih komandi može da radi samo **root** korisnik*

### 3.1.2 Komande u programskom jeziku C
Svaki modul mora imati funkciju za učitavanje i funkciju za brisanje, koje imaju sledeći oblik:
- Funkcija za učitavanje modula:
```c
static int func_init(void)
{
  // telo funkcije
  // funkcija vraca 0 ako se uspesno izvrsila
  return 0;
}
```
- Funkcija za brisanje modula:
```c
static void func_exit(void)
{
  // telo funkcije
  // ova funkcija treba da oslobodi sve resurse koje je modul zauzeo u toku svog izvršavanja
}

```
- Na kraju je potrebno iskoristiti makroe **module_init** i **module_exit**, kako bi se ove funkcije pravilno vezale za date događaje
```c
module_init(func_init);
module_exit(func_exit);
```

## 3.2 Komunikacija sa modulom

### 3.2.1 C funkije i *fops* struktura

### 3.3.2 Komunikacija iz operativnog sistema
Najprostiji način da se čita/piše direkno u modul je korišćenjem komandi **cat** i **echo**.

Za pisanje u modul:  
`echo "<podaci>" > <putanja do uređaja>`  
Za čitanje modula:  
`cat <putanja do uređaja>`

