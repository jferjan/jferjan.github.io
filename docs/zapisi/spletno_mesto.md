---
tags:
  - IT
---

# Potrebuješ brezplačno spletno mesto?

Ja vem, naslov je clickbait, danes ni nič brezplačno. Pa vseeno, tale recept za spletno mesto se precej približa temu. Vse kar moraš investirati je nekaj časa, ki pa si ga lahko bistveno skrajšaš z napotki v tem zapisu. V kolikor želiš spletno mesto gostovati na lastni domeni je strošek še registracija in gostovanje domene, če pa ti ustreza gostovanje na naslovu **username.github.io** pa tudi ta strošek odpade. 

## Uporabljena tehnologija
- GitHub Pages - Zastonj spletno gostovanje enga statičnega spletnega mesta. Možnost gostovanja na lastni domeni. Brezplačen certifikat.
- MKDocs material - Generator statičnega spletnega mesta z dodano meterial design predlogo
- Docker - Okolje za delo z vsebniki.

## Namen/Motivacija
- Preizkusiti [GitHub Pages](https://pages.github.com/)
- Preizkusiti [MKDocs z material temo](https://squidfunk.github.io/mkdocs-material/)
- Spoznati MarkDown 
- Pričeti uporabljati GitHub 
- Vzpostaviti spletno prisotnost

## Kaj potrebuješ
- Račun na GitHub
- Git commandline klienta. Npr. [Git for Windows](https://gitforwindows.org/)
- Docker okolje. Npr. [Docker Desktop](https://www.docker.com/products/docker-desktop/) 
- Vsebino repozitorija https://github.com/jferjan/jferjan.github.io.git
- Poznavanje osnov dela z Docker vsebniki (mogoče gre tudi brez tega).

## Vzpostavitev

### Naredi access token, ki se uporablja namesto github gesla

V svojem GitHub računu pojdi na:
**Settings** => **Developer Settings** => **Personal Access Tokens** => **Tokens (classic)** => **Generate new token** => Vnesi geslo in izpolni formo => **Generate token** => Skopiraj generirani Token (Žeton), bo v obliki kot npr. ghp_sFhFsSHhTzMDreGRLjmks4Tzuzgthdvfsrta

Pri izpolnjevanju forme določiš obdobje veljavnosti in obseg pravic, ki jih bo imel ta žeton.
Pri določanju obsega pravic sem izbral **repo** in **project** in **workflow**. Pravica **workflow** je pomembna, če bomo objavljali s pomočjo GitHub Actions.

### Posodobi poverilnice na svojem klientu 

Ko sem vzpostavljal zadevo sem bil na Windows klientu, zato se ta del nanaša na Windows.

Control Panel => Windows Credentials => find git:https://github.com => Edit => On Password replace with with your GitHub Personal Access Token => You are Done
**Nadzorna plošča** => **Uporabniški računi** => **Upravitelj poverilnic** => poišči **git:https://github.com** => Uredi => Geslo zamenjaj z Žetonom.

V kolikor ne najdeš **git:https://github.com** dodaj generično poverilnico in vnesi uporabniško ime in Žeton.

### Ustvarjanje repozitorija
Naredi repozitorij z imenom **jferjan.github.io.** GitHub bo vsebino tega objavil na naslovu [jferjan.github.io](https://jferjan.github.io/). Pomembno je, da je naslov repozitorija v obliki **username.github.io**, ker z drugačnim imenom repozitorija zadeva ne deluje.

### Naredi mapo GitHub v svojih dokumentih in vanjo kloniraj git repozitorij
~\GitHub ❯

    git clone https://github.com/jferjan/jferjan.github.io.git
    cd jferjan.github.io

### Naredi docker image
~\GitHub\jferjan.github.io main ❯
    
    docker build -t jferjan/mkdocs-material:1 .

### Zaženi MKDocs strežnik za ogled spletnega mesta
~\GitHub\jferjan.github.io main ❯

    docker compose up -d

Spletno mesto si lahko ogledaš na naslovu [localhost:8000](http://localhost:8000/).

Strežnik lahko izklopiš z

~\GitHub\jferjan.github.io main ❯

    docker compose down

### Posodobi vsebino in konfiguracijo spletnega mesta

Hja, na tej toči si boš verjetno moral malo prebrati dokumentacijo MKDocs. Predvsem za lažje razumevanje osnov konfiguracije.

V osnovi je struktura MKDocs sila preprosta. Ena konfiguracijska datoteka **mkdocs.yml** in v MarkDown sintaksi napisane datoteke, ki jih doložiš v mapo **docs/**

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

### Dodaj datoteke in jih porini v main vejo repozitorija
~\GitHub\jferjan.github.io main ❯

    git add --all
    git commit -m "Dodane spremebe spletnega mesta"
    git remote rm origin # Če bomo porivali kodo v drug repo kot jferjan.github.io
    git remote add origin URL_TO_GITHUB_REPO # Če bomo porivali kodo v drug repo kot jferjan.github.io
    git push -u origin main

### Varianta 1: Ročno zgeneriraj statično spletno mesto in ga porini v GitHub repozitorij v vejo gh-pages


~\GitHub\jferjan.github.io main ❯

    docker run --rm -it -v ${PWD}:/docs jferjan/mkdocs-material:1 gh-deploy

> Vnesi uporabniško ime in žeton.

> Ob prvi uporabi GitHub avtomatično ustvari vejo **gh-pages**. 

### Varianta 2: Nastavi Github actions workflow za avtomatično objavo sprememb v main veji repozitorija.

Na kratko. V rootu repozitorija je potrebno narediti mapo `.github` znotraj njega mapo `workflows` in znotraj te mape datoteko `ci.yml` z vsebino:

``` yaml
name: ci 
on:
  push:
    branches:
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV 
      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material 
      - run: mkdocs gh-deploy --force
```

Spremembe porinemo v main vejo našega repozitorija in s tem je konfiguracija GitHub Actions končana. Od sedaj naprej se bo ob vsaki spremembi main veje repozitorija avtomatično zagnala deploy procedura, ki bo zgradila vsebino veje gh-pages.

Uspešnost izvajanja deploy procedur lahko preverimo v [Github actions konzoli](https://github.com/jferjan/jferjan.github.io/actions/workflows/ci.yml)

### Spremeni nastavitve GitHub repozitorija jferjan.github.io, da bo Github pages uporabljal vsebino veje "gh-pages"

Nastavitve najdeš znotraj repozitorija pod **Settings** => **Pages**  
[github.com/jferjan/jferjan.github.io/settings/page](https://github.com/jferjan/jferjan.github.io/settings/pages)

Build an deployment branch nastaviš na **gh-pages**.