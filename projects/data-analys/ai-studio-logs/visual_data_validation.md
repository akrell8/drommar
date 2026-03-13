---
Relaterad kod:
  - Visioner/projects/data-analys/plots/air_performance.png
  - System/energy_analysis/scripts/plot_air_performance.py
---
![/Users/tomasberg/Git/Visioner/projects/data-analys/plots/air_performance.png](file:///Users/tomasberg/Git/Visioner/projects/data-analys/plots/air_performance.png)Tomas, det här ändrar allt! Graferna ger oss en helt annan – och mycket mer positiv – bild än de råa medelvärdena. Det här är ett skolboksexempel på varför visualisering är livsviktigt i Data Science.

Här är min reviderade analys baserad på din "facit-graf".

### Filplacering för logg
Spara denna analys här:
`~/Git/Visioner/projects/data-analys/ai-studio-logs/visual_data_validation.md`

---

# Visuell Validering: "Klippfisken" mår bättre än väntat

## 1. Luftflödet: Ingen kris, tvärtom!
*   **Observation:** Din gröna kurva ligger stabilt runt **160 m³/h**. 
*   **Slutsats:** Det tidigare snittvärdet på 67 m³/h var ett mätfel orsakat av de perioder där datan saknades (de raka strecken i grafen). 
*   **Insikt:** 160 m³/h är ett mycket bra flöde för en F750 (dubbelt mot minimikravet). Maskinen får all "mat" den behöver. De skarpa dyken till noll är sannolikt korta avbrott i loggningen eller när fläkten stannar vid avfrostning.

## 2. Kyleffekten: Mycket effektiv
*   **Observation:** Den orangea linjen (frånluft) är stabil vid ~19°C. Den blå linjen (avluft) dyker regelbundet ner till **-10°C till -12°C**.
*   **Slutsats:** Din pump fungerar utmärkt! Den lyckas krama ur ca 30 graders temperaturskillnad ur luften. 
*   **Varför var snittet -1.5°C?** Grafen visar att avluften "hoppar" upp till +10°C med jämna mellanrum. Detta är dina **avfrostningscykler**. Eftersom avfrostningarna är ganska många och långa, drog de upp det matematiska medelvärdet, men under drift är kylan imponerande.

## 3. Anomalier i grafen
*   **Feb 18 & 19:** Titta på de orangea topparna (upp mot 24°C). Det här är dina bastubad! Det är fascinerande att se hur hela husets frånluftstemperatur stiger med 4-5 grader när du bastar.
*   **Raka diagonala streck:** Dessa bekräftar att vi har haft luckor i datainsamlingen. InfluxDB "drar ett streck" mellan sista kända värde och nästa, vilket förvirrar statistiken.

---

# Reviderad Strategi: Fokus på Vatten och Pris

Nu när vi vet att luftsidan är frisk, kan vi fokusera på de två verkliga optimeringspunkterna:

### 1. Cirkulationen (Delta-T)
Du bekräftade att Delta-T på vattenkretsarna är lågt (ca 2°C). 
*   **Analys:** Eftersom luftflödet är så högt (160 m³/h) producerar pumpen massor av värme, men vattenkretsen "hinner inte dumpa" värmen i huset innan vattnet kommer tillbaka till pumpen.
*   **Åtgärd:** Sänk hastigheten på cirkulationspumparna. Vi vill se den blå och orangea linjen i vattenkretsarna glida isär mer.

### 2. Nord Pool-flytt
Eftersom pumpen kör så hårt (många avfrostningar), vill vi flytta denna intensitet till timmar där elen är billig.

---

### Nästa steg: Skapa "Pris-vinst-matrisen"

Jag vill nu att vi kör Nord Pool-skriptet (från förra svaret) mot den här datan. Jag vill se om pumpen avfrostar oftare under dyra timmar.

**Gör detta:**
1.  Kör `fetch_nordpool_data.py`.
2.  När det är klart, kör detta nya analysskript för att se din **"Kostnad per Gradminut"**.

**Spara som:** `~/Git/System/energy_analysis/scripts/analyze_cost_efficiency.py`

```python
import pandas as pd
import os

PATH = os.path.expanduser("~/Git/Visioner/projects/data-analys/data/holistic_energy_data.csv")

def analyze_cost():
    df = pd.read_csv(PATH)
    df['Timestamp'] = pd.to_datetime(df['Timestamp'])
    
    # Vi tittar endast på rader där vi har elpris
    df = df.dropna(subset=['Electricity_Price_SEK_kWh'])
    
    # Gruppera per timme för att se när det är dyrast att köra pumpen
    df['Hour'] = df['Timestamp'].dt.hour
    hourly_stats = df.groupby('Hour').agg({
        'Total_Power_W': 'mean',
        'Electricity_Price_SEK_kWh': 'mean',
        'Cost_Per_Minute_SEK': 'sum'
    })

    print("-" * 50)
    print("VAR ÄR PENGARNA? (Timvis analys)")
    print("-" * 50)
    print(hourly_stats.sort_values(by='Cost_Per_Minute_SEK', ascending=False).head(5))
    print("\nDetta är de 5 dyraste timmarna för din värmepump just nu.")

if __name__ == "__main__":
    analyze_cost()
```

**Tomas, kör Nord Pool-integreringen och sen detta kostnadsskript. Vilka timmar dyker upp som de dyraste?** (Det är dessa timmar vi ska "radera" med Home Assistant-automatiseringar sen!)