---
tags:
  - IT
---

# Tailscale in Headscale: Zasebno omrežje po tvoji meri

>TL:DR V tem zapisu predstavim primer izvedbe VPN omrežja za potrebe upravljanja [homelab](homelab.md) okolja.

## Tailscale 

**Tailscale** ustvari varno in enostavno **VPN mesh omrežje** med vsemi tvojimi napravami, kjerkoli že so. Deluje kot "zero-config" VPN, kar pomeni, da je namestitev in uporaba izjemno preprosta. V osnovi uporablja protokol **WireGuard** in ti omogoča, da do svojih naprav dostopaš, kot da so vse na istem lokalnem omrežju, ne da bi te skrbelo nastavljanje požarnih zidov ali preusmerjanje vrat (port forwarding). Za osebno uporabo je Tailscale brezplačen za določeno število naprav.

## Headscale

Medtem ko je Tailscale odlična rešitev, se njegova brezplačna različica in komercialni paketi upravljajo prek njihovih strežnikov, kar morda ni idealno za vse. Tu pride v igro **Headscale**.

**Headscale** je **odprtokodna, samostojno gostujoča (self-hosted) alternativa** centralnim strežnikom Tailscale. Namesto da bi se tvoje naprave povezovale na Tailscale kontrolne strežnike, jih lahko povežeš na svoj lasten Headscale strežnik. To ti omogoča:

* **Popoln nadzor:** Prevzameš popoln nadzor nad avtentikacijo in avtorizacijo svojih naprav in uporabnikov.
* **Zasebnost podatkov:** Vsi metapodatki o tvojem omrežju ostanejo na tvojem strežniku, namesto na strežnikih tretje osebe.
* **Prilagodljivost:** Za napredne uporabnike Headscale ponuja več možnosti konfiguracije in prilagoditve, ki niso na voljo v standardni Tailscale ponudbi.
* **Neomejeno število naprav/uporabnikov:** Ker gostuješ sam, te ne omejujejo omejitve Tailscale brezplačnega plana glede števila naprav in uporabnikov.

## Zakaj bi jih uporabljal skupaj?

Kombinacija Tailscale (kot odjemalca na tvojih napravah) in Headscale (kot tvojega lastnega kontrolnega strežnika) ti omogoča, da izkoristiš preprostost in robustnost Tailscale tehnologije, hkrati pa ohranjaš **popolno avtonomijo in nadzor nad svojim omrežjem**. To je idealno za uporabnike, ki cenijo zasebnost, želijo imeti popoln nadzor nad infrastrukturo ali potrebujejo večje število naprav, kot jih ponuja brezplačni Tailscale plan.

Če torej želiš enostavno in varno povezljivost med svojimi napravami, a si hkrati želiš popolnega nadzora in neodvisnosti od komercialnih ponudnikov, je kombinacija **Tailscale odjemalcev in lastnega Headscale strežnika** odlična rešitev.


## Arhitektura Headscale/Tailscale homelab omrežja

V nadaljevanju sledi opis konkretne implementacije v [homelab](homelab.md) okolju.

* **Headscale strežnik (Docker):** Nameščen kot Docker vsebnik na našem homelab strežniku, deluje kot centralni kontrolni strežnik za naše Headscale/Tailscale omrežje.
* **Homelab host (Linux CLI):** Naš homelab VM strežnik (ki je hkrati tudi Docker host), opremljen s Tailscale CLI, se poveže na naš Headscale strežnik, ki je nameščen v Docker vsebniku.
* **Oddaljeni Windows klient:** Naš Windows računalnik, z nameščenim Tailscale klientom, se oddaljeno poveže na naš Headscale strežnik.
* **Usmerjevalnik:** Na našem domačem usmerjevalniku je odprt **le port 443 (HTTPS)**. To je ključnega pomena za varnost, saj zmanjšuje število izpostavljenih vrat na minimum. Tailscale (in s tem Headscale) je zasnovan tako, da deluje tudi za požarnim zidom z uporabo tehnik, kot je **hole punching** omogoča neposredne "peer-to-peer" povezave med odjemalci, tudi če so za NAT-om. Port 443 bo uporabljen za začetno avtentikacijo in signalizacijo.

**Namen:** Omogočiti varen, šifriran in direkten dostop do upravljanja homelab okolja z oddaljenega Windows klienta, kot da bi bil fizično prisoten v našem lokalnem omrežju, z minimalno izpostavljenostjo omrežja.

### Kako deluje varen dostop?

Ko se oba Tailscale odjemalca (naš homelab VM strežnik in Windows klient) uspešno povežeta na Headscale strežnik, bosta postala del istega **virtualnega zasebnega omrežja (VPN)**. To pomeni, da bomo lahko z oddaljenega Windows računalnika:

* **Dostopali do drugih naprav in storitev v našem homelab okolju**, kot da bi bili fizično prisotni doma.
* Uporabljali **SSH za upravljanje homelab okolja**, dostopali do **web vmesnikov**, delili datoteke ali dostopali do katere koli druge storitve, ki teče v našem lokalnem omrežju.

Vsa komunikacija med odjemalci bo **šifrirana od konca do konca** z uporabo WireGuard protokola, kar zagotavlja visoko raven varnosti. Z minimalnim odprtim portom na usmerjevalniku bo bistveno zmanjšano tveganje, hkrati pa bomo ohranili popoln in varen dostop do homelab okolja.


## Namestitev Headscale

Pripravimo datotečno strukturo.

``` sh
./headscale
└── config
  └── config.yml
└── data  
└── docker-compose.yml
```

    mkdir /vagrant/headscale

    mkdir /vagrant/headscale/config
    
    mkdir /vagrant/headscale/data

Pripravimo datoteko `docker-compose.yml`.

    wget https://goauthentik.io/docker-compose.yml

> Izhajamo lahko iz uradne datoteke `docker-compose.yml`, ki jo prilagodimo za uporabo s Traefik.

``` yaml
services:
  headscale:
    container_name: headscale
    image: headscale/headscale:0.25.1
    command: serve
    restart: unless-stopped
    volumes:
      - ./config:/etc/headscale/  # Configuration files
      - ./data:/var/lib/headscale # Data persistence
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.headscale.rule=Host(`headscale.lab.ferjan.net`)"
      - "traefik.http.routers.headscale.entrypoints=https"
      - "traefik.http.routers.headscale.tls=true"
      - "traefik.http.services.headscale.loadbalancer.server.port=8080"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

Pripravimo datoteko `config.yml`.

>Izhajamo lahko iz uradne `config-examle.yml` datoteke.

    wget https://raw.githubusercontent.com/juanfont/headscale/refs/heads/main/config-example.yaml -O /vagrant/headscale/config/config.yml

V config datoteki popravimo `server_url`, `base_domain`, `listen_addr`.

``` yaml
server_url: https://headscale.lab.ferjan.net
##
base_domain: headscale.lab.ferjan.net
##
listen_addr: 0.0.0.0:8080
```

Poženemo kontejner.

    docker compose up -d

Ob prvem zagonu se bo v data direktoriju ustvarila **sqlite baza** in datoteka `noise_private.key`.

Preverimo delovanje.

    https://headscale.lab.ferjan.net/windows

Ustvarimo uporabnika `myuser`.

    docker exec headscale headscale users create myuser

Ustvarimo pre-auth ključ.

    docker exec headscale headscale preauthkeys create -e 24h -u myuser

> Headscale uporablja distroless slike in ne vsebuje sh ali bash lupine.
> https://github.com/juanfont/headscale/issues/1800

Preverimo node.

    docker exec headscale headscale nodes list

Preverimo route.

    docker exec headscale headscale routes list

### Tailscale klient

Za povezavo na headscale strežnik je potrebno namestiti [Tailscale klienta](https://tailscale.com/download/).

>V našem je bil izbran Windows klient, ker poteka upravljanje iz prenosnika z nameščenim Windows okoljem. 

Iz Windows se je potrebno ob prvi povezavi povezati preko CLI kot administrator.

    C:\Windows\System32>tailscale up --login-server https://headscale.lab.ferjan.net --authkey <your_auth_key>

>Ko končamo z delom se ne odjavimo ampak le ugasnemo klienta. Ob naslednjem zagonu klienta bomo tako že prijavljeni.
>V primeru, da smo se odjavili moramo ponovno izdati pre-auth ključ in se ponovno prijaviti preko CLI.

Tailscale Linux klienta namestimo tudi v naš homelab VM strežnik (Ubuntu).

    curl -fsSL https://tailscale.com/install.sh | sudo sh

Na homelab VM strežniku se s klientom povežemo na Headscale strežnik, ki teče v Docker vsebniku.

    sudo tailscale up --login-server=https://headscale.lab.ferjan.net --authkey <your_auth_key>

Z nodes list preverimo povezane Tailscale kliente.

    docker exec headscale headscale nodes list

Preverimo [Magic DNS](https://tailscale.com/kb/1081/magicdns).

    nslookup 100.64.0.1
    nslookup lab.headscale.lab.ferjan.net

Dodajanje [subnet routinga](https://tailscale.com/kb/1019/subnets), da omogočimo dostop do celotnega domačega omrežja.

    sudo tailscale set --advertise-routes=192.168.1.0/24
    docker exec headscale headscale routes list
    docker exec headscale headscale routes enable -r 1

Tako, namestitev je končana. V kolikor smo se z našim Windows in Linux klientom uspešno povezali v Tailscale omrežje lahko dostopamo do vsega kar je dosegljivo v našem zasebnem omrežju.