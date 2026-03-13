Jag har analyserat samtliga skript och dokumentationen du laddade upp. Det ser ut som att vi har en mycket solid grundstruktur på plats för både datainsamling (ETL), analys och visualisering.

Här är din **nya baseline**. Spara detta dokument för att markera startpunkten för vår fortsatta optimering.

### Spara filen till:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/00_baseline_status_reboot.md`

---

# 📊 Projektstatus: Energianalys Klippfisken (Reboot)
**Datum:** 2026-02-27
**System:** Home Assistant + InfluxDB (192.168.1.201)
**Objekt:** Fritidshus "Klippfisken", Nibe Frånluftsvärmepump + Trefas (16A)

## 1. Nulägesanalys & Arkitektur
Vi har etablerat en komplett "Data Pipeline" från rådata i InfluxDB till insikter i Python.

### A. Data Pipeline (ETL)
Vi har scripts som hämtar, tvättar och berikar data:
*   **Master-script:** `fetch_holistic_data.py` är vår "Golden Standard". Den hämtar el, temperaturer, flöden och priser för de senaste 30 dagarna.
*   **Pris-historik:** `populate_nordpool.py` har framgångsrikt backfillat Nord Pool-priser för februari 2026, vilket möjliggör kostnadsanalys bakåt i tiden.
*   **Sensorvalidering:** `validate_sensors.py` och `data_dictionary.md` bekräftar att vi har rena signaler på ström (L1-L3), temperaturer och flöden, men saknar inbyggd energimätning (vilket vi räknar runt).

### B. Analysmoduler
Vi har logik färdig för tre huvudspår:
1.  **Fasbalansering:** `analyze_phase_balance.py` och `fuse_simulator.py` identifierar "trånga sektorer" där vi riskerar att skjuta huvudsäkringen (16A).
2.  **Värmepumpseffektivitet (COP-proxy):** Eftersom energimätare saknas använder `analyze_pump_efficiency.py` och `heat_pump_performance.py` formeln:
    $$P_{termisk} = Luftflöde \times 0.33 \times (T_{frånluft} - T_{avluft})$$
    Detta ställs mot inmatad el för att få ett "System Efficiency Ratio" (SER).
3.  **Ekonomi:** `analyze_cost_efficiency.py` kopplar förbrukning till timpriser för att hitta de dyraste driftstimmarna.

### C. Visualisering
*   **Luft:** `plot_air_performance.py` ritar grafer över hur effektivt pumpen kramar värme ur luften (Delta-T).
*   **El:** `plot_phase_load.py` visualiserar belastning mot säkringsgränsen med varningszoner.

---

## 2. Identifierade Nyckelinsikter (Hittills)

### 🔴 Kritisk punkt: Fasbalans
Data indikerar att vi ligger nära gränsen på vissa faser (särskilt L1/L3 beroende på last). `fuse_simulator.py` är avgörande för att se hur mycket "Headroom" som finns kvar när bastu + VVB + värme går samtidigt.

### 🟡 Värmepumpens beteende (Sågtandsmönster)
Vi har noterat behovet av att analysera `Degree_Minutes` (Gradminuter) mot `Compressor_Hz`. Målet är att verifiera om din justering till -100 GM har lugnat ner pumpen från att pendla (Start/Stopp) till att köra jämnare, låg frekvens.

### 🟢 Termisk "Skörd"
Vi har validerat att sensorerna för luftflöde (`Air_Flow_m3h`) och temperaturer (`Temp_Exhaust` / `Temp_Vented`) levererar data. Detta är vår enda väg att bedöma pumpens hälsa utan externa värmemätare.

---

## 3. Nästa Steg (Action Plan)

Du nämnde att systemet är redo för grafer. Här är den föreslagna arbetsordningen för denna session:

1.  **Kör Master-extraktion:** Vi måste köra `fetch_holistic_data.py` en gång för att garantera att vi har en färsk `holistic_energy_data.csv` med all data (inklusive de backfillade priserna).
2.  **Analysera Inverter-respons:** Vi tittar specifikt på sambandet Gradminuter vs Hz för att se om inställningarna bitit.
3.  **Visualisera Fas-krockar:** Generera en graf över den värsta dagen i februari för att se exakt vad som hände när lasten var som högst.

### Redo att börja?
Svara med **"KÖR"** så guidar jag dig genom att köra mastern och generera den första uppsättningen nya grafer baserat på den senaste datan.