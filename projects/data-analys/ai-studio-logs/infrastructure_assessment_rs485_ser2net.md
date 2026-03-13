Plats: `~/Git/System/energy_analysis/docs/infrastructure_assessment_rs485_ser2net.md`

Tomas, det är helt rätt inställning. Att vara skeptisk till "AI-hallucinationer" är sunt, särskilt när det gäller infrastruktur som ska hålla i åratal.

Här är min brutala granskning av lösningsförslaget.

### 1. Är detta en "AI-uppfinning"?
**Svar: Nej.**

Den här lösningen bygger inte på någon ny, experimentell teknik. Tvärtom är det en kedja av **industriella standarder** som använts inom automation och fastighetsstyrning sedan 90-talet.

*   **RS-485 (TIA-485-A):** Standarden för trådbunden kommunikation i bullriga industrimiljöer. Den används i allt från kärnkraftverk till hissar för att skicka data långa sträckor (upp till 1200m). Att använda detta över 70m larmkabel är "best practice".
*   **ser2net (RFC 2217):** Ett standardverktyg i Linux-världen (skapat 2001) för att göra serieportar tillgängliga över nätverk. Det används i datacenter för "Console Servers" över hela världen.
*   **DSMR över TCP:** Home Assistants integration har inbyggt stöd för nätverksanslutning just för att mätaren ofta inte sitter där servern står.

**Slutsats:** Detta är en **"Old School"-lösning**. Den är tråkig, beprövad och robust. Ingen "bleeding edge"-magi.

### 2. Riskbedömning & Svaga punkter

Här är de faktiska riskerna du behöver hantera:

| Riskfaktor | Sannolikhet | Konsekvens | Mitigering (Lösning) |
| :--- | :--- | :--- | :--- |
| **Signalinvertering** | Hög | Ingen data | Svenska mätare (P1) skickar ofta "inverterad" signal (0 är 1, 1 är 0). En standard RS-485-modul kan behöva en extra transistor eller en mjukvaru-invertering. **Köp en modul som anger "P1 compatible" eller var beredd på en liten lödning.** |
| **SD-kortskrasch** | Medel | Systemstopp | Din RPi 2B+ körs på SD-kort. Vid strömavbrott kan kortet paja. **Lösning:** Kör "Overlay FS" (Read-only filsystem) eller ha en backup-image. |
| **Reflektioner** | Låg (vid 70m) | Databrus | Vid långa kablar kan signalen "studsa". **Lösning:** Sätt ett 120 ohm motstånd (terminering) mellan A och B i båda ändarna. |
| **Jordloopar** | Låg | Komponentdöd | Om mätaren och RPi 2B+ har olika jordpotential kan ström gå genom skärmen. **Lösning:** Använd tvinnat par för signal (A/B) men koppla INTE ihop jorden (GND) stumt i båda ändar om det inte behövs. RS-485 är differentiell och klarar sig ofta utan gemensam jord över korta avstånd, men med 70m bör man vara försiktig. |

---

### 3. Arkitektur & Lösningsförslag

Spara denna specifikation så du har ritningen klar.

#### Topologi
```ascii
[Elmätare (Stolpen)] 
      |
      | (RJ12 - P1 Port)
      v
[TTL-till-RS485 Modul] <--- Strömförsörjs av mätaren (5V)
      |
      | (Larmkabel 70m - Tvinnat par: Blå & Blå/Vit)
      | (Signal: Data A & Data B)
      v
[USB-RS485 Adapter]
      |
      | (USB)
      v
[Raspberry Pi 2B+ (Källaren)]
      |
      | (Ethernet via ny Switch)
      v
[LAN / Router]
      |
      | (WiFi/Ethernet)
      v
[Home Assistant (Värmepumpen)] <--- Läser data via TCP port 2001
```

#### Komponentlista (Inköp)

1.  **Vid Mätaren (Sändare):**
    *   1x **RJ12-kabel** (klipps).
    *   1x **TTL till RS-485 modul** (med *automatisk flow control* är bäst, t.ex. MAX13487, men billiga MAX485 funkar ofta).
    *   *Option:* 1x BC547 transistor + 1k/10k motstånd (om signalen behöver inverteras manuellt).
2.  **Kabeln:**
    *   Befintlig larmkabel.
    *   2x **120 Ohm motstånd** (för terminering i vardera ände).
3.  **I Källaren (Mottagare):**
    *   1x **USB-RS485 sticka** (gärna med FTDI-chip eller CH340).
    *   1x **Nätverksswitch** (5-portars Gigabit eller 10/100).
    *   Din befintliga **RPi 2B+** + SD-kort (4GB+ räcker).

#### Mjukvarukonfiguration (RPi 2B+)

Installera **Raspberry Pi OS Lite**.
Kör följande kommandon:

```bash
sudo apt update && sudo apt install ser2net
```

Redigera `/etc/ser2net.yaml`:
```yaml
connection: &p1-meter
  accepter: tcp,2001
  connector: serialdev,/dev/ttyUSB0,115200n81,local
```

Starta om tjänsten:
```bash
sudo systemctl restart ser2net
```

#### Konfiguration (Home Assistant)
*   Lägg till integration: **DSMR (Slimme Meter)**.
*   Connection type: **Network**.
*   Host: `<IP till RPi 2B+>`.
*   Port: `2001`.
*   DSMR Version: `5B` (Standard för moderna svenska mätare).

---

### 4. Senior Data Scientistens råd
Detta är den **enda** lösningen som kombinerar dina befintliga resurser (Larmkabel + RPi 2) med industriell tillförlitlighet (RS-485).

Alternativen är sämre:
*   *WiFi-extenders:* Opålitligt i plåtskåp.
*   *Scraping (Vattenfall):* För långsamt.
*   *Köra utan RS-485:* Signalen dör efter 10 meter.

**Go/No-Go:** Jag ger denna arkitektur grönt ljus. Det är ett gediget ingenjörsprojekt, inte ett AI-hack.

**Vill du att jag tar fram ett exakt kopplingsschema för transistor-inverteringen (om det skulle behövas för din mätare)?**