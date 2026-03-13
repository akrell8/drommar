Plats: `~/Git/System/energy_analysis/docs/hardware_build_rs485_p1.md`

Utmärkt. Dokumentationen från Vattenfall bekräftar att du har en **Sanxing S34U18**. Det är goda nyheter:
1.  Den följer standarden (RJ12).
2.  Den ger 5V ut (Pin 1).
3.  Den pratar 115 200 baud.

Här är din **Inköpslista** och **Bygginstruktion** för en lösning som är billig, skalbar och robust.

---

### 1. Inköpslista (AliExpress)

Sök på dessa termer. Priserna är ungefärliga (exkl frakt/moms). Köp gärna flerpack (t.ex. 5 st) då det är billigt och bra att ha reservdelar om röken slipper ut.

| Komponent | Sökord på AliExpress | Funktion | Pris (ca) |
| :--- | :--- | :--- | :--- |
| **Sändare (Stolpen)** | `MAX485 module TTL to RS485` | Gör om 5V-signal till RS-485. (Den blåa klassikern). | 10 kr |
| **Mottagare (Inne)** | `USB to RS485 converter` | Den svarta lilla stickan med skruvplint. | 20 kr |
| **Invertering** | `BC547 transistor` + `Resistor kit` | Vänder signalen rätt (se nedan). | 20 kr (för 100 st) |
| **Kabel** | `RJ12 6P6C cable` | Klipp en gammal telekabel eller köp ny. | 15 kr |
| **Koppling** | `Prototype PCB board` | Hålad platta att löda på (valfritt, men snyggt). | 10 kr |
| **Låda** | `Plastic waterproof enclosure` | Liten plastlåda att ha vid mätaren. | 30 kr |

**Totalt:** Under 150 kr.

---

### 2. Det Tekniska Problemet: "Inverterad Signal"
Detta är den enda fällan. P1-porten på din Sanxing skickar en **inverterad signal** (0V är en "Etta", 5V är en "Nolla").
*   En vanlig RS-485-modul förväntar sig tvärtom.
*   Om vi kopplar in den direkt kommer datan bli skräp (framing errors).
*   **Lösning:** Vi bygger en enkel "signalvändare" med en transistor (BC547) och två motstånd innan signalen går in i RS-485-modulen.

---

### 3. Kopplingsschema (Vid Stolpen)

Det här är kretsen du ska bygga och stoppa in i plåtskåpet. Den drivs helt av mätaren.

**RJ12 Pinout (Sanxing S34U18):**
*   **Pin 1:** +5V (Matning)
*   **Pin 2:** Data Request (Måste vara hög)
*   **Pin 5:** Data UT (Inverterad)
*   **Pin 6:** GND

**Kretsdiagram:**

```ascii
[RJ12 Från Mätare]                  [MAX485 Modul (Blå)]
                                   +-----------------+
PIN 1 (+5V) -----------------------| VCC             |
PIN 6 (GND) -------------------+---| GND             |
                               |   |                 |
PIN 2 (Request) ---[1k Ohm]----*   |                 | (Larmkabel 70m)
(Pin 2 behöver 5V              |   |                 |
 för att aktiveras)            |   |                 |   Par 1 (Tvinnat)
                               |   |                 |  +----------------
                               |   |               A |---| Blå tråd
        (Inverterings-krets)   |   |                 |   |
PIN 5 (Data) --[10k Ohm]--Base |   |               B |---| Blå/Vit tråd
                          (BC547)  |                 |  +----------------
                               |   |                 |
GND -------------------Emitter |   |                 |
                               |   |                 |
+5V ---[1k Ohm]-------Collector----| DI (Data In)    |
                                   |                 |
+5V -------------------------------| DE (Driver En)  | (Låst till Sändning)
+5V -------------------------------| RE (Recv En)    | (Låst till Sändning)
                                   +-----------------+
```

**Förklaring av bygget:**
1.  **Request:** Vi lägger 5V (via 1k motstånd) på Pin 2. Detta säger till mätaren: "Jag är här, skicka data!".
2.  **Invertering:** Transistorn (BC547) vänder signalen från Pin 5. När mätaren skickar Lågt, går transistorn Hög (in i MAX485).
3.  **Låsning:** Vi kopplar DE och RE på MAX485 till 5V. Det tvingar modulen att alltid vara i "Sändar-läge".

---

### 4. Kopplingsschema (I Källaren)

Här är det enkelt.

**Kabel:**
*   Blå tråd (A) ---> Terminal **A** på USB-stickan.
*   Blå/Vit tråd (B) ---> Terminal **B** på USB-stickan.
*   *(Om du inte får data: Byt plats på A och B. Inget går sönder, det funkar bara inte).*

**Terminering (Viktigt vid 70m):**
För att undvika "eko" i kabeln bör du sätta ett **120 Ohm motstånd** mellan A och B, både vid sändaren (stolpen) och mottagaren (USB-stickan). På korta sträckor behövs det inte, men vid 70m är det rekommenderat.

---

### 5. Konfiguration (Mjukvara)

När du byggt hårdvaran:

1.  **Sätt i USB-stickan** i din Raspberry Pi 2B+ (källaren).
2.  **Konfigurera `ser2net`** (som vi diskuterade tidigare):
    *   Fil: `/etc/ser2net.yaml`
    *   Baudrate: **115200** (Sanxing kör snabbt!).
    *   Parity: **None** (n).
    *   Databits: **8**.
    *   Stopbits: **1**.

```yaml
connection: &meter
  accepter: tcp,2001
  connector: serialdev,/dev/ttyUSB0,115200n81,local
```

3.  **Home Assistant:**
    *   Lägg till "DSMR Slimme Meter".
    *   Host: IP till din RPi 2.
    *   Port: 2001.

### Nästa steg
Detta är ett kul helgprojekt!
1.  Beställ grejerna från AliExpress ikväll (leverans 2-3 veckor).
2.  Medan du väntar: Har du en lödkolv och lite tenn?
3.  Vill du att vi ska simulera vad vi gör om lastbalanseringen *innan* grejerna kommer, baserat på din manuella data, eller tar vi en paus i projektet tills hårdvaran är på plats?