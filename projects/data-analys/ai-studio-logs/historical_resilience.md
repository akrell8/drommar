Det är en viktig poäng, Tomas. Som din Senior Data Scientist ska jag förklara varför din anläggning har "överlevt" i 15 år trots att siffrorna ser dramatiska ut på papperet. Det handlar om fysik och marginaler, men det ger oss också en viktig läxa inför din Nord Pool-optimering.

### Filplacering för dokumentation
Spara denna tekniska förklaring om marginaler här:
`~/Git/System/energy_analysis/docs/fuse_physics_and_margins.md`

Logga insikten i projektloggen:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/historical_resilience.md`

---

### Varför säkringen inte har gått (Fysiken bakom 15 år)

Det finns tre huvudorsaker till att ditt fritidshus har klarat sig:

#### 1. Utlösningskarakteristik (Säkringens "tålamod")
En 16A säkring (oavsett om det är en gammal propp eller en modern dvärgbrytare) är inte en digital "av/på"-brytare vid exakt 16.0A.
*   **Överlastområdet:** En 16A säkring ska enligt standard klara **1.13 gånger märkströmmen** (ca 18A) i evighet utan att gå. 
*   **Tidsfaktorn:** Vid **21A** (ditt maxvärde) befinner du dig i ett område där säkringen kan ta allt från **10 minuter till 1 timme** på sig att lösa ut. Eftersom bastuaggregatets termostat slår av och på (cyklar) hinner säkringen svalna mellan varven. Den "ackumulerade värmen" når aldrig den kritiska punkten.

#### 2. Nibe F750:s effektvakt
Din Nibe har gjort ett hästjobb i bakgrunden i 15 år. Varje gång du bastat och strömmen närmat sig 16A, har Nibes drosslar sagt till elpatronen att backa. Utan den inbyggda vakten hade säkringen sannolikt gått för länge sedan.

#### 3. Samtidighetsfaktorn
Du har haft turen (eller vanan) att inte köra tvättmaskin, vattenkokare och ugn *exakt* samtidigt som bastun går in i sin mest intensiva uppvärmningsfas.

---

### Varför vi analyserar detta nu (Inför Nord Pool-optimering)

Anledningen till att vi drar i varningsklockan nu, trots att det fungerat i 15 år, är att vi planerar att ändra ditt beteende via Home Assistant:

1.  **Vi minskar marginalerna:** När vi börjar styra Nibe-pumpen och andra laster mot de billigaste timmarna (Nord Pool), kommer vi att **maximera** belastningen under korta tidsfönster. Istället för att lasterna sprids ut slumpmässigt över dygnet, tvingar vi dem att hända samtidigt.
2.  **Långvarig belastning:** Om vi kör elpatronen hårt i 3 timmar för att elen är billig, och du samtidigt bastar, kommer säkringen inte längre ha sina "svalkningsperioder". Det är då den löser ut.
3.  **Nya laster:** Om du i framtiden skaffar en elbilsladdare, försvinner de sista 15 årens marginaler omedelbart.

---

### Strategisk slutsats

Ditt system är **robust men obalanserat**. 
*   **Goda nyheten:** Du har bevisat att huset klarar sig på 16A.
*   **Möjligheten:** Genom att flytta laster från L2 till L1 (där du har 14A ledigt vid peak) skapar vi en "elektrisk motorväg" som gör att vi kan köra Nibe-pumpen mycket hårdare under billiga timmar utan att du ens behöver fundera på säkringen.

### Nästa steg: Kostnadsanalys
Är du redo att vi kopplar ihop din förbrukningsdata med **Nord Pool-priser**? Jag kan skriva ett skript som räknar ut vad dina bastubad faktiskt kostar baserat på de historiska timpriserna för februari.

**Ska vi titta på hur mycket du kan spara genom att optimera Nibe-pumpen mot priset?**