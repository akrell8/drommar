Tack för manualerna, Tomas. Det här är "the smoking gun". Genom att korsreferera din InfluxDB-data med de tekniska specifikationerna i manualerna för **NIBE F750** och **Tylö Sense Combi 8kW**, kan vi nu exakt diagnostisera varför din säkring riskerar att ryka.

### Filplacering för dokumentation
Spara denna tekniska analys här:
`~/Git/System/energy_analysis/docs/sauna_nibe_conflict_analysis.md`

Logga insikten i projektloggen:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/sauna_diagnosis.md`

---

### 1. Diagnos: Bastuaggregatet (Tylö Sense Combi 8kW)
Enligt manualen (sidan 9, Tabell för 400-415 V 3N~):
*   Ett fungerande 8kW aggregat ska dra **12A per fas** vid trefasanslutning.
*   **Din data visar:** L1 = 1.7A, L2 = 21.2A, L3 = 12.7A.

**Slutsats:** Din bastu är **inte** balanserat inkopplad som trefas, eller så är ett av de tre rörelementen i bastun trasigt. 
*   **L3** drar ~12A, vilket stämmer med manualen.
*   **L2** drar ~21A, vilket tyder på att den bär en dubbel last eller att andra tunga förbrukare (kök/Nibe) ligger här.
*   **L1** drar nästan ingenting (1.7A), vilket betyder att bastun helt lämnar L1 orörd.

### 2. Diagnos: NIBE F750 och Effektvakten
Enligt manualen (sidan 22-23):
*   F750 har en inbyggd effektvakt som läser dina drosslar. Den kan styra **elpatronen (eltillsatsen)** i steg om 0.5 kW upp till 6.5 kW.
*   **Problemet:** Effektvakten kan bara styra *sin egen* elpatron. Den kan inte stänga av kompressorn direkt (den varvar bara ner i nödfall) och den kan absolut inte styra din bastu.

**Varför gick strömmen inte?**
Vid peaken på **21.2A** på L2 (32% överlast) höll säkringen tack vare att det är en trög säkring. Men om Nibe-pumpen samtidigt hade velat köra en legionellakörning eller om någon startat en vattenkokare på L2, hade det blivit svart direkt.

### 3. Konflikten: Nibe vs Bastu
Nibe F750 lägger sin kompressordrift (upp till 13A) främst på **L1** (se manual s. 22). 
*   **Detta är goda nyheter!** Eftersom din bastu helt verkar undvika L1, finns det utrymme för Nibe att köra sin kompressor där.
*   **Detta är dåliga nyheter:** Om Nibe-pumpens elpatron (eltillsats) kliver in för att stötta vid -5°C, försöker den fördela stegen på de faser som har plats. Men om bastun redan "stjäl" 21A på L2, kommer effektvakten i Nibe blockera elpatronen helt på den fasen. Huset kan då få svårt att hålla värmen trots att du har 14A ledigt på L1.

---

### Strategisk rekommendation

Du bör visa detta för din elektriker. Här är vad som behöver göras:

1.  **Balansera Bastun:** Kontrollera om bastun är kopplad som 2-fas eller om ett element är trasigt. Den **ska** dra 12A på L1, L2 och L3. Om vi får över 12A från L2 till L1, sjunker din peak från 21A till ca 13-14A.
2.  **Fasväxling:** Om bastun av någon anledning *måste* vara 2-fas, se till att den ligger på **L1 och L3**, eftersom Nibe-pumpens tyngsta interna laster (eltillsats-steg) ofta konfigureras för att prioritera L2 när kompressorn går hårt på L1.

---

### Nästa steg: Python-skript för "Säkrings-simulator"
För att vara helt säker, låt oss skriva ett skript som räknar ut hur mycket "Headroom" du har kvar på din 16A-säkring per fas under hela månaden.

```python
import pandas as pd
import os

INPUT_PATH = os.path.expanduser("~/Git/Visioner/projects/data-analys/data/phase_export_last_month.csv")
FUSE_LIMIT = 16

def analyze_headroom():
    df = pd.read_csv(INPUT_PATH)
    
    for p in ['L1_Current_A', 'L2_Current_A', 'L3_Current_A']:
        # Hur många Ampere är kvar till 16A? (Negativt betyder överlast)
        df[f'{p}_Headroom'] = FUSE_LIMIT - df[p]
        
    # Hitta de 10 mest kritiska tidpunkterna totalt sett
    df['Min_Headroom'] = df[['L1_Headroom', 'L2_Headroom', 'L3_Headroom']].min(axis=1)
    critical_events = df.sort_values(by='Min_Headroom').head(10)
    
    print("DE 10 MEST KRITISKA TILLFÄLLENA (Minsta marginal till 16A):")
    print(critical_events[['Timestamp', 'L1_Current_A', 'L2_Current_A', 'L3_Current_A', 'Min_Headroom']])

if __name__ == "__main__":
    analyze_headroom()
```

**Tomas, om du kör detta skript, ser du då att det bara är vid bastubad som marginalen (Min_Headroom) blir negativ, eller händer det även vid andra tider?**