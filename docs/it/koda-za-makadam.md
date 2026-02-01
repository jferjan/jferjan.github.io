---
tags:
  - IT
---

# Koda za makadam: Kako sem z AI in odprtimi podatki našel lokacijo za učenje vožnje avtomobila

Vsak starš, ki želi otroka naučiti osnov vožnje (speljevanje, občutek za sklopko) preden gre v avtošolo, naleti na isto težavo: **Kje?** Parkirišča so polna, ceste so prometne, uradni poligoni pa so pogosto zasedeni ali daleč.

Namesto da bi se vozil naokoli in "na slepo" iskal primeren makadam, sem se odločil za inženirski pristop. Uporabil sem kombinacijo **umetne inteligence** (Gemini), **odprtih podatkov** (OpenStreetMap) in **satelitskih posnetkov** (Google maps).

Tukaj je metoda s katero sem v par minutah našel primerno lokacijo na obrobju Ljubljane.

## 1. Korak: Ideja in strategija (Gemini)

Najprej sem uporabil **[Gemini](https://gemini.google.com/)** kot "brainstorming" partnerja. Gemini je najprej predlagal specifična območja (odmaknjene ceste, industrijske cone), a ob poskusu identifikacije točne lokacije vztrajno ponujal netočne Google Maps lokacije. Ko pa sem vprašal po metodi za identifikacijo primernih makadamov je predlagal uporabo **[OpenStreetMap (OSM)](https://www.openstreetmap.org/)** podatkov, ki vsebujejo atribute o vrsti površine (surface) in tipu ceste.

## 2. Korak: Rudarjenje podatkov (Overpass Turbo)

To je bil ključni trenutek. Namesto ročnega iskanja po zemljevidu sem uporabil orodje **[Overpass Turbo](https://overpass-turbo.eu/)**, ki omogoča izvajanje poizvedb po bazi OSM.

Cilj je bil jasen: najti makadamske poti, ki so:

- [x] Dovolj dolge (da se ne obračaš vsakih 50 metrov).  
- [x] Utrjene (da ne uničiš avta).  
- [x] Servisne ali zapuščene (da ne bo prometa).

Uporabil sem tole specifično kodo, ki mi je filtrirala šum (kratke dovoze in pešpoti) in izrisala samo "zlate" kandidate:

``` cpp
// Določi format izpisa (JSON) in časovno omejitev (30s), da strežnik ne prekine iskanja pri večjih območjih.
[out:json][timeout:30];
(
  // Išče servisne poti, ki so makadamske in daljše od 200m
  way["highway"="service"]["surface"~"gravel|compacted|unpaved"](if:length() > 200)({{bbox}});
  
  // Išče utrjene kmetijske poti (grade1), daljše od 300m
  way["highway"="track"]["tracktype"="grade1"]["surface"~"gravel|compacted"](if:length() > 300)({{bbox}});
  
  // Išče zapuščene ceste
  way["highway"="abandoned"](if:length() > 200)({{bbox}});
);
// Izpiše podatke ('body') skupaj z geometrijo ('geom'), kar omogoči izris linij.
out body geom;

{{style:
/* To prepreči vmesniku, da bi točke (nodes) prikazal kot ikone ali krožce */
node {
  opacity: 0;
  fill-opacity: 0;
}
}}
```

![Overpass Turbo](https://lh3.googleusercontent.com/pw/AP1GczN6gmfGN5BKMu5eIgwMg-bhixUouhTulZut_yTuv1vCYgmclVhnt845U0WX-DqHgqxyyp9JogBtjd05pAOVBnt1NZt7aZhH885_eqDXqSiacq6y2zJMksm7JLuKArYfHP3lOzc1WEvc_WSRUdnJGQP_Ig=w1038-h813-s-no-gm?authuser=0 "Overpass Turbo"){referrerpolicy="no-referrer"}

Rezultat? Zemljevid se je "očistil" in pokazal le nekaj dolgih modrih linij na območju Ljubljane in okolice. Po kratkem pregledu sem se odločil, da podrobneje preverim eno: **Cesta v Gorice**, oz. makadam, ki od te ceste slepo vodi do par hiš.

## 3. Korak: Vizualna verifikacija (Google Maps) 

Podatki so eno, realnost pa drugo. Ko mi je Overpass Turbo pokazal lokacijo, sem jo preveril še na **[Google Maps Satellite view]((https://www.google.com/maps/place/46%C2%B001'05.0%22N+14%C2%B028'10.1%22E))** ter na **Google Maps Street view**.

![Cesta v Gorice](https://lh3.googleusercontent.com/pw/AP1GczPjbLdmRIGkjByFllsji_k-w6M-cXo1LNrWLGg3t7ge8EauTejx9sZtFntv1vzBsbsuIjLr4KIHLd5mEOnqU8czyUucSV4sagyTC3pOevz-g7Uf_F3b-Rp29IoNBnkUECuX732xZNTemPAgI7hWMS4fpg=w1470-h699-s-no-gm?authuser=0 "Cesta v Gorice"){referrerpolicy="no-referrer"}

Posnetki so potrdili, da gre za dovolj širok in utrjen makadam, ki je dovolj odmaknjen od glavnih prometnic. Ulica je slepa in vodi le do par hiš, tukaj skoraj zagotovo ne bo prometa. 

## Rezultat

Odpravila sva se na lokacijo. Izkazalo se je, da je tehnologija delovala brezhibno. Našla sva popolnoma raven in širok makadam, kjer – ironično, a pričakovano – zaradi prometnega znaka za prepoved vožnje ni bilo žive duše. To je ustvarilo tiste redke pogoje popolnega miru, ki so potrebni za prvi stik z avtomobilom brez stresa drugih udeležencev v prometu.

**Nauk:** Včasih najboljših poti ne najdeš z volanom, ampak s pravilno kodo.

---

*Opomba: Zapis prikazuje način iskanja lokacij s pomočjo tehnologije in odprtih podatkov. Pri uporabi cest vedno upoštevajte prometno signalizacijo in veljavno zakonodajo.*