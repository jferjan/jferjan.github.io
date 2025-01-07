---
tags:
  - _draft
---

# Ferjanove avtomatične čestitke - avtomatizacija čestitk za rojstni dan (zapis še ni končan - stay tuned)

Saj veš kako to ponavadi poteka. Tvoj koledar, Facebook ali podobna storitev ti pošlje avtomatizirano obvestilo, da ima prijatelj rojstni dan. Ker si dober prijatelj mu pošlješ en izviren "Vse najboljše za rojstni dan!", čemur po navadi sledi en izviren "Hvala!". Prijateljev imaš seveda veliko zato ta korak ponoviš vsaj enkrat na teden. 

Kaj pa če ... bi zadevo avtomatizirali, tako da bi "izvirno" čestitko prejel vsak prijatelj, ki si to resnično želi.

Spoznaj "Ferjanove avtomatične čestitke", storitev, ki prijateljem omogoča, da se naročijo na avtomatizirane čestitke za svoj rojstni dan.

[cestitke.ferjan.net](http://cestitke.ferjan.net)

  >Da ne bo kdo narobe razumel. Svoje prijatelje za rojstni dan raje pokliči/obišči in jim čestitaj iz srca.
  >Pričujoča storitev je namenjena zgolj učenju Google APP scripta. Gre za nekakšen "proof of concept" primer, ki sem ga uporabil spoznavanje osnov Google APP scripta. 

## Kaj potrebujemo

- [x] [Spletni obrazec za prijavo](#spletni-obrazec-za-prijavo) (Google Forms)
- [x] Zbirko prijavljenih uporabnikov (Google Spreadsheets)
- [x] Predloge 
    * [x] Predloga za potrditev prijave na storitev (Google Docs)
    * [x] Predloga za pošiljanje čestitke (Google Docs)
- [x] Skripte 
    * [x] Skripta z API-jem za prijavo/odjavo od storitve (Google Apps Scrip)
    * [x] Skripta za pošiljanje čestitk (Google Apps Scrip)
    * [x] Skripto za "Garbage Collection" (Google Apps Scrip)
- [ ] Opcijsko
    * [ ] promocijski naslov storitve (DNS, preusmeritev)
    * [ ] poštni naslov storitve (Google Workspace)

### Spletni obrazec za prijavo

[Spletni obrazec](https://docs.google.com/forms/d/e/1FAIpQLSeDGwYFHfKZaCR3yZq9OfqTnXkaSsmVFsIaJUB_bvB6G-pkCw/viewform) ima 5 obveznih polj:

* Polje za vnos e-mail naslova uporabnika.
* Polje za vnos Imena uporabnika.
* Polje za vnos datuma rojstva uporabnika.
* Polje za soglasje k obdelavi osebnih podatkov (GDPR pa take fore)
* Polje za soglasje s pogoji uporabe storitve.

### Zbirka prijavljenih uporabnikov

Zbirka prijavljenih uporabnikov je Google Spreadsheets datoteka, ki jo avtomatično ustvari Google Forms.

Preglednica ima sledeče stolpce:

`Časovni žig`

:   Čas oddaje Google Forms obrazca.
    Avtomatično zabeleženo ob dodaji obrazca.

`E-poštni naslov`	

:   E-poštni naslov uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Ime`

:   Ime uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Datum rojstva`	

:   Datum rojstva uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Varstvo osebnih podatkov`	

:   Soglasje z obdelavo osebnih podatkov uporabnika storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

`Pogoji uporabe`	

:   Soglasje s pogoji uporabe storitve.
    Avtomatično zabeleženo ob dodaji obrazca.

Preglednico razširimo in dodamo tri dodatne stolpce

`Poslano`	

:   Časovni žig zadnjega pošiljanja čestite
    Zabeleži skripta za pošiljanje čestitk.

`formId`	

:   ID številka obrazca za prijavo na storitev.
    Zabeleži skripta za ...

`ActiveStatus`

:   Polje v katerem beležimo, če je posamezna prijava aktivirana, t.j. je uporabnik potrdil lastništvo vnešenega e-mail naslova.
    Zabeleženo ob potrditvi prijave s strani Skripte za potrditev prijave.


...to be continued