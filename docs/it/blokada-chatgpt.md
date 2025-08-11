---
tags:
  - IT
author: 
description: Kako blokirati ChatGPT
---

# Kako blokirati ChatGPT?

Objavljate kaj takšnega, za kar ne želite, da se znajde v rezultatih prihodnjih različic ChatGPT? 

ChatGPT pajka, ki sliši na ime **GPTBot** lahko dokaj enstavno blokirate.

## Blokada z Robots.txt datoteko.

V `robots.txt` dodamo: 

``` txt
User-agent: GPTBot 
Disallow: /
```

## Blokada IP naslovov ChatGPT bota

- [Naslovi GPTBot-a](https://openai.com/gptbot-ranges.txt)

## Kaj pa ostali boti?

Postopajte podobno kot pri GPTBotu. Nekaj napotkov najdete na sledečih povezavah.

- [https://github.com/ai-robots-txt/ai.robots.txt](https://github.com/ai-robots-txt/ai.robots.txt)
- [https://www.cert.si/si-cert-2025-04/](https://www.cert.si/si-cert-2025-04/)

## Zakaj?

Po več kot 30 letih se menja poslovni model, ki se je na spletu uveljavil s prihodom iskalnika Google. Do sedaj je veljalo, da iskalniki našo vsebino poindeksirajo in nam v zameno posredujejo stranke. S prihodom velikih jezikovnih modelov se ta model podira. Uporabniki namesto iskalnika posegajo po chat botih, ki se hranijo z našo vsebino brez, da bi nas za to kompenzirali[^1].

Čas je, da se uveljavi HTTP status koda [402 Payment Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/402) in model po zgledu [Pay per crawl](https://blog.cloudflare.com/introducing-pay-per-crawl/).

[^1]:  [Content Independence Day](https://blog.cloudflare.com/content-independence-day-no-ai-crawl-without-compensation/)