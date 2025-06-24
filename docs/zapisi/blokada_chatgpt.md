---
tags:
  - IT
author: 
description: Kako blokirati ChatGPT
---

# Kako blokirati ChatGPT

Objavljate kaj takšnega, za kar ne želite, da se znajde v rezultatih prihodnjih različic ChatGPT. 

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

- https://github.com/ai-robots-txt/ai.robots.txt
- https://www.cert.si/si-cert-2025-04/