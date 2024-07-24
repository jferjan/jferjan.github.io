---
tags:
  - IT
---

# Testiranje GitHub Actions

Funkcionalnost GitHub actions sem stestiral na enostavnem primeru. Zadal sem si nalogo, da v repozitoruju [jferjan/smrekca](https://github.com/jferjan/smrekca) :material-github: vzpostavim avtomatiziran CI workflow, ki bo ob spremembah kode repozitorija zagnal avtomatizirano izgradnjo Docker slike `jferjan/smrekca:latest` in jo naložil v oddaljeni repozitorij Docker Hub.


## 1. Ustvarjanje žetona na DockerHub

V nastavitvah Docker Hub računa si ustvariš *"Personal access token"*
https://app.docker.com/settings/personal-access-tokens

**Description:** GitHub

**Scope:** Read & Write

## 2. Dodajanje skrivnosti v GitHub repozitorij

V GitHUb repozitoriju greš v nastavitve repozitorija: 

"*Settings > Secrets and Variables > Actions*"

Dodaš dve skrivnosti. Gumb "*New Repository Secret*".

`DOCKERHUB_USERNAME`

`DOCKERHUB_TOKEN`

## 3. Narediš datoteko ci.yml

V mapi `.github/workflows` narediš datoteko `ci.yml` s sledečo vsebino:

!!! note
    Ime mape ni poljubno. Mapa `.github/workflows` se mora nahajati v rootu repozitorija.
    Ime datoteke `ci.yml` je poljubno. 
    V mapi `.github/workflows` imamo lahko več delotokov, t.j. več `.yml` datotek.

``` yaml
name: Build and Push Docker Image to Docker Hub

on: push
jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub registry
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Login to Docker Hub registry
        uses: docker/login-action@v3
        with:
          username: ${{secrets.DOCKERHUB_USERNAME}}
          password: ${{secrets.DOCKERHUB_TOKEN}}

      - name: Build and push Docker image to Docker Hub registry
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: jferjan/smrekca:latest
```
### Legenda

`on:push`

: Workflow se bo izvedel vsakič, ko naredimo spremembo v GitHub repozitoriju. 

`jobs:`

: Seznam nalog, ki jih bo workflow izvedel. V našem primeru se bo izvedla ena naloga (job) z imenom `push_to_registy`.

`push_to_registry:`

: Ime naloge. Ime je poljubno.

`runs-on: ubuntu-latest`

: Naloga (job) se bo izvedela na GitHub runnerju ubuntu-latest. 

`steps:`

: Seznam korakov znotraj naloge. V našem primeru se bodo znotraj naloge `push_to_registry` izvedli trije zaporedni koraki (steps). V našem primeru so vsi trije koraki v obliki predpripravljenih GitHub akcij.

`uses: actions/checkout@v4`

: Predpripravljena GitHub akcija [actions/checkout](https://github.com/marketplace/actions/checkout), ki v GitHub runnerju naredi checkout repozitorija.

`uses: docker/login-action@v3`

: Predpripravljena GitHub akcija [docker/login-action](https://github.com/marketplace/actions/docker-login), ki Github runner prijavi v Docker Hub. Pri tem uporabimo skrivnosti, ki smo jih uporabili v koraku 2.

`docker/build-push-action@v6`

: Predpripravljena GitHub akcija [docker/build-push-action](https://github.com/marketplace/actions/build-and-push-docker-images), ki poskrbi za izgradnjo Docker slike in porine sliko v Docker Hub repozitorij.


## 4. Preveriš delovanje

Workflow `ci.yml`, ki smo ga pripravili v prejšnjem poglavju se ob izvedel ob vsaki spremembi v GitHub repozitoriju. To pomeni, da se bo prvič izvedel že ob prvem commitu datoteke `ci.yml`.

V GitHub v repozitoriju izbereš zavihek *"Actions"* in preveriš, če se je workflow uspešno izvedel.
