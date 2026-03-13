---
title: "Ekonomisk Strategi: Rörligt Månadspris vs Timpris (Nord Pool)"
date: 2026-03-03
tags:
  - energy-optimization
  - economics
  - nord-pool
  - strategy
related:
  - "[[00_project_baseline_v2.md]]"
---

# Ekonomisk Strategi: Potentialen med Timpris

Att byta från rörligt månadspris till timpris (timtaxa) är det sista steget i vår optimering. Men innan vi trycker på knappen är det viktigt som Data Scientist att hantera förväntningarna. Det finns en utbredd missuppfattning om *hur mycket* man faktiskt kan spara.

## 📊 Den krassa kalkylen (Grov uppskattning)

En genomsnittlig frånluftsvärmepump i SE3 (din region) i ett fritidshus drar ungefär **8 000 - 12 000 kWh/år**. Din Vattenfall-data bekräftar denna profil: höga toppar på vintern (upp mot 10 kWh/h) och en mycket låg baslast på sommaren (~0.3 kWh/h).

>[!info] Vad kan vi faktiskt påverka?
> Din totala elkostnad består av tre delar:
> 1. **Elnät & Skatt (Vattenfall + Staten):** Ca 1.00 - 1.50 kr/kWh. **(Opaverkbar)**
> 2. **Elhandelsavgift & Påslag (Borås Elhandel):** Ca 0.05 kr/kWh. **(Opaverkbar)**
> 3. **Spotpris (Nord Pool):** Ca 0.40 - 0.80 kr/kWh i snitt. **(Påverkbar!)**

Forskning och data från liknande Home Assistant-projekt visar att en smart styrning av en Nibe F750 (Varmvatten + Pre-heating) snittar på en besparing om **10% till 20% av Spotpriset**.

*   **Uppskattad besparing:** Ca 10-15 öre per kWh.
*   **Årlig besparing i kronor:** **1 000 kr - 2 500 kr / år**.

## ⚖️ "Timprisfällan" för Värmepumpar

Om du byter till timpris *utan* att bygga våra Home Assistant-automationer, kommer du med största sannolikhet att **förlora pengar**. 

Varför? Titta på din data från **22 januari 2026 kl 08:00**:
*   Utetemperatur: -0.7°C
*   Din förbrukning: 5.005 kWh (Värmepumpen kämpar).
*   *Konsekvens:* Kalla vintermorgnar är när elen är som absolut dyrast (alla kokar kaffe och industrin startar). Med ett månadspris är du skyddad av "kollektivets snitt". Med timpris tar du hela smällen själv.

## 🛠️ Vad vi kan flytta (Vår "Shift-strategi")

För att vända timpriset till en vinst måste vi disponera om dygnet.

| Last-typ | Kan vi flytta den? | Åtgärd i Home Assistant | Risk / Biverkning |
| :--- | :--- | :--- | :--- |
| **Varmvatten (VVB)** | **Lätt** | Blockera VVB kl 06-10 och 17-20. | Ingen. Vattnet i F750 håller värmen. |
| **Huskroppen (Värme)** | **Medium** | "Pre-heat". Höj kurvan +2 inatt. Sänk på dagen. | Trögheten i golvvärmen (S2) är perfekt för detta. Radiatorer (S1) svalnar fortare. |
| **Bastu (8kW)** | **Mycket Svårt** | Ingen. | Du bastar när du vill basta (ofta kvällstid = dyrt). |
| **Disk/Tvätt** | **Lätt** | Schemaläggning via maskinernas inbyggda timer. | Kräver nytt beteende av dig. |

>[!warning] Vår unika flaskhals: 16A Säkringen
> Om vi programmerar Home Assistant att "gasa för fullt" kl 03:00 på natten för att elen är billig (Kompressor + Elpatron + Varmvatten), **riskerar vi att skjuta din huvudsäkring** eftersom vi vet att din fas L2 redan är tungt belastad. Vi *måste* bygga en lastbalanseringsvakt i Home Assistant innan vi aktiverar aggressiv prisstyrning.

---

## 📈 Nästa Steg: Simuleringsskriptet

För att ge dig exakta siffror istället för en grov uppskattning, har jag skrivit ett Python-skript. Detta skript tar din nya Vattenfall-data för helåret, hämtar historiska priser och simulerar "Vad hade hänt om du haft timpris förra året?".