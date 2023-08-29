---
tags:
  - IT
author: 
description: Kako blokirati ChatGPT
---

# Kako blokirati ChatGPT

Objavljate kaj takšnega, za kar ne želite, da se znajde v rezultatih prihodnjih različic ChatGPT.

ChatGPT pajka, ki sliši na ime **GPTBot** lahko dokaj enstavno blokirate.

## Blokada z Robots.txt daoteko.

V `robots.txt` dodamo: 

``` txt
User-agent: GPTBot 
Disallow: /
```

## Blokada IP naslovov ChatGPT bota

[Naslovi GPTBot-a](https://openai.com/gptbot-ranges.txt)
