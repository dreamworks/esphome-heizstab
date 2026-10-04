# Messprotokoll Heizstab – Material für den Bürgerblick-Artikel (Print)

Laufendes Protokoll mit Messwerten und Erfahrungen. Alle Werte stammen aus
Home Assistant (Wechselrichter RCT, Temperaturfühler am Heizkreis), sofern
nicht anders angegeben.

## Ausgangslage

- Alte Ölheizung, kein Öl mehr im Tank. PV-Anlage mit Hausbatterie vorhanden.
- Ziel: Heizkreis elektrisch mit einem Heizstab beheizen, später gezielt mit
  PV-Überschuss.
- Hardware: Heizlando MDC 230 (3 kW, Thermostat 30–75 °C, STB 85 °C),
  Fotek SSR-40DA, Steuerung über ESP32-Display (CYD) mit ESPHome.
  Leistung in Stufen 0/25/50/75/100 % per Burst-Fire (60 s Periode).
- Konfiguration und Doku: https://github.com/dreamworks/esphome-heizstab

## 2026-10-04 – Inbetriebnahme und Testheizen

### Zeitlicher Ablauf

| Uhrzeit | Ereignis |
|---|---|
| 18:07 | Heizstab erstmals eingeschaltet (Teillast, dann Volllast ab 18:15) |
| 18:20 | Heizstab aus – das ESP hat neu gestartet und steht danach immer auf 0 % |
| 19:23 | Wieder eingeschaltet, Volllast |
| ab 19:27 | ESP verliert wiederholt das WLAN, Heizstab läuft lokal weiter |

### Messwerte

- **Elektrische Leistung Heizstab:** ca. **2,8 kW** (Phase L1 ca. 3,2 kW
  abzüglich Grundlast ca. 0,44 kW). Nennleistung 3 kW.
- **Testheizen bis 20:19:** 65 min Heizzeit, **2,98 kWh** (aus dem
  Leistungsverlauf L1 integriert).
- **Rücklauftemperatur:** 19,6 °C (18:10) → 20,75 °C (18:27) → 24,3 °C (20:10).
  Anstieg bei Volllast ca. 5–6 K pro Stunde; der Rücklauf steigt auch nach dem
  Abschalten noch einige Minuten weiter → Wasser zirkuliert.
- Rechnerisch würden 2,2 kWh für +3,9 K etwa 480 l Wasser erwärmen. Ein großer
  Teil der Wärme geht also schon unterwegs ab (Heizkörper, Rohre, kalter Ölkessel).
- **Strom kam fast komplett aus der Hausbatterie:** PV am Abend ca. 0 W,
  Akku von 85 % (18:10) auf 49 % (20:10) bei ca. 3,4 kW Entladung.

### Kosten des Testheizens (2,98 kWh)

| Annahme | Preis pro kWh | Kosten |
|---|---|---|
| Strom aus PV/Akku, bewertet mit entgangener Einspeisevergütung | ca. 0,08 € (eigenen Wert einsetzen) | ca. 0,24 € |
| Strom aus dem Netz | ca. 0,35 € (eigenen Tarif einsetzen) | ca. 1,04 € |

### Vergleich mit Heizöl

- Heizöl am 04.10.2026: ca. **1,64 €/l** (heizoel24.de, 3.000 l Abnahme).
- 1 l Heizöl ≈ 10 kWh; alter Kessel ca. 85 % Wirkungsgrad → **ca. 0,19 € pro kWh Wärme**.
- Heizstab: 1 kWh Strom ≈ 1 kWh Wärme (über 99 %).
  - mit Netzstrom: **ca. 0,35 €/kWh Wärme** → etwa doppelt so teuer wie Öl
  - mit PV-Überschuss: **ca. 0,08 €/kWh Wärme** → deutlich günstiger als Öl
- Wichtig: Volllast gilt nur beim Aufheizen. Ist die Temperatur erreicht,
  taktet der Thermostat, und es wird nur noch die tatsächliche Heizlast des
  Hauses verbraucht. Diese wird in den nächsten Tagen gemessen.

### Zwischenstand 20:49

- **Heizzeit gesamt:** 92 min, **4,23 kWh** (aus dem Leistungsverlauf L1 integriert,
  Grundlast 436 W abgezogen)
- **Rücklauf:** 25,9 °C, also +6,3 K seit Beginn (19,6 °C um 18:10)
- **Akku:** 31,5 %
- **Kosten bisher:** ca. 0,34 € (bewertet mit Einspeisevergütung 0,08 €/kWh)
  bzw. ca. 1,48 € (mit Netzstrom 0,35 €/kWh)
- Um 20:33 hat der Thermostat des MDC 230 erstmals kurz abgeschaltet und nach
  etwa einer Minute wieder eingeschaltet. Vermutlich steht er nahe 30 °C.

### Zwischenstand 21:29

- **Heizzeit gesamt:** 131 min, **5,97 kWh**
- **Rücklauf:** 27,9 °C (+8,3 K seit 18:10)
- **Akku:** 12,6 %, Netzbezug noch 0 W
- **Kosten bisher:** ca. 0,48 € (Einspeisevergütung 0,08 €/kWh) bzw. ca. 2,09 €
  (Netzstrom 0,35 €/kWh)
- ESP wechselt alle paar Minuten zwischen online und offline (WLAN), der
  Heizstab läuft dabei ununterbrochen weiter.

### Abschluss des Abends

- **21:48–21:51:** Thermostat des MDC 230 schaltet erstmals für etwa 3 Minuten ab.
  Ab jetzt taktet der Heizstab, statt durchzulaufen.
- **ab ca. 21:45:** Akku bei 7 % (Untergrenze), danach Netzstrom
- **22:00:** Heizstab aus, weil das ESP zum Flashen vom Netzteil getrennt wurde
- **Bilanz Testheizen 18:07–22:00:** etwa **6,5 kWh** ins Heizwasser, Rücklauf
  von 19,6 °C auf 28,9 °C (+9,3 K)
- **22:06:** Firmware v1.2.0 per USB geflasht, ESP bucht sich beim Start in den
  stärksten Mesh-Zugangspunkt ein: **−61 dBm statt −77 bis −80 dBm**. Aber der
  freie Speicher sinkt auf **124 Bytes**, Home Assistant kann sich nicht verbinden.
- **22:15:** v1.3.0 ohne Bluetooth per USB geflasht. RAM-Belegung 26 % statt 58 %.
- **22:18:** ESP stabil in Home Assistant, WLAN −60 dBm.

**Ursache aller Verbindungsprobleme des Abends:** Das Display-Board (ESP32 ohne
Zusatzspeicher) war mit Bluetooth-Proxy, Display, WLAN und verschlüsselter
Verbindung überfordert. Dazu hing die erste Firmware an einem schwachen
Zugangspunkt fest. Für den Artikel: Ein Board, das ein Display treibt, sollte
nicht nebenbei noch Bluetooth-Proxy spielen.

### Probleme und Erkenntnisse

- **WLAN im Heizungsraum zu schwach:** −77 bis −80 dBm. Dazu kam ein
  Bluetooth-Dauerscan der ersten Firmware-Version. Folge: Das ESP verliert
  ständig das WLAN, Updates über WLAN (OTA) scheitern – auch eine extra kleine
  Rettungs-Firmware. Lösung: einmal per USB flashen, WLAN verbessern.
- **Vorlauffühler defekt:** Er meldet konstant 85,0 °C (typischer Fehlerwert
  eines DS18B20 bei fehlender Versorgung) bzw. gar nichts. Verdacht:
  Wackelkontakt an der Versorgungsader.
- **Sicherheitsverhalten bestätigt:** Nach jedem Neustart steht der Heizstab
  auf 0 % und heizt nicht von selbst wieder los. Fällt das WLAN aus, läuft er
  mit der zuletzt gewählten Stufe weiter und ist nur noch am Display bedienbar.

### Offene Punkte

- [ ] Firmware v1.2.0 per USB aufspielen
- [ ] WLAN im Heizungsraum verbessern (Ziel: −70 dBm oder besser)
- [ ] Vorlauffühler reparieren
- [ ] FI-Prüftaste testen, Erstprüfung durch Elektrofachkraft
- [ ] Temperatur SSR-Kühlkörper unter Volllast prüfen
- [ ] Fragen an den Heizungsbauer (Martin): Durchfluss/Pumpenstufe, Ölkessel
      absperren, Thermostat-Einstellung, Entlüften/Anlagendruck
- [ ] Tatsächliche Heizlast im Normalbetrieb messen (kWh pro Tag)
- [ ] Automation „Heizstab nur bei PV-Überschuss“
- [ ] Eigenen Strompreis und Einspeisevergütung eintragen
