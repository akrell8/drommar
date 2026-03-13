Plats: `~/Git/Visioner/projects/data-analys/ai-studio-logs/infrastructure_p1_solution.md`

Det här är en klassisk "Edge Computing"-utmaning, Tomas.

**Lägesanalys:**
1.  **Avstånd:** 70 meter är för långt för P1-signalen (TTL) att färdas direkt i en kabel utan förstärkning. Den dör efter 5-10 meter.
2.  **Hinder:** Ett plåtskåp är en Faradays bur. En Wi-Fi-dongel (Tibber/HomeWizard) inuti skåpet kommer ha mycket svårt att nå huset (och även verkstaden, beroende på väggar).
3.  **Infrastruktur:** Du har kablar (trasig Ethernet + hel Larmkabel) mellan Huset och Verkstaden. Verkstaden ligger 10m från mätaren.

Här är mina tre lösningsförslag, rangordnade efter "Ingenjörsmässig robusthet".

---

### Lösning 1: "MacGyver-lösningen" (RS-485 via Larmkabel) – **REKOMMENDERAS**
Eftersom du har en mångpolig tvinnad larmkabel med lediga par är detta den absolut stabilaste lösningen. Vi konverterar signalen till industristandarden RS-485 som klarar över 1000 meter.

**Arkitektur:**
`Mätare (P1)` -> `Kort sladd (RJ12)` -> `TTL-till-RS485` -> **Larmkabel (70m)** -> `RS485-till-USB` -> `Home Assistant (Huset)`

**Du behöver:**
1.  **Vid mätaren (eller i verkstaden om du drar kabel dit):** En P1-läsare som kan skicka data seriellt, eller en enkel krets som konverterar P1-signalen till RS-485.
2.  **Kabel:** Använd ett tvinnat par i larmkabeln (t.ex. Blå + Blå/Vit) för signalen A/B.
3.  **Hemma:** En USB-RS485-sticka in i din server.

*Fördel:* Ingen elektronik som hänger sig ute i stolpen. Ingen Wi-Fi som strular. 100% trådburet.

### Lösning 2: "Rädda Ethernet-kabeln" (För Wi-Fi i verkstaden)
Om du vill använda en standardlösning (t.ex. HomeWizard eller Tibber) måste vi få Wi-Fi till verkstaden. Vi kan försöka "rädda" din Ethernet-kabel.

Gigabit Ethernet (1000 Mbps) kräver alla 8 trådar. Men **Fast Ethernet (100 Mbps)** kräver bara 4 trådar (Par 1, 2, 3 och 6).
*   **Hack:** Om kortslutningen ligger i det blå eller bruna paret, kan du kontaktera om (krimpa om) kabeln i båda ändar och bara använda **Orange** och **Grön** par.
*   **Resultat:** Du får ett fungerande nätverksuttag i verkstaden. Sätt en Access Point där.
*   **Mätaren:** Sätt en HomeWizard P1 i plåtskåpet. Om signalen är för svag, köp en enkel USB-förlängare eller RJ12-förlängare och dra ut dongeln utanför plåtskåpet (i en plastdosa).

### Lösning 3: ESPHome via Larmkabel (Strömförsörjning + Data)
Om du är bekväm med lödkolv och ESPHome (vilket jag tror du är):

1.  **I Verkstaden:** Placera en ESP32 (t.ex. en SlimmeLezer-klon).
2.  **Mellan Verkstad och Mätare (10m):** Dra en enkel telekabel/nätverkskabel (RJ12). 10 meter *kan* fungera för P1-signalen om kabeln är bra, men det är på gränsen.
3.  **Ström:** Mata ESP32:an i verkstaden med en vanlig USB-laddare.
4.  **Data:** Koppla ESP32:an till ditt nätverk (antingen via det "lagade" Ethernetet eller via Larmkabeln om du kör seriell överföring).

---

### Min Strategiska Rekommendation

Eftersom du har "lediga trådar" i larmkabeln som går hela vägen in i routercentralen:

1.  **Bygg Lösning 1 (RS-485).** Det är extremt billigt (två små moduler för 50 kr styck) och bombsäkert.
2.  Du slipper bråka med Wi-Fi-täckning i ett plåtskåp.
3.  Du får in datan direkt via USB i Home Assistant som en seriell sensor ("DSMR"-integrationen).

**Vill du att jag tar fram en inköpslista och kopplingsschema för RS-485-lösningen via larmkabeln?**