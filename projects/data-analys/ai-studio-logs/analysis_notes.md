---
Relaterad kod: "[[energy_analysis.py]]"
"tags:   - python   - influxdb   - energy-optimization":
---
Det är en mycket skarpsynt observation, Tomas. Som din Senior Data Scientist ska jag reda ut begreppen här, för det är en vanlig fallgrop i energianalys.

Svaret är: **Ja och nej**, beroende på vad vi vill räkna ut.

### Filplacering för dokumentation
Spara denna tekniska förklaring här för framtida referens:
`~/Git/System/energy_analysis/docs/theory_rms_vs_phase.md`

Logga insikten i projektloggen:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/analysis_notes.md`

---

### 1. Varför RMS är "rätt" för säkringsanalys
Du har helt rätt i att värdena du får ut (t.ex. från din Nibe eller en Shelly) är **RMS (Root Mean Square)**. 

För din **huvudsäkring** är det precis detta värde som räknas. En säkring bryter baserat på den termiska effekten av strömmen som passerar genom den. Eftersom RMS-värdet representerar den "likströms-ekvivalenta" strömmen som skulle ge samma värmeutveckling, behöver vi **inte** bry oss om fasförskjutningen ($\cos \phi$) mellan ström och spänning när vi analyserar om säkringen håller.

*   **Slutsats:** Om L1_Current_A visar 15.5A och du har en 16A säkring, då är du nära gränsen, oavsett om lasten är reaktiv (motorer) eller resistiv (element).

### 2. När vi INTE kan addera dem rakt av
Här kommer det viktiga "men":et. Det finns två scenarier där vi måste vara försiktiga:

#### A. Total effekt (Watt)
Om du vill räkna ut den totala effekten ($P$) i huset:
$$P_{tot} = (U_{L1} \cdot I_{L1} \cdot \cos \phi_1) + (U_{L2} \cdot I_{L2} \cdot \cos \phi_2) + (U_{L3} \cdot I_{L3} \cdot \cos \phi_3)$$
Här spelar fasförskjutningen ($\cos \phi$) roll. Om du bara tar $230V \cdot (L1+L2+L3)$ får du fram **S (Skenbar effekt)** i VA (Volt-Ampere), inte **P (Aktiv effekt)** i Watt. För en sommarstuga med mycket värmeelement (resistiv last) är $\cos \phi$ nära 1.0, så felet blir litet, men med värmepumpar och växelriktare kan det skilja sig.

#### B. Strömmen i Neutralledaren (N)
I ett trefassystem ligger faserna 120 grader förskjutna mot varandra. 
*   Om du belastar L1, L2 och L3 med exakt 10A var, blir strömmen i neutralledaren **0A**, inte 30A. 
*   De "slår ut" varandra tack vare 120-gradersförskjutningen.

### 3. Hur vi hanterar detta i din analys
För vårt mål – **trefasbalansering och säkringsoptimering** – är din nuvarande metod perfekt. Vi kommer att fokusera på:

1.  **$\max(L1, L2, L3)$:** Vilken enskild fas är närmast sin gräns? (Detta är det viktigaste för att slippa gå ut i snöslasket och byta säkring).
2.  **Obalans-index:** Hur stor är skillnaden mellan högsta och lägsta fas?
    *   *Hög obalans:* Du har ledigt utrymme på en fas men riskerar att skjuta en annan. Här kan vi föreslå att flytta laster.

---

### Sammanfattning för din kod
Du kan tryggt fortsätta använda dina RMS-värden. När vi summerar `Total_Current_A` i skriptet, ser vi det som en "belastningsindikator" för hela huset, men vi kommer alltid att hålla ett extra öga på den **högst belastade enskilda fasen**.

**Är du redo att köra en analys på din CSV-fil för att se vilken av dina faser som är "flaskhalsen" i fritidshuset?**