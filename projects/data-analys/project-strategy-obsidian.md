# ⚡ Projektstrategi: AI-Driven Energioptimering
**Dokumenttyp:** Projekt-MOC (Map of Content)
**Lokation:** `~/Git/Visioner/projects/data-analys/`

---

## 🎯 Övergripande Syfte
Att använda detta projekt som ett "Proof of Concept" för att integrera **AI Studio** i mitt arbetsflöde och använda **Obsidian** som navet som knyter ihop hem-it, teknisk analys och personliga mål.

## 🗺️ Obsidian-grinden (Kopplingar)
Detta projekt bryggar över mina tre huvudkontexter:

### 1. 🏗️ SYSTEM (Källa)
*Logik: Varifrån kommer datan?*
- **Mapp:** `~/Git/System/ha_klippfisken_1/`
- **Kopplingar:** [[Home Assistant Konfiguration]], [[InfluxDB-exporter]], [[Sensormappning]].
- **Uppgift:** Säkerställa korrekt dataexport (CSV) för analys.

### 2. 📚 STUDIER (Metod)
*Logik: Hur analyserar jag datan?*
- **Mapp:** `~/Git/Studier/`
- **Kopplingar:** [[Dataanalys-principer]], [[Energiberäkning Trefas]], [[Prompt Engineering]].
- **Uppgift:** Dokumentera lärdomar om AI-metodik och el-teknik.

### 3. 🔭 VISIONER (Mål & Strategi)
*Logik: Varför gör jag detta?*
- **Mapp:** `~/Git/Visioner/projects/data-analys/`
- **Kopplingar:** [[project-idea]], [[Fritidshus Strategi 2026]].
- **Uppgift:** Hålla i projektets framdrift och ai-loggar.

---

## 🤖 AI Studio Arbetsflöde
För att hålla ordning på utvecklingen i AI Studio används följande flöde:

1. **Input:** Exportera CSV från System-mappen.
2. **Process:** Analys i Google AI Studio (Gemini 1.5 Pro).
3. **Output:** Spara insikter och kodförslag i `../ai-studio-logs/` (Markdown).
4. **Integration:** Uppdatera [[project-idea]] med nästa steg.

---

## 📂 Filstruktur i Projektet
- `project-idea.md` - Ursprunglig vision.
- `project-strategy-obsidian.md` - Detta dokument.
- `ai-studio-logs/` - Loggfiler från AI-sessioner.
- `data-samples/` - Små test-set av CSV-data (tunga filer i iCloud-Arkivet).

---
**Status:** #projekt/initierat  
**Nästa steg:** Skapa första system-prompten i AI Studio.