---
title: "Analys av Officiell Förbrukningsdata (Vattenfall) vs Lokal Extraktion"
date: 2026-02-27
tags:
  - energy-optimization
  - vattenfall
  - data-validation
  - peak-analysis
related:
  - "[[00_project_baseline_v2.md]]"
  - "[[plot_phase_load.py]]"
---

# Analys av Officiell Förbrukningsdata (Vattenfall)

Genom att korsköra din officiella nätdatabas från Vattenfall med vår lokala InfluxDB-kalkyl får vi äntligen det "ekonomiska facitet". Det här är ett mycket viktigt steg för att säkerställa att våra lokala beräkningar inte leder oss fel när vi framöver ska börja styra mot Nord Pool.

## 📊 Observation: Månadens 3 högsta effekttoppar

Jag har analyserat Vattenfalls rådata för februari 2026. Här är de tre timmar då huset drog absolut mest el. Jag har också lagt in din referens på ca 930W (0.93 kWh/h) för värmepumpens kompressor för att visa proportionerna.

| Datum & Timme | Total Förbrukning (Vattenfall) | Utetemp | Est. Värmepump (0.93 kW) | Est. Övrigt (Bastu/VVB/Spets) | VP-andel av totalen |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **4 feb kl 07:00** | **10.95 kWh** | -4.4 °C | ~0.93 kWh | ~10.02 kWh | **8.5 %** |
| **5 feb kl 14:00** | **9.40 kWh** | -4.8 °C | ~0.93 kWh | ~8.47 kWh | **9.9 %** |
| **6 feb kl 15:00** | **8.67 kWh** | -6.1 °C | ~0.93 kWh | ~7.74 kWh | **10.7 %** |

*(Notering: Även 18 februari kl 16:00-18:00 hade toppar på upp mot 8.6 kWh, vilket sammanfaller med det bastubad vi tidigare identifierat i fas-loggarna).*

>[!info] Insikt om värmepumpens roll
> Även om Nibe-pumpen ibland aktiverar sin eltillsats (som drar mer), bekräftar detta att din **grundläggande värmepump (kompressorn) utgör en försvinnande liten del av dina effekttoppar**. Nästan all energi under dessa timmar går till rena resistiva laster (Bastu på 8kW, Varmvattenberedare eller spetsel). 

## ⚖️ Konsekvens: Vattenfall vs Vår Trefas-kalkyl

Du frågade hur väl vår formel (`Total_Power_W = (L1 + L2 + L3) * 230`) står sig mot Vattenfalls officiella data. Svaret är att det finns en **inbyggd matematisk diskrepans** som vi måste vara medvetna om:

1. **Aktiv vs Skenbar effekt:** 
   - Vattenfall debiterar dig för **Aktiv effekt (W/kWh)**.
   - Vår ström-formel beräknar **Skenbar effekt (VA)**. 
   Eftersom värmepumpens kompressor är en elmotor har den en fasförskjutning ($\cos \phi$). Det betyder att den drar fler Ampere än vad som faktiskt registreras som Watt på Vattenfall-mätaren.
2. **Externa förbrukare:** Vattenfall mäter allt, inklusive dina utomhuslaster, medan våra fas-klämmor sitter på en specifik plats i centralen.

>[!warning] Data Scientistens Slutsats
> **Vår kalkyl överskattar ofta Watt-förbrukningen något** jämfört med Vattenfall. Men detta är faktiskt en enorm fördel! Eftersom dina huvudsäkringar (16A) bryter baserat på den Skenbara effekten (strömmen), är vår lokala metodik mycket säkrare för att bygga en **Säkringsvakt** i Home Assistant. Vattenfalls data hade varit för "snäll" och kunnat leda till att vi blåste säkringen.

---

## 🛠️ Action Plan

Nu när vi validerat datan vet vi exakt hur vi ska använda de olika källorna framåt:

1. **För Säkringsbalansering (Operation 16A):** Vi fortsätter lita till 100% på InfluxDB och Ampere-faserna (`L1`, `L2`, `L3`).
2. **För Kostnadskalkyl (Nord Pool):** När vi räknar på besparingar i Home Assistant kommer vi applicera en korrigeringsfaktor (ca 0.9) på värmepumpens skenbara effekt för att matcha Vattenfalls faktiska debitering.
3. **Identifiera 4:e februari:** Toppen på **10.95 kWh** kl 07:00 på morgonen är enorm. Bastade du kl 07 på morgonen? Om inte, tyder detta på att Nibe-pumpens elpatron (eltillsats) gick in på max (ca 6.5 kW) samtidigt som varmvattnet laddades och kanske ett motorvärmaruttag användes.

>[!check] Nästa steg
> Nu har vi ställt in flödena (Delta-T) och validerat effekten. Ska vi nu köra det script som ritar upp din förbrukning timme för timme över ett dygn (Dygns-profil), så att du visuellt kan se varför 4-6 februari blev så dyra?