---
Relaterad kod:
  - "[[analyze_phase_balance.py]]"
  - "[[phase_export_last_month.csv]]"
---
Det här är mycket intressanta siffror, Tomas. Som din Senior Data Scientist ser jag direkt ett par kritiska punkter som vi behöver adressera för att säkra driften i ditt fritidshus.

Här är min analys av din data.

### Filplacering för rapporten
Spara denna analys här:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/phase_analysis_report_v1.md`

---

# Analysrapport: Fasbalansering och Säkringsutnyttjande

## 1. Den "Röda Flaggan": L2-överbelastning
Din **L2** har nått hela **21.20A**. 
*   **Status:** Allvarlig. Du ligger 32.5% över din 16A huvudsäkring.
*   **Varför har inte säkringen gått?** En vanlig 16A trög smältsäkring eller dvärgbrytare (karaktäristik C) tål en viss överlast under en begränsad tid (ofta 1.1 - 1.45 gånger märkströmmen i upp till en timme). Men 21A är precis på gränsen där den kan lösa ut när som helst.

## 2. Den "Gömda Potentialen": L1-ledighet
Titta på dina senaste 5 loggade tillfällen:
*   Vid tidpunkten `2026-02-19 13:10:00` drar **L2 21.2A** och **L3 12.7A**, medan **L1 bara drar 1.7A**.
*   **Slutsats:** Du har en enorm obalans. Du har nästan 14A "ledigt" på L1 samtidigt som L2 riskerar att mörklägga huset.

## 3. Statistisk sammanfattning
*   **Genomsnittlig belastning:** Ca 4A per fas. Det betyder att ditt hus i snitt bara drar ca 2.7 kW. Din baslast är alltså mycket fin och balanserad.
*   **Maximal Obalans (19.50A):** Detta bekräftar att du har en eller flera kraftfulla **enfas-laster** som ligger på fel faser. 
    *   *Misstänkta:* Vattenkokare, diskmaskin, tvättmaskin eller en enfas elbilsladdare/värmepump-elpatron.

---

## Rekommendationer (Action Plan)

### Steg 1: Identifiera "Boven"
Vi behöver veta vad som startade den 19:e februari kl 13:10. 
*   Var du i stugan då? 
*   Kördes ugnen, bastun eller laddades en bil?
*   Eftersom L1 bara drog 1.7A (vilket ofta bara är standby-el/belysning), så ligger nästan all tung utrustning på L2 och L3.

### Steg 2: Lastflytt
Du bör be en elektriker flytta en av de tunga enfas-grupperna (t.ex. diskmaskinen eller en specifik radiator-slinga) från **L2 till L1**. Detta skulle jämna ut kurvan avsevärt.

### Steg 3: Optimering mot Nord Pool
Innan vi börjar styra laster mot billiga timmar (Nord Pool), **måste** vi balansera faserna. Om vi börjar köra tunga laster när elen är billig men allt ligger på L2, kommer säkringen gå direkt trots att elen är billig.

---

## Nästa steg i analysen

För att vi ska kunna ge elektrikern ett exakt underlag, föreslår jag att vi skapar en **visualisering** av dygnet den 19:e februari. Då kan vi se exakt hur länge överlasten pågår.

**Vill du att jag tar fram ett Python-skript för att generera en graf över de mest kritiska dygnen?** (Detta sparar vi i så fall som en PNG i din `data-analys` mapp).