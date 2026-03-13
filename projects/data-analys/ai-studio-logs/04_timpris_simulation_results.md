---
title: "Resultat: Sanningens ögonblick för Timpris (Facit)"
date: 2026-03-03
tags:
  - energy-optimization
  - nord-pool
  - simulation-results
  - baseline
related:
  - "[[simulate_true_cost.py]]"
  - "[[02_timpris_kalkyl_strategi.md]]"
---

# Resultat av Historisk Timprissimulering

Genom att para ihop 12 månaders historisk förbrukning från Vattenfall (timme för timme) med exakta historiska spotpriser från Nord Pool (SE3), har vi fått fram den sanna kostnaden för fritidshuset.

>[!warning] Kärninsikt: Systemet är inte redo för Timpris
> Med nuvarande beteende och ooptimerad värmepump hade ett timprisavtal kostat **896 SEK mer** än det rörliga månadssnittet under det senaste året. 

## 📊 Analys av Datatrenden

1. **Vinterstraffet:** Titta på februari 2026 (-279 kr) och januari 2026 (-106 kr). Det är under de kalla månaderna förlusten sker. Värmepumpen (och bastun) går som hårdast under morgnar och kvällar när nätet är som mest ansträngt och priset är som högst. Du konsumerar el exakt när elen är som dyrast.
2. **Den falska tryggheten i månadssnitt:** Just nu "räddas" du av att du har ett rörligt månadspris. Du köper dyr el på morgonen, men betalar ett snittpris som dras ner av de billiga nätterna (när du inte förbrukar lika mycket).
3. **Sommarmånaderna (Juni +10 kr):** På sommaren, när värmepumpen bara gör varmvatten och bastun kanske inte används lika ofta, är profilen nästan neutral. Men volymerna är så små (484 kWh) att det inte spelar någon roll för helåret.

## ⚙️ Vad krävs för att vända förlust till vinst?

För att ett timpris ska löna sig måste vi inte bara radera denna underliggande förlust på 896 kr, vi måste dessutom spara mer därtill. Detta kallas **Load Shifting** (Lastförskjutning).

För att lyckas med detta måste vi aktivera Home Assistant.

### Action Plan: Automationsstrategi (HA)
1. **Steg 1: VVB-Blockering.** Varmvattenberedaren i Nibe F750 drar mycket effekt. Vi måste blockera den från att värma vatten kl 06:00-09:00 och 17:00-20:00 varje dag.
2. **Steg 2: Termisk tröghet (Pre-heating).** Vi tvingar upp värmekurvan (+2 grader) kl 01:00-05:00 på natten när elen kostar ören (eller är gratis). Plattan och huset lagrar värmen. Kl 06:00 sänker vi kurvan (-2 grader), vilket får värmepumpen att stanna helt under den dyra morgonen.
3. **Steg 3: Säkringsvakten.** Eftersom vi kommer trycka in mycket energi på natten, måste vi ha en kod som säger: *"Om bastun startas, stäng omedelbart av Nibe-pumpens eltillsats så vi inte skjuter 16A-säkringen"*.

>[!check] Beslut
> Vi byter **inte** avtal till Borås Elhandel Timpris ännu. Vi stannar på rörligt månadspris tills automationerna i Home Assistant är byggda, testade och bekräftade fungerande i våra grafer.