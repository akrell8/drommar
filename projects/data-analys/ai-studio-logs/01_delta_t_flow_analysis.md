---
title: "Diagnos: För högt flöde i värmesystemet (Delta-T Analys)"
date: 2026-02-27
tags:
  - energy-optimization
  - heat-pump
  - nibe-f750
  - easy-wins
  - delta-t
related:
  - "[[plot_heating_delta.py]]"
  - "[[00_project_baseline_v2.md]]"
---

# Diagnos: För högt flöde i värmesystemet (Delta-T)

Efter att ha kört och visualiserat flödesdatan för S1 (Radiatorer) och S2 (Golvvärme) kan vi konstatera att systemet lider av en klassisk "termisk jojo"-effekt. Detta är en fantastisk "easy win" eftersom det inte kräver några elinstallationer att åtgärda, utan endast omkonfigurering av befintliga cirkulationspumpar.

>[!danger] Kärnproblemet
> Vattnet pumpas runt i huset alldeles för snabbt. Det hinner inte avge sin värme till rummen innan det skickas tillbaka in i värmepumpen.

## 📊 Observationer från datan

| Krets | Uppmätt Snitt (ΔT) | Målvärde (ΔT) | Status |
| :--- | :--- | :--- | :--- |
| **S1 (Radiatorer)** | 2.56 °C | 5.0 - 10.0 °C | ❌ För lågt / För snabbt flöde |
| **S2 (Golvvärme)** | 1.83 °C | 4.0 - 6.0 °C | ❌ För lågt / För snabbt flöde |

*   **Visuell bekräftelse:** Grafen visar extrem taggighet och korta pulser. Returtemperaturen (in i pumpen) ligger nästan klistrad mot framledningstemperaturen (ut ur pumpen).

## ⚙️ Konsekvens ("Den termiska jojon")
När returvattnet kommer tillbaka onormalt varmt, luras Nibe F750 att tro att huset är fullt uppvärmt.
1. Pumpen drar ner frekvensen (Hz) eller stannar helt.
2. Några minuter senare, eftersom huset egentligen inte blev uppvärmt i djupet (trögheten), sjunker temperaturen snabbt.
3. Gradminuterna faller och pumpen tvingas panikstarta igen.
*Detta sliter på kompressorn och förstör vår COP (verkningsgrad).*

---

## 🛠️ Action Plan: Operation "Lugna ner systemet"

Dessa justeringar bör göras omgående för att låta värmen "landa" i huset.

### Åtgärd 1: Radiatorkretsen (S1) - Intern pump
Justera Nibe F750:s inbyggda värmebärarpump (GP1).
- **Hur:** Håll inne "Tillbaka"-knappen i 7 sekunder för att nå Servicemenyn.
- **Var:** Gå till Meny `5.1.11` (Värmebärarpumpens hastighet).
- **Ändring:** Byt från `Auto` (eller nuvarande höga värde) till `Manuell` och ställ in på ca **60%**.

### Åtgärd 2: Golvvärmekretsen (S2) - Extern pump
Justera den externa cirkulationspumpen på shuntgruppen (ofta ECS 40/41, vit Grundfos eller röd Wilo).
- **Hur:** Leta upp pumpen fysiskt vid golvvärmefördelaren.
- **Var:** Reglage direkt på pumphuset.
- **Ändring:** Sänk hastigheten ett snäpp (t.ex. från läge III till II, eller stäng av AutoAdapt och välj ett fast lägre tryck).

---

## 📈 Förväntat resultat (Feedback Loop)
Efter 24 timmar ska vi köra extraktionen och plottningen igen. Vi förväntar oss att se:
1. **Utplanade kurvor:** Mindre "taggighet" och mjukare dygnsvariationer.
2. **Ökat gap (ΔT):** Avståndet mellan framledning (röd/blå) och retur (orange/cyan) i grafen ska öka tydligt.
3. **Ökad effektivitet:** Pumpen kommer att gå på en jämnare frekvens (Hz) över längre tidsperioder, vilket maximerar termisk utvinning.

> [!check] Nästa steg
> Genomför justeringarna fysiskt och låt systemet logga data i minst 24 timmar. Utvärdera därefter genom att generera en ny graf med `plot_heating_delta.py`.