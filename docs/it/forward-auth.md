---
tags:
  - IT
---

# Vnaprejšnja avtentikacija z Authentikom in Traefikom: Enostavna zaščita spletnih storitev

Ste se kdaj vprašali, kako bi lahko učinkovito zaščitili svoje spletne storitve, kot so Portainer, Gitea ali kakšna druga aplikacija v vašem lokalnem omrežju, ne da bi se morali prijavljati v vsako posebej? S pomočjo **Authentika** in **Traefika** lahko vzpostavite enotno prijavo (Single Sign-On oz. SSO) in centralizirano upravljate dostop do vseh svojih aplikacij. Tudi do takšnih, ki nimajo lastnega mehanizma za avtentikacijo.

> Zapis je nadaljevanje opisa postavitve [homelab](homelab.md) okolja. Na primeru pokaže kako lahko s pomočjo Forward Auth zaščitimo posamezne spletne storitve.

## Zakaj sploh potrebujemo Authentik in Traefik?

Varnost je ključnega pomena, še posebej za aplikacije, ki so morda dostopne tudi z interneta. Namesto da vsaki aplikaciji dodajate ločeno prijavo, Authentik deluje kot osrednja točka za avtentikacijo. Vsak, ki želi dostopati do zaščitene storitve (v spodnjem primeru Portainer), se najprej prijavi v Authentik. Traefik pa nato poskrbi za usmerjanje prometa in deluje kot **povratni proksi**, ki preveri, ali je uporabnik avtenticiran, preden mu odobri dostop. Takšen način avtentikacije se imenuje **vnapejšnja avtentikacija** (Forward Authentication).

Delovni proces je v bistvu takšen:

1.  Uporabnik poskuša dostopati do vaše storitve (npr. `https://portainer.lab.ferjan.net`).

2.  Traefik prestreže zahtevo in jo preusmeri v Authentikov outpost, da preveri, ali je uporabnik prijavljen.

3.  Če uporabnik ni prijavljen, ga Authentik preusmeri na svojo stran za prijavo.

4.  Po uspešni prijavi Authentik uporabnika preusmeri nazaj na spletno storitev.

5.  Traefik zdaj ve, da je uporabnik avtenticiran, in mu omogoči dostop.

## Korak 1: Nastavitve v Authentik Admin vmesniku

Najprej moramo Authentiku povedati, da bomo zaščitili novo storitev. To storimo z nastavitvijo dveh komponent: **Proxy Provider** (ponudnik proksija) in **Application** (aplikacija).

  * **Ustvarjanje Proxy Providerja:**

      * V navigaciji poiščite **Applications \> Providers** in kliknite **Create**.

      * Izberite **Proxy Provider**.

      * **Name:** poimenujte ga na primer `Portainer Forward Auth`.

      * **Type:** izberite `Forward Auth (Single Application)`. To je ključno, saj pove Authentiku, naj avtentikacijo preveri, a usmerjanje prometa prepusti Traefiku.

      * **External Host:** vnesite URL do vaše storitve (npr. `https://portainer.lab.ferjan.net`).


  * **Ustvarjanje aplikacije:**

      * V navigaciji poiščite **Applications \> Applications** in kliknite **Create**.

      * **Name:** `Portainer`.

      * **Provider:** izberite ponudnika, ki ste ga ustvarili v prejšnjem koraku (`Portainer Forward Auth`).

      * **Launch URL (neobvezno):** lahko vnesete URL do storitve (`https://portainer.lab.ferjan.net`), da se prikaže v Authentikovem vmesniku.

  * **Posodobitev Outposta:**

      * Pojdite na **Applications \> Outposts**, poiščite `authentik Embedded Outpost` in ga uredite.

      * V polju **Applications** dodajte `Portainer`. Na ta način se bo outpost zavedal, da mora zaščititi to aplikacijo.

## Korak 2: Definiranje middleware za Traefik

V `docker-compose.yml` datoteki Traefik vsebnika pripravimo oznake (labels) z definicijo middleware.
Traefik routerji, ki bodo uporabljali ta middleware bodo morali zahtevke usmeriti skozi Authentik. 

-----

``` yaml
      # Oznake za Authentik ForwardAuth Middleware
      # Zamenjaj "authentik-server" z imenom docker servisa authentik serverja.
      - "traefik.http.middlewares.authentik.forwardAuth.address=http://authentik-server:9000/outpost.goauthentik.io/auth/traefik"
      - "traefik.http.middlewares.authentik.forwardAuth.trustForwardHeader=true"
      # Authentik lahko po uspešni prijavi aplikaciji vrne željena zaglavja (opcijsko)
      - "traefik.http.middlewares.authentik.forwardAuth.authResponseHeaders=X-authentik-username,X-authentik-groups,X-authentik-email,X-authentik-name,X-authentik-uid"
```

  * `"traefik.http.middlewares.authentik-auth.forwardauth.address=..."`: Ta oznaka definira middleware `authentik-auth` in določa naslov, kam naj Traefik pošlje zahteve za preverjanje avtentikacije – to je Authentikov outpost. Tukaj bodite pozorni, da naslov odraža interni naslov docker servisa authentik strežnika. V našem primeru je to `authentik-server:9000`.

## Korak 3: Dodajanje authentik middleware na Portainer

Zdaj moramo Portainerju povedati, da mora zahtevke, ki prihajajo na `https://portainer.lab.ferjan.net`, usmeriti skozi middleware `authentik`. To storimo z dodajanjem oznak (labels) v `docker-compose.yml` datoteko za Portainer vsebnik.

``` yaml
      # Dodaj authentik middleware na portainer router
      - "traefik.http.routers.portainer.middlewares=authentik"
```
  * `"traefik.http.routers.portainer.middlewares=authentik"`: Ta oznaka je ključna. Pove Traefiku, naj pred dostopom do Portainer servisa uporabi middleware, imenovan `authentik`.

Ko ste dodali te spremembe v `docker-compose.yml`, Portainer vsebnik ponovno zaženite. Zdaj bi moral biti vaš Portainer strežnik zaščiten z Authentikovo enotno prijavo.

Enako postopamo za katerokoli drugo storitev, ki teče za Traefikom.