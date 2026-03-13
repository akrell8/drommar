---
Relaterad kod:
  - System/energy_analysis/scripts/heat_pump_performance.py
  - System/energy_analysis/scripts/correlation_script.py
---
Tomas, tack för dessa insikter. Som din Senior Data Scientist måste jag vara ärlig: **Siffrorna och grafen visar en maskin som kämpar extremt hårt.** 

Dina justeringar (-100 gradminuter och kurva 4) var helt rätt tänkta, men datan visar att huset just nu förlorar värme snabbare än vad pumpen hinner producera, vilket skapar en "stress-loop".

Här är min tekniska analys av din audit.

### Filplacering
Spara denna fördjupade analys här:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/efficiency_deep_dive_v1.md`

---

# Analys: Värmepumps-Audit "Klippfisken"

## 1. COP-Analys: 1.89 (Lågt men förklarligt)
En COP på **1.89** är i det lägre skiktet för en F750. 
*   **Orsak:** Tittar vi på grafen ser vi att kompressorn (`Hz`) ligger på max (90–120 Hz) nästan hela tiden. En inverterpump är som effektivast vid 30–50 Hz. Vid maxvarvtal sjunker verkningsgraden dramatiskt, och förångaren fryser igen mycket snabbare.

## 2. Delta-T: "Vattnet rusar"
Dina Delta-T-värden (**2.57°C** för radiatorer och **1.83°C** för golvvärme) är **kritiskt låga**.
*   **Diagnos:** Dina cirkulationspumpar körs med för högt flöde. Vattnet snurrar så snabbt genom rören att det inte hinner avge sin värme till rummen.
*   **Effekt:** Detta lurar värmepumpen att tro att huset är mättat, trots att innetemperaturen är låg. Det ökar också slitaget och elförbrukningen på cirkulationspumparna i onödan.
*   **Rekommendation:** Sänk hastigheten på cirkulationspumparna (GP1/GP6) tills du når ett Delta-T på ca **5–8°C** för radiatorer och **3–5°C** för golvvärme.

## 3. Graf-analys: "Stress-mönstret"
Grafen `pump_response.png` är extremt talande:
*   **Röda spikar (Hz):** Du ser att frekvensen ständigt slår i taket och sedan dyker till noll. De tvära dyken till noll är **avfrostningar**. 
*   **Gradminuter (Blå linje):** Den ligger "parkerad" i botten runt **-360**. 
*   **Slutsats:** Eftersom gradminuterna aldrig klättrar uppåt mot 0, får kompressorn aldrig vila. Den kör 100% tills den isar igen, avfrostar i 45 minuter (då gradminuterna sjunker ännu mer), och börjar sedan om i panik.

---

### Strategisk Action Plan

För att vi ska kunna optimera mot Nord Pool **måste** vi först få pumpen att "hinna ikapp" huset. Annars finns det inget utrymme att flytta laster.

#### Steg 1: Balansera flödet (Högsta prioritet)
Justera ner cirkulationspumparna i Nibe-menyn (Meny 5.1.11). Målet är att höja Delta-T. Detta kommer att göra värmeöverföringen till huset mer effektiv.

#### Steg 2: Analysera "Vented Air" (Avluft)
Jag misstänker att när kompressorn kör på 100 Hz, så går `Temp_Vented_Air` ner extremt djupt (kanske -15°C). 
**Kör detta lilla Python-utdrag för att kolla korrelationen:**

```python
import pandas as pd
df = pd.read_csv("~/Git/Visioner/projects/data-analys/data/holistic_energy_data.csv")
# Kolla snitt-avluft när kompressorn kör hårt (>80 Hz)
hard_running = df[df['Compressor_Hz'] > 80]
print(f"Snitt-avluft vid hård körning: {hard_running['Temp_Vented_Air'].mean():.2f} °C")
```

#### Steg 3: Home Assistant-styrning (Framtiden)
När vi börjar med Nord Pool-optimering, ska vi använda en strategi som heter **"Pre-heating"**. 
1.  Vi höjer börvärdet (kurvan) *innan* elen blir dyr.
2.  Vi "laddar" husets tröghet (golvvärmen) så att gradminuterna är på **0** när den dyra timmen börjar.
3.  Då kan vi låta pumpen vila helt under de dyraste timmarna.

### Nästa steg
Tomas, är du redo att vi tittar på **Nord Pool-datan** för samma period? Jag vill se om de timmar då din pump "panikkör" råkar sammanfalla med de dyraste timmarna på dygnet. 

**Ska jag skriva ett skript som hämtar Nord Pool-priser och lägger till dem i din CSV-fil?**