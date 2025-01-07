---
tags:
  - IT
---


# Homelab z Docker, Traefik, Portainer, Authentik

## Uvod

### Namen

-   Vzpostaviti lokalno oklje za testiranje dokeriziranih informacijskih rešitev.
-   Nabiranje izkušenj s Cloudflare, Traefik, Vagrant, Ubuntu, Let's Encrypt, Portainer, Authentik

### Cilji
Cilj je vzpostaviti **homelab** okolje, ki bo v prvi vrsti namenjeno testiranju dokeriziranih informacijskih rešitev oz. aplikacij. Vse aplikacije znotraj homelab okolja bodo od zunaj dosegljive preko ingres kontrolerja **Traefic** preko **HTTPS** protokola. Certifikati bodo od **Let's Encrypt** z vzpostavljeno avtomatizacijo nameščanja/podaljševanja. Pri tem bo uporabljen **Cloudflare API** za **Let'Encrypt DNS challenge**. Upravljanje vsebnikov bo potekalo preko upravljavske konzole **Portainer**. Dostopi do posameznih upravljavskih vmesnikov bodo omogočeni preko OAuth2 servisa **Authentik**. Celotna postavitev bo zaradi lažje prenosljivosti postavljena v **Ubuntu** virtualki s pomočjo **Vagrant** okolja in **Virtualbox** providerja.


### Zahteve
-   [Vagrant](https://www.vagrantup.com/)
-   [Virtualbox](https://www.virtualbox.org/)
-   [Cloudflare](https://dash.cloudflare.com/)

## Namestitev

Opis postopka namestiteve. 

Namestitev je potekala na:
- macOS Big Sur 11.7.10
- Vagrant 2.4.1
- Virtualbox 7.0

Namestitev je potekala znotraj **docker-ce** okolja na virtualki **ubuntu/focal64**. 

Virtualka je bila zagnana s pomočjo Vagrant-a in Virtualboxa.

V virtualki je bil nameščen docker-ce. 

Z docker compose so bili zagnani vsebniki: 
-   Traefik - ingress kontroler, avtomatizacija Let's Encrypt
-   Portainer - GUI za upravljanje vsebnikov
-   Authentik - Oauth2 servis

### Zagon virtualke Ubuntu z Vagrant

S pomočjo Vagranta bomo zagnali virtualko [ubuntu/focal64](https://app.vagrantup.com/ubuntu/boxes/focal64).

Naredi template za `Vagrantfile`.

    vagrant init ubuntu/focal64

Ciljna vsebina datoteke `Vagranfile`:

``` ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "lab.ferjan.net"
  config.vm.network "public_network", ip: "192.168.1.10", bridge: "en0: Wi-Fi (AirPort)"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "8192"
  end
end
```

Nastavimo javno omrežje tipa bridged network. Omrežje bo imelo fiksni IP naslov 192.168.1.10.

> Za bridge uporabimo ime omrežnega vmesnika preko katerega smo povezani. V kolikor ne vemo imena vmesnika lahko v inicialni kongfiguraciji navedbo `, bridge: "en0: Wi-Fi (AirPort)"` izpustimo in nas bo Vagrant pozval k izbiri vmesnika ob vsakokratnem zagonu virtualke. V tem primeru na Vagrant ponudi seznam vseh mrežnih vmesnikov z imeni mrežnih vmesnikov.   

Virtualko zaženemo z:

    vagrant up

Ko je virtualka zagnana se lahko nanjo povežemo preko:

    vagrant ssh

Preko SSH se lahko povežemo tudi tako, da dodamo vagrant ssh-config v konfiguracijo svojega klienta.

    vagrant ssh-config >> ~/.ssh/config

Nato se lahko povežemo z:

    ssh default

> V kolikor nas moti privzeto ime "default" ga lahko spremenimo v konfiguraciji v Vagrantfile.

Tako, povezani bi morali biti v virtualko.

Preverimo stanje omrežnih vmesnikov:

    ip addr

Ugotovimo, da je naš IP naslov `192.168.1.10` na vmesniku `enp0s8`, vagrant pa je dvignil tudi IP naslov v omrežju `10.0.2.0/24` na vmesniku `enp0s3`

Preverimo route:

    sudo apt install net-tools  
    route -n 

Ugotovimo, da je privzeti gateway usmerjen na IP naslov `10.0.2.2` privzetega NAT omrežja na vmesniku `enp0s3`

    vagrant@ubuntu-focal:~$ route -n | grep UG
    0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3

Kot vidimo privzeti gateway ni ustrezen, ker kaže na IP naslov `10.0.2.2` privzetega NAT omrežja na vmesniku `enp0s3`, medtem ko je IP naslov `192.168.1.10` bridge omrežja na vmesniku `enp0s8`. Potrebno bo vpisati novi gateway `192.168.1.1` in pobrisati obstoječega na vmesniku `enp0s3`. 

>Kljub temu, da smo nastavili bridge network bo Vagrant vedno skonfiguriral tudi NAT omrežje, kamor bo privzeto usmerjen tudi privzeti gateway. Gre za privzeto delovanje Vagrant-a, ki pa je lahko problematično v primeru, da poleg privzetega vagrant omrežja dodamo še dodatno omrežje za dostop od zunaj. Rešitev je v spremembi privzetega gateway-a, ki jo dokumentiram v nadaljevanju. Zadeva se v podobni obliki nahaja tudi v uradni dokumentaciji (https://developer.hashicorp.com/vagrant/docs/networking/public_network#default-router), vendar so bili v našem primeru potrebni popravki. 


Spremembo privzetaga gateway-a lahko naredimo preko Vagrant shell provision skripte, ki jo dodamo na konec Vagrantfile datototeke.


``` ruby
  # Change default gateway  (Run Always)
  config.vm.provision "shell",
  run: "always", inline: <<-SHELL
      # set default gateway to 192.168.1.1
      sudo route add default gw 192.168.1.1
      # delete default gateway enp0s3 (could be eth0 in your case)
      eval `route -n | awk '{ if ($8 == "enp0s3" && $2 != "0.0.0.0") print "sudo route del default gw " $2; }'`
  SHELL
```

Vagrant ponovno preženemo in preverimo, če smo uspešno poravili privzeti gateway.  

    vagrant@ubuntu-focal:~$ route -n | grep UG
    0.0.0.0         192.168.1.1     0.0.0.0         UG    0      0        0 enp0s8

### Konfiguracija SSH dostopa

Na svojem klientu naredimo keypair z `ssh-keygen`.
Vsebino javnega ključa ~/.ssh/id_rsa.pub dodamo v virtualko v datoteko `~/.ssh/authorized_keys`

Obstoječi javni ključ iz `~/.ssh/authorized_keys` izbrišemo. Virtualka bo namreč dosegljiva od zunaj in ključ predstavlja nesprejemljivo varnostno tveganje.
Ko izbrišemo obstoječi javni ključ je potrebno v Vagrantfile dodati:

``` ruby
# SSH config
config.ssh.private_key_path = "~/.ssh/id_rsa"
config.ssh.forward_agent = true
```
Brez eksplicitne nastavitve lokacije privatnega ključa nam ne bo deloval `vagrant up`.

### Namestitev Docker-ce

``` bash
    # Update packages
    sudo apt update
    # Install prerequisites
    sudo apt -y install apt-transport-https ca-certificates curl software-properties-common
    # add the GPG key for the official Docker repository
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    # Add the Docker repository to APT sources
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
    # Install Docker
    sudo apt -y install docker-ce
    # Add vagrant user to docker group
    sudo usermod -aG docker vagrant
    # Change current login session to docker group
    newgrp docker
```

Docker je nameščen.  

### DNS oz. hosts

V lokalno hosts datoteko dodamo:

```sh
# lab.ferjan.net
192.168.1.10 traefik.lab.ferjan.net
192.168.1.10 portainer.lab.ferjan.net
192.168.1.10 authentik.lab.ferjan.net
```

V kolikor imamo nameščen DNS strežnik (npr. pi-hole) lahko namesto hosts datoteke vnesemo zapise v DNS. 

DNS zapise lahko vnesemo tudi v javni DNS. V kolikor želimo, da je homelab dosegljiv od zunaj uporabimo javni IP, v nasprotnem primeru pa preslikamo zapise v lokalni IP. 

> V kolikor naslovov ne bomo imeli v DNS se je potrebno zavedati, da nekatere aplikacije potrebujejo razreševanje domenskih naslovov. Pri takšnih vsebnikih lahko preslikave zagotovimo z [extra_hosts](https://github.com/compose-spec/compose-spec/blob/main/spec.md#extra_hosts).

### Docker omrežja

Za proksiranje zahtevkov od Traefic do zalednih storitev bomo potrebovali Docker omrežje, ki smo ga poimenovali "**proxy**".

    docker network create proxy

Za zaledno komunikacijo med vsebniki Authentik bomo potrebovali Docker omrežje, ki smo ga poimenovali "**backend**".

    docker network create backend

## Traefik

Pripravimo datotečno strukturo za Traefik.

``` sh
./traefik
├── data
│   ├── acme.json
│   └── traefik.yml
└── .env
└── cf_api_token.txt
└── docker-compose.yml
```

`acme.json` - tukaj bo Traefik avtomatično odlagal certifikate
`traefik.yml` - konfiguracija Traefik-a 
`.env` - okoljske spremenljivke
`cf_api_token.txt`

### traefik/data/acme.json

Traefik zahteva, da ima datoteka `acme.json` ustrezne pravice.

    chmod 600 acme.json

### traefik/data/traefik.yml

V konfiguraciji Traefika je potrebno za začetek le **popraviti email**. 

Kasneje, ko bomo preverili in potrdili uspešnost avtomatizirane izdaje staging certifikatov, bomo konfiguracijo **caServer** popravili iz **Staging** na **Prod**. Produkcije ne nastavimo predčasno, ker se lahko, v primeru napak, zgodi, da bo zaradi rate-limitinga začasno blokirano izdajanje certifikatov. Ko bomo na tej točki, da bomo popravljali na **Prod** je potrebno pred spremembo počistiti vsebino `acme.json` datoteke, da bo lahko Trefik v njo dodal produkcijski certifikat.

``` yaml
api:
  dashboard: true
  debug: true
entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: ":443"
serversTransport:
  insecureSkipVerify: true
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  # file:
  #   filename: /config.yml
certificatesResolvers:
  cloudflare:
    acme:
      email: example@example.com
      storage: acme.json
      # caServer: https://acme-v02.api.letsencrypt.org/directory # prod (default)
      caServer: https://acme-staging-v02.api.letsencrypt.org/directory # staging
      dnsChallenge:
        provider: cloudflare
        #disablePropagationCheck: true # uncomment this if you have issues pulling certificates through cloudflare, By setting this flag to true disables the need to wait for the propagation of the TXT record to all authoritative name servers.
        #delayBeforeCheck: 60s # uncomment along with disablePropagationCheck if needed to ensure the TXT record is ready before verification is attempted 
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

### traefik/cf_api_token

Naredi token na [Cloudflare User API tokens](https://dash.cloudflare.com/profile/api-tokens) in ga daj v datoteko `traefik/cf_api_token.txt`

-   Token name: **Homelab Traefik**
-   Permissions: **Zone, Zone, Read**   |   **Zone, DNS, Edit**
-   Zone Resources: **Include, Specific zone, ferjan.net**
-   Client IP address filtering: **Is in, *my_public_ip***

več na https://www.youtube.com/watch?v=n1vOfdz5Nm8&t=1076s

### traefik/.env

Potrebujemo paket **apache2-utils**, da bomo lahko uporabili orodje **htpasswd**.

    sudo apt -y install apache2-utils

Ustvari poverilnice in nastavi geslo.
    
    echo $(htpasswd -nB admin) | sed -e s/\\$/\\$\\$/g

Poverilnice vstavi v `.env` datoteko z vsebino:

    TRAEFIK_DASHBOARD_CREDENTIALS=admin:dsfsdfsasf... 


### traefik/docker-compose.yml

Pripravimo datoteko `docker-compose.yml`.

> Image tag nastavimo na [zadnjo stabilno različico](https://github.com/traefik/traefik/releases/latest).

``` yaml
services:
  traefik:
    image: traefik:v3.1.5
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - 80:80
      - 443:443
      # - 443:443/tcp # Uncomment if you want HTTP3
      # - 443:443/udp # Uncomment if you want HTTP3
    environment:
      CF_DNS_API_TOKEN_FILE: /run/secrets/cf_api_token # note using _FILE for docker secrets
      # CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN} # if using .env
      TRAEFIK_DASHBOARD_CREDENTIALS: ${TRAEFIK_DASHBOARD_CREDENTIALS}
    secrets:
      - cf_api_token
    env_file: .env # use .env
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/acme.json:/acme.json
      # - ./data/config.yml:/config.yml:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.rule=Host(`traefik.lab.ferjan.net`)"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=${TRAEFIK_DASHBOARD_CREDENTIALS}"
      - "traefik.http.middlewares.traefik-https-redirect.redirectscheme.scheme=https"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.traefik.middlewares=traefik-https-redirect"
      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik.lab.ferjan.net`)"
      - "traefik.http.routers.traefik-secure.middlewares=traefik-auth"
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=lab.ferjan.net"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.lab.ferjan.net"
      - "traefik.http.routers.traefik-secure.service=api@internal"

secrets:
  cf_api_token:
    file: ./cf_api_token.txt

networks:
  proxy:
    external: true
```

Zaženemo Traefik

    docker compose up -d

Preverimo delovanje:

    docker logs traefik
    cat /vagrant/traefik/data/acme.json

Preverimo nadzorno konzolo na naslovu https://traefik.lab.ferjan.net, kjer se prijavimo s poverilnicami iz datoteke `.env`.

## Portainer

Pripravimo datotečno strukturo za Portainer.

``` sh
./portainer
├── data
└── docker-compose.yml
```

Image tag nastavimo na [zadnjo stabilno različico](https://github.com/portainer/portainer/releases/latest).

``` yaml
services:
  portainer:
    image: portainer/portainer-ce:2.21.3
    container_name: portainer
    restart: unless-stopped
    volumes:
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.lab.ferjan.net`)"
      - "traefik.http.routers.portainer.entrypoints=https"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
    networks:
      - proxy
    # command: --log-level DEBUG
    extra_hosts:
      - "authentik.lab.ferjan.net=192.168.1.10" # only if u don't use DNS

networks:
  proxy:
    external: true
```

> `extra_hosts` definiramo le v primeru, da namesto DNS uporabljamo lokalno hosts dototeko. Portainer potrebuje informacijo na katerem IP naslovu se nahaja naš Authentik, sicer Oauth ne bo deloval. V primeru, da smo imena vpisali v DNS (priporočljivo) lahko `extra_hosts` izpustimo.

Zaženemo Portainer

    docker compose up -d

Tako, Portainer je nastavljen. Odpremo lahko https://portainer.lab.ferjan.net in nastavimo geslo admin uporabnika.

## Authentik

Pripravimo datotečno strukturo za Authentik.

``` sh
./authentik
└── .env
└── docker-compose.yml
```

Pripravimo datoteko `docker-compose.yml`.

> Izhajamo lahko iz uradne `docker-compose.yml` datoteke (wget https://goauthentik.io/docker-compose.yml), ki jo prilagodimo za uporabo s Traefik.

``` yaml
services:
  postgresql:
    image: docker.io/library/postgres:16.4
    container_name: authentik-postgresql
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      start_period: 20s
      interval: 30s
      retries: 5
      timeout: 5s
    volumes:
      - database:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${PG_PASS:?database password required}
      POSTGRES_USER: ${PG_USER:-authentik}
      POSTGRES_DB: ${PG_DB:-authentik}
    env_file:
      - .env
    networks:
      - backend
  redis:
    image: docker.io/library/redis:7.4.1
    container_name: authentik-redis
    command: --save 60 1 --loglevel warning
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep PONG"]
      start_period: 20s
      interval: 30s
      retries: 5
      timeout: 3s
    volumes:
      - redis:/data
    networks:
      - backend
  server:
    image: ghcr.io/goauthentik/server:2024.8.3
    container_name: authentik-server
    restart: unless-stopped
    command: server
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
    volumes:
      - ./media:/media
      - ./custom-templates:/templates
    env_file:
      - .env
    depends_on:
      - postgresql
      - redis
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.authentik.rule=Host(`authentik.lab.ferjan.net`)"
      - "traefik.http.routers.authentik.entrypoints=https"
      - "traefik.http.routers.authentik.tls=true"
      - "traefik.http.services.authentik.loadbalancer.server.port=9000"
      - "traefik.docker.network=proxy"
    networks:
      - backend
      - proxy
  worker:
    image: ghcr.io/goauthentik/server:2024.8.3
    container_name: authentik-worker
    restart: unless-stopped
    command: worker
    environment:
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__USER: ${PG_USER:-authentik}
      AUTHENTIK_POSTGRESQL__NAME: ${PG_DB:-authentik}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${PG_PASS}
    # `user: root` and the docker socket volume are optional.
    # See more for the docker socket integration here:
    # https://goauthentik.io/docs/outposts/integrations/docker
    # Removing `user: root` also prevents the worker from fixing the permissions
    # on the mounted folders, so when removing this make sure the folders have the correct UID/GID
    # (1000:1000 by default)
    #user: root
    volumes:
    #  - /var/run/docker.sock:/var/run/docker.sock
      - ./media:/media
      - ./certs:/certs
      - ./custom-templates:/templates
    env_file:
      - .env
    depends_on:
      - postgresql
      - redis
    networks:
      - backend

volumes:
  database:
    driver: local
  redis:
    driver: local

networks:
  proxy:
    external: true
  backend:
    external: true
```

Pripravimo `.env` spremenljivke:

    # Create .env variables 
    echo "PG_PASS=$(openssl rand -base64 36 | tr -d '\n')" >> .env
    echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 60 | tr -d '\n')" >> .env
    echo "AUTHENTIK_ERROR_REPORTING__ENABLED=true" >> .env

Poženemo docker vsebnik.

    docker compose up -d

Preverimo, da so se vsebniki uspešno zagnali in da ne pišejo napak v log zapise. 

> POZOR: Vsebnika server in worker se zaganjata malce dlje časa.

### Nastavitve Authentik

Odpremo https://authentik.lab.ferjan.net/. Če smo vse naredili prav nas bo pričakalo prijavno okno Authentika.
Pred prvo prijavo si ustvarimo inicialni uporabniški račun na naslovu https://authentik.lab.ferjan.net/if/flow/initial-setup/.

V admin nastavitvah si poleg privzetega admin uporabnika `akadmin` naredimo še svojega uporabnika, mu nastavimo geslo, ga damo v admin grupo in mu vklopimo MFA.

Preverimo prijavo s svojim uporabnikom preko MFA in onemogočimo privzeti račun `akadmin`.

> Pri vklopu MFA sem imel nekaj težav, ki jih je rešil šele ponovni zagon.

#### Dodajanje novega providerja v Authentik

Za začetek dodamo OAuth2/OpenID providerja z imenom **portainer**.
Zabeležimo si **ClientID** in **ClientSecret**.
Za **redirectURIs** nastavimo naslov našega Porteinerja, t.j. https://portainer.lab.ferjan.net/

#### Dodajanje aplikacije v Authentik

Pod Applicatins > Applications dodamo novo aplikacijo **Portainer**.

Ime: Portainer

Slug: portainer

Provider: portainer 

#### Nastavitve Portainer za Oauth
V nastavitveh Portainerja pod Settings > Authentication nato nastavimo Oauth, označimo "Automatic user provisioning".

Za OAuth konfiguracijo sledimo dokumentaciji https://docs.goauthentik.io/integrations/services/portainer/.

> Pri Logout URL-ju pazimo, da uporabimo ime našega portainerja

#### Test delovanja
Odjavimo se iz Portainerja in se ponovno prijavimo, tokrat z Oauth.
Če smo vse prav naredili nas bo prijavilo v Portainer z uporabnikom s katerim smo prijavljeni v Authentik.
Ker uporabnik v Portainerju še nima ustreznih pravic se v Portainer ponovno prijavimo z internim admin uporabnikom, ter novega uporabnika dodamo v ustrezno skupino (npr. med admine).

## Nastavitve Mac OS za neprestano delovanje
Kot že omenjeno celotna postavitev teče na starejšem Macbook PRO prenosniku. V kolikor bo homelab neprestano prižgan bo potrebno nastaviti, da se prenosnik ne bo samodejno ugasnil in da bo deloval tudi pri zaprtem zaslonu.

Za delovanje pri zaprtem zaslonu nastavimo **System Preferences > Battery > Power adapter > prevent computer from sleeping automatically when the display is off**.

Za preprečitev samodejnega ugašanja namestimo in vklopimo aplikacijo **Amphetamine**.