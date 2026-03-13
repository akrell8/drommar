---
Relaterad kod:
  - "[[fuse_simulator.py]]"
---
Tomas, de här siffrorna är guld värda. De bekräftar svart på vitt att din elanläggning lever på lånad tid varje gång bastun startar. Som din Senior Data Scientist kan jag nu ge dig en exakt analys av vad som händer bakom kulisserna i elcentralen.

### Filplacering för rapporten
Spara denna fördjupade analys här:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/fuse_stress_test_results.md`

---

# Analys: Säkringssimulering och Belastningsmönster

## 1. Den kritiska punkten (L2-problemet)
Vi ser ett återkommande mönster där **L2** kliver upp över 20A.
*   **Exempel rad 2655:** L2=21.2A, L3=12.7A, L1=1.7A. 
    *   *Tolkning:* Bastun körs sannolikt som en **2-fas-last** (L2+L3). Om det vore en balanserad 3-fas 8kW bastu skulle vi se ~12A på alla tre faser. Här "stjäl" L2 nästan dubbelt så mycket som den borde.
*   **Exempel rad 1153:** L1=14.0A, L2=20.6A, L3=14.4A. 
    *   *Tolkning:* Här är det riktigt illa. Bastun går (L2/L3) samtidigt som Nibe-pumpens kompressor sannolikt går för fullt på L1 (~13A). Din marginal (Headroom) är **-4.6A**. Du överbelastar hela systemet.

## 2. Varför räddar inte Nibe F750 situationen?
Du nämnde att Nibe har effektbegränsning via drosslar. Manualen (s. 23) förklarar varför det inte räcker:
1.  **Nibe kan bara styra sig själv:** När drosslarna känner av >16A, kommer Nibe att stega ner sin egen elpatron till 0kW.
2.  **Baslasten är för hög:** Om bastun (felbalanserad) drar 18-19A på L2, så spelar det ingen roll om Nibe stänger av sin elpatron helt. Säkringen kommer ändå att gå till slut eftersom bastun ensam ligger över 16A.
3.  **Kompressorn:** Nibe kan varva ner kompressorn i nödfall, men den prioriterar att hålla huset varmt.

## 3. Slutsats för elbalansen
Din anläggning är för närvarande **inte redo för Nord Pool-optimering**. Om vi via Home Assistant schemalägger tunga laster till billiga timmar (ofta nattetid när det är som kallast), och bastun råkar gå samtidigt, kommer huset mörkläggas omedelbart.

---

### Action Plan: "Operation 16 Ampere"

#### Steg 1: Elektriker-check (Högsta prioritet)
Visa dessa loggar för din elektriker. Hen behöver:
1.  Öppna Tylö-aggregatet och kontrollera om alla tre rörelement fungerar och är inkopplade på varsin fas. Det **ska** vara 12A + 12A + 12A.
2.  Om bastun är korrekt, måste hen flytta andra tunga enfaslaster (t.ex. ugn, vattenkokare eller en specifik radiatorgrupp) **bort från L2 och L3 till L1**.

#### Steg 2: Home Assistant "Pre-emptive Shedding" (Mjukvara)
Vi kan bygga en logik i Home Assistant som agerar *innan* Nibes inbyggda vakt hinner reagera.

**Spara i:** `~/Git/System/energy_analysis/docs/ha_automation_strategy.yaml`

```yaml
# Strategi: Om bastun startar, tvinga Nibe i "Ekonomiläge" 
# eller blockera elpatronen helt via SG Ready-ingången.
automation:
  - alias: "Energi: Skydda huvudsäkring vid bastubad"
    trigger:
      - platform: numeric_state
        entity_id: sensor.power_l2 # Eller din bastu-sensor om du har en
        above: 15
    action:
      - service: notify.mobile_app_tomas
        data:
          title: "Hög belastning detekterad!"
          message: "L2 är över 15A. Begränsar Nibe-pumpen tillfälligt."
      # Här kan vi skicka kommando till Nibe via NibeGW eller Uplink 
      # att sätta 'eltillsats_stopp' till en mycket låg temperatur.
```

---

### Nästa steg i analysen
Vill du att vi gör en **"Worst Case"-analys** där vi räknar ut hur många timmar totalt under månaden du legat i riskzonen (över 16A)? Det ger oss ett mått på hur ofta du faktiskt har tur att säkringen inte går. 

Eller ska vi börja titta på hur vi räknar om Ampere till **Nord Pool-kostnad** för att se vad bastubaden faktiskt kostar dig i kronor?