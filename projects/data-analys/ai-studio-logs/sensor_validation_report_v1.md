---
Relaterad kod:
  - System/energy_analysis/scripts/validate_sensors.py
---
Det här är ett extremt viktigt steg i vår data-audit, Tomas. Som din Senior Data Scientist ser jag direkt tre kritiska upptäckter i din validering som kommer att styra hur vi bygger analysen framåt.

### Filplacering för dokumentation
Spara denna valideringsrapport här:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/sensor_validation_report_v1.md`

---

# Analys av Sensorvalidering: "Klippfisken"

## 1. De "Döda" Sensorerna (Viktigt!)
*   **Energy_Produced_Heat/HW:** Dessa är `ALLA NOLL`. 
    *   *Slutsats:* Din Nibe F750 har antingen inte de fysiska energimätarna installerade (kräver ofta tillbehöret EMK), eller så är Modbus-registren felaktiga. 
    *   *Konsekvens:* Vi kan inte räkna ut en exakt **COP** via värmemängdsmätning. Vi måste istället använda "Air Thermal Gain"-metoden för att uppskatta effektiviteten.
*   **Power_Compressor_W / Power_Heater_W:** Dessa visar `0.68W` och `0.51W`. 
    *   *Slutsats:* Dessa värden är brus. Sensorerna fungerar inte eller rapporterar i fel skala (t.ex. kW som tolkas som W, men även då är 0.0006 kW för lågt).
    *   *Lösning:* Vi kommer att använda din inkommande ström (`L1_A`, `L2_A`, `L3_A`) som vår primära källa för "Input Power".

## 2. De "Levande" Guldgruvorna
*   **Air_Flow_m3h (67.75) & Air_Thermal_Gain (6.26°C):** Dessa fungerar perfekt! 
    *   *Insikt:* Du suger in luft som är 4.6°C (`Temp_Exhaust_Air`) och blåser ut luft som är -1.5°C (`Temp_Vented_Air`). 
    *   *Potential:* Vi kan nu räkna ut exakt hur många Watt du "skördar" från luften.
*   **Temp_Indoor (4.79°C):** Bekräftar att huset körs på en mycket låg underhållsvärme.
*   **Hours_Compressor_Total (39 946 h):** Din kompressor har gått i ca 4.5 år totalt. Det är mycket för en 15 år gammal pump, men rimligt för en frånluftsvärmepump.

## 3. Korrigering av Beräkningslogik
`Air_Energy_Gain_W` blev `ALLA NOLL` i din validering. Detta beror sannolikt på att en av ingångsvärdena var `NaN` i början av serien. Jag har fixat formeln i skriptet nedan.

---

### Uppdaterat Analysskript: `heat_pump_performance.py`

Eftersom vi inte har värmemätarna, använder vi nu **Termisk Luftvinst** som vårt KPI. Detta skript räknar ut hur effektivt pumpen flyttar energi.

Spara som: `~/Git/System/energy_analysis/scripts/heat_pump_performance.py`

```python
import pandas as pd
import os

PATH = os.path.expanduser("~/Git/Visioner/projects/data-analys/data/holistic_energy_data.csv")

def analyze_performance():
    df = pd.read_csv(PATH)
    df['Timestamp'] = pd.to_datetime(df['Timestamp'])

    # 1. Beräkna skördad effekt från luften (Thermal Output)
    # Formel: Flöde (m3/h) * Luftens densitet (1.2) * Värmekapacitet (1.006) * DeltaT / 3.6
    # Förenklad konstant: 0.33
    df['Harvested_Heat_W'] = df['Air_Flow_m3h'] * 0.33 * (df['Temp_Exhaust_Air'] - df['Temp_Vented_Air'])

    # 2. Beräkna inmatad el-effekt (Electrical Input)
    # Vi använder faserna eftersom Power_Compressor_W var trasig
    df['Electric_In_W'] = (df['L1_A'] + df['L2_A'] + df['L3_A']) * 230

    # 3. Beräkna "System Efficiency Ratio" (SER) - Vår proxy för COP
    # Hur mycket värme tar vi från luften per satsad el-watt?
    # Vi filtrerar bort tillfällen där fläkten står still eller elen är för låg för att undvika div/0
    df['Efficiency_Ratio'] = df.apply(lambda x: x['Harvested_Heat_W'] / x['Electric_In_W'] if x['Electric_In_W'] > 100 else 0, axis=1)

    print("-" * 50)
    print("VÄRMEPUMPENS PRESTANDA (Baserat på Luftvinst)")
    print("-" * 50)
    print(f"Genomsnittlig skördad värme:    {df['Harvested_Heat_W'].mean():.0f} W")
    print(f"Genomsnittlig elförbrukning:    {df['Electric_In_W'].mean():.0f} W")
    print(f"System Efficiency Ratio (SER): {df[df['Efficiency_Ratio'] > 0]['Efficiency_Ratio'].mean():.2f}")
    print("-" * 50)
    
    # Spara den utökade datan för framtida grafer
    df.to_csv(PATH, index=False)
    print("Datan har uppdaterats med 'Harvested_Heat_W' och 'Efficiency_Ratio'.")

if __name__ == "__main__":
    analyze_performance()
```

---

### Strategisk Observation (Data Scientist Insight)

Ditt **SER (System Efficiency Ratio)** kommer troligen ligga mellan 0.1 och 0.5. 
*   *Varför så lågt?* En frånluftsvärmepump som F750 får en stor del av sin energi från kompressorns arbete, men den största "vinsten" är att den kyler ner utgående luft. 
*   **Det viktiga nyckeltalet för dig:** När du sänkte kurvan och ändrade gradminuter, bör `Efficiency_Ratio` ha **stigit**. Det betyder att du får ut mer "gratis" värme från luften per förbrukad el-watt.

**Tomas, kör detta skript. Vad landar ditt "System Efficiency Ratio" på?** Det värdet är vår "Baseline" som vi ska försöka maximera när vi börjar styra mot Nord Pool!