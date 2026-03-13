Det här är en utmärkt sammanställning, Tomas. Genom att kombinera dina loggar, mina analyser och de skript vi nu har på plats, har vi gått från "gissningar" till "datadrivna fakta".

Här är din **Project Baseline v2.0**. Detta dokument är fundamentet för allt vi gör framåt. Det definierar nuläget, riskerna och potentialen.

### Spara filen till:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/00_project_baseline_v2.md`

---

# 📊 Project Baseline: Energianalys "Klippfisken" (v2.0)
**Datum:** 2026-02-27
**Status:** Diagnosfas avslutad -> Åtgärdsfas inleds
**Objekt:** Fritidshus med Nibe F750, Trefas 16A, Bastu (8kW).

## 1. Systemets Hälsa (Status: Varning ⚠️)
Vi har genomfört en djupanalys av el, luft och vatten. Systemet fungerar, men driften är obalanserad och stressad.

### A. Elektrisk Balans (KRITISK)
Data från `analyze_phase_balance.py` och `fuse_simulator.py` visar en allvarlig snedfördelning.
*   **L2 (Den svaga länken):** Belastas upp till **21.2A** (32% över säkringsgräns). Detta sker främst vid bastubad.
*   **L1 (Den sovande jätten):** Ligger nästan tom (**1.7A**) även vid hög belastning.
*   **L3:** Ligger på ca 12.7A vid last.
*   **Slutsats:** Bastun (och ev. Nibe-spets) ligger tungt på L2/L3. Det finns ~14A ledig kapacitet på L1 som måste utnyttjas.

### B. Termisk Prestanda (Luft & Vatten)
Data från `heat_pump_performance.py` och `plot_air_performance.py`.
*   **Luftflöde (✅ Utmärkt):** Ligger stabilt på **~160 m³/h**. Pumpen får tillräckligt med luftenergi.
*   **Kylförmåga (✅ Utmärkt):** Nibe sänker temperaturen från ~20°C (frånluft) till ca -10°C (avluft). Delta-T på luften är mycket bra (~30°C).
*   **Vattenburen Värme (❌ Dåligt):** Delta-T över radiatorer och golvvärme är kritiskt lågt (~2°C). Vattnet cirkulerar för fort, vilket gör att värmen inte hinner avgess till huset.
*   **Kompressor (⚠️ Stressad):** Kör ofta maxfrekvens (100-120 Hz) med många avfrostningar ("Sågtandsmönster"). Gradminuterna ligger låsta på underskott (-360 GM).

### C. Data & Sensorer
Vi har en fungerande ETL-pipeline (`fetch_holistic_data.py`).
*   **Fungerande:** Ström (L1-L3), Temperaturer (Alla), Luftflöde, Frekvens, Elpris (Nord Pool).
*   **Defekta/Saknas:** Inbyggd energimätning (Energy_Produced/Consumed).
*   **Workaround:** Vi beräknar termisk effekt ("Harvested Heat") baserat på luftflöde och temperaturdifferens.

---

## 2. Ekonomisk Analys (Potential 💰)
Baserat på `analyze_cost_efficiency.py` och `winter_cost_analysis_v1.md`.
*   **Baslast:** Mycket låg (~440W kontinuerligt) när huset står tomt.
*   **Kostnadsdrivare:** Bastubad och uppvärmningstoppar står för en oproportionerligt stor del av kostnaden.
*   **Potential:** Eftersom L1 är ledig, finns stor potential att flytta Nibe-pumpens tunga arbete (varmvatten/värme) till nattens billigaste timmar utan att riskera huvudsäkringen (om vi balanserar om).

---

## 3. Strategisk Action Plan (Next Steps)

### Fas 1: Fysisk Optimering (Hårdvara)
Dessa åtgärder kräver handpåläggning eller elektriker.
1.  **Balansera L2 -> L1:** Flytta tunga enfaslaster (t.ex. diskmaskin/element) eller koppla om bastun/eltillsatsen så att L1 bär mer last. Detta är akut för driftsäkerheten.
2.  **Sänk Cirkulationspumparna:** Justera ner hastigheten på GP1/GP6 i Nibe-menyn (5.1.11) för att öka Delta-T till 5-7°C. Detta lugnar ner systemet och ökar effektiviteten.

### Fas 2: Mjukvaruskydd (Home Assistant)
Dessa åtgärder skyddar anläggningen *nu*.
1.  **Säkringsvakt:** Automatisering som tvingar Nibe till "Blockerad tillsats" eller "Låg frekvens" om L2 > 18A.
2.  **Bastu-detektion:** Identifiera bastustart via energimönster och vidta åtgärder (t.ex. stoppa VVB-laddning).

### Fas 3: Smart Styrning (Nord Pool)
När Fas 1 & 2 är klara aktiverar vi besparingen.
1.  **Pre-heating:** Öka värmekurvan (+2) under billiga timmar (natt) för att lagra värme i plattan.
2.  **VVB-optimering:** Blockera varmvattenproduktion under dyra timmar (07-11, 17-19).

---

## 4. Nuvarande Verktygslåda (Scripts)
Dessa script ligger i `~/Git/System/energy_analysis/scripts/` och är redo att användas:

| Script | Syfte | Status |
| :--- | :--- | :--- |
| `fetch_holistic_data.py` | Hämtar all data (El, Temp, Pris) till CSV | ✅ Master |
| `analyze_phase_balance.py` | Hittar maxlaster och obalans | ✅ Klar |
| `fuse_simulator.py` | Beräknar "Headroom" mot 16A säkring | ✅ Klar |
| `heat_pump_performance.py` | Beräknar "Harvested Heat" & COP-proxy | ✅ Klar |
| `analyze_cost_efficiency.py` | Hittar dyraste timmarna | ✅ Klar |
| `populate_nordpool.py` | Laddar hem historiska elpriser | ✅ Klar |

---

### Data Scientistens Kommentar:
> "Vi har gått från att undra *om* säkringen håller, till att veta exakt *varför* den är nära att gå. Klippfisken har goda förutsättningar (bra luftflöde, ledig kapacitet på L1), men kräver ombalansering innan vi kan trycka gasen i botten med prisstyrning."

**Hur vill du gå vidare? Ska vi börja skissa på Home Assistant-automationen för "Säkringsvakt", eller vill du ha hjälp att tolka cirkulationspumpens inställningar?**