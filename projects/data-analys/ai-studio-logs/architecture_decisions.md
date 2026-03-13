# Arkitekturbeslut - Energianalys

## Miljökonfiguration (2023-10-XX)
- **Datakälla:** InfluxDB körs på `rpi-klippfisken-1`.
- **Analysmiljö:** Lokal Mac.
- **Anslutning:** Python-skript ansluter via `influxdb-client` över lokalt nätverk.
- **Bibliotek:** Pandas för datatvätt, Influxdb-client för extraktion.