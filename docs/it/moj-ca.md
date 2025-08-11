---
tags:
  - IT
---

# Si želiš imeti svoj CA in ustvarjati lastne (self-signed) certifikate za lokalne domenske naslove?

Spodaj opisujem kratek primer kako sem naredil lastni CA in ustvaril wildcard certifikat za *.ferjan.local.

## Ustvari svoj CA

1. Ustvari RSA ključ **ca-key.pem**

        sudo openssl genrsa -aes256 -out ca-key.pem 4096
    >Izberi močno geslo in si ga varno shrani

2. Generiraj javni CA certifikat **ca.pem** (z veljavnostjo 10 let). 

        sudo openssl req -new -x509 -sha256 -days 3650 -key ca-key.pem -out ca.pem
    >Za CN sem napisal "ferjan", ostalo pustim prazno.

## Ustvari certifikat (s svojim CA)

1. Najprej ustvarimo RSA ključ **cert-key.pem**

        sudo openssl genrsa -out cert-key.pem 4096

2. Ustvarimo zahtevek za podpis potrdila **cert.csr** (Certificate Signing Request (CSR))

        sudo openssl req -new -sha256 -subj "/CN=ferjan" -key cert-key.pem -out cert.csr

3. ustvarimo datoteko **extfile.cnf** z vsemi SAN imeni (Subject Alternative Name)

        echo "subjectAltName=DNS:*.ferjan.local" | sudo tee -a extfile.cnf
    >Dodamo lahko tudi več SAN imen, in/ali IP naslovov.
    >
    >echo "subjectAltName=DNS:your-dns.record,IP:257.10.10.1" | sudo tee -a extfile.cnf

4. V **extfile.cnf** iz prejšnje točke dodamo informacijo, da je certifikat namenjen za avtentikacijo strežnika

        echo "extendedKeyUsage = serverAuth" | sudo tee -a extfile.cnf

5. Naredimo certifikat **cert.pem** (z veljavnostjo 10 let)
        
        sudo openssl x509 -req -sha256 -days 3650 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial

6. Naderimo fullchain certifikat **fullchain.pem**

        cat cert.pem | sudo tee -a fullchain.pem
        cat ca.pem | sudo tee -a fullchain.pem

> Fullchain certifikat ni nič drugega kot združitev wildcard certifikata *ferjan.local s certifikatom CA. 

## Dodaj CA med zaupanje vredne CA-je

Če želimo, da je certifikat označen kot zaupanja vreden je potrebno zaupati CA-ju, ki ga je izdal.
V ta namen dodamo certifikat CA v svoj brskalnik ali v svoj OS.

Primer uvoza CA certifikata v MS Windows:
        
        Import-Certificate -FilePath "~\CA\ca.pem" -CertStoreLocation Cert:\LocalMachine\Root

> Powershell je potrebno zagnati kot admin  
> Pod FilePath je potrebno navesti pot do CA certifikata