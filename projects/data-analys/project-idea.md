# Projekt: AI-Driven Energioptimering (Fritidshus)
**Status:** 🏗️ Uppstart  
**Plattformar:** Home Assistant, InfluxDB, Grafana, Google AI Studio  
**Mål:** Sänka elkostnader, optimera värmepump och skapa intelligenta dygnsmönster.

---

## 📋 Övergripande Vision
Att transformera passiv loggdata från mitt fritidshus till aktiva insikter med hjälp av Large Language Models (LLM). Genom att använda **Google AI Studio** som analysmotor ska jag identifiera mönster som är svåra att se i vanliga grafer, såsom dolda laster och korrelationer mellan utetemperatur och trefasbelastning.

## 🛠️ Teknisk Miljökonfiguration

### 1. Datakälla: Home Assistant & InfluxDB
- **Sensorer:** Trefasström (Huvudmätare), Värmepump (Modbus/ESPHome), Larm, Temperatur (Inne/Ute).
- **Strategi:** Exportera aggregerad data (15-minuters eller 1-timmes intervall) till CSV för analys.
- **Viktiga fält:** `current_L1`, `current_L2`, `current_L3`, `power_kw`, `temp_outdoor`.

### 2. Analysmiljö: Google AI Studio
- **Modell:** Gemini 1.5 Pro (för stort kontextfönster och kodförståelse).
- **System Prompt (Draft v1):**
  > Du är en expert på energisystem och hemautomation. Din roll är att agera "Data Scientist" för mitt fritidshus. Analysera bifogad CSV-data för att hitta anomalier, optimera värmekurvor och föreslå schemaläggning av laster baserat på timspriser (Nord Pool). Svara alltid med teknisk precision och konkreta åtgärdsförslag.

---

## 📈 Utforskningsområden

### A. Effektbalansering (Trefas)
- **Problem:** Riskerar jag att lösa ut huvudsäkringen vid hög belastning?
- **AI-uppgift:** Analysera fasfördelningen över ett dygn. Föreslå ombalansering av laster eller logik för "Load Shedding" i Home Assistant.

### B. Termisk Tröghet & Värmepump
- **Problem:** Värmepumpen jobbar ofta när elen är som dyrast.
- **AI-uppgift:** Korrelera utetemperatur med elförbrukning. Kan vi "förvärma" huset när priset är lågt utan att innetemperaturen svänger för mycket?

### C. Val av Elleverantör
- **AI-uppgift:** Mata in en månads förbrukning och jämför kostnaden mellan ett fast pris, rörligt månadspris och timpris (spot). Vilket mönster passar min faktiska profil på landstället?

---

## 📝 Logg & Milstolpar
- [x] Projektet initierat.
- [ ] Första exporten av CSV-data från InfluxDB.
- [ ] Skapa "Energy Dashboard" i AI Studio med System Instructions.
- [ ] Generera första Flux-queryn för Grafana via AI.
- [ ] Utvärdera automatiseringsförslag för värmepump.

---

## 🔗 Resurser & Länkar
- [[InfluxDB Queries]] – *Mina mest använda Flux-script.*
- [[Grafana Dashboards]] – *Skärmdumpar och struktur.*
- [Google AI Studio](https://aistudio.google.com/)