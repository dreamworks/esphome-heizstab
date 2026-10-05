# Messprotokoll Heizstab – Material für den Bürgerblick-Artikel (Print)

Laufendes Protokoll mit Messwerten und Erfahrungen. Alle Werte stammen aus
Home Assistant (Wechselrichter RCT, Temperaturfühler am Heizkreis), sofern
nicht anders angegeben.

## Ausgangslage

- **Altbau mit geringer Dämmung.**
- Alte Ölheizung, kein Öl mehr im Tank. PV-Anlage mit Hausbatterie vorhanden.
- Heizwasser vor Beginn kalt: Rücklauf 18,2 °C (16:10), etwa Raumtemperatur.
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
- **Bilanz Testheizen 18:07–22:00:** 154 min Heizzeit, **6,95 kWh** ins Heizwasser
  (aus dem Leistungsverlauf L1 integriert, Grundlast 450 W abgezogen), Rücklauf von
  19,6 °C auf 28,9 °C (+9,3 K)
- **Kosten Testheizen:** ca. 0,56 € (Einspeisevergütung 0,08 €/kWh) bzw. ca. 2,43 €
  (Netzstrom 0,35 €/kWh). Tatsächlich kam der Strom bis etwa 21:45 aus dem Akku,
  danach aus dem Netz.
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

### Abkühlung und Wärmeverlust (vorläufige Abschätzung)

- **Abkühlung in der Pause 22:00–22:19** (Heizstab aus, Pumpe läuft): Rücklauf
  28,9 °C → 27,9 °C, also ca. **3,5 K pro Stunde** bei ca. 9 K über Ausgangstemperatur.
- Die Abkühlung verläuft exponentiell (Zeitkonstante grob 3 h). Über Nacht (8 h)
  wäre das Wasser praktisch wieder bei der Ausgangstemperatur von 19–20 °C.
- **Wärmeabgabe des Heizkreises**, geschätzt aus Aufheizen (+5,5 K/h bei 2,8 kW) und
  Abkühlen (−3,5 K/h): ca. **1 kW** bei knapp 30 °C Heizwasser.
- **KORREKTUR:** Diese 1 kW sind **nicht** der Wärmebedarf des Hauses, sondern nur,
  was das lauwarme Heizwasser abgibt. Das Wohnzimmer hatte 21,4 °C. Diese Wärme
  stammt nicht aus 30 °C warmen Heizkörpern, sondern aus gespeicherter Wärme der
  Mauern, Sonne und anderen Quellen.

### Reicht der Heizstab für den Winter? (Einordnung mit Richtwerten)

- Haus: Altbau, gering gedämmt, ca. **220 m²** Wohnfläche.
- Temperaturen am 04.10. abends: außen 12,8 °C (Fühler unter der Terrasse, Schatten;
  OpenWeatherMap 10,4 °C), innen Wohnzimmer 21,4 °C, Obergeschoss 19,3 °C.
- Übliche Richtwerte für ungedämmte Altbauten (keine Messung!):
  - Heizlast bei −12 °C ca. 100–150 W/m² → **22–33 kW** (Heizstab: 3 kW)
  - Jahreswärmebedarf ca. 150–250 kWh/m² → **33.000–55.000 kWh/Jahr**
    (Heizstab im Dauerbetrieb über ca. 200 Heiztage: max. ca. 14.000 kWh)
- **Fazit:** Der 3-kW-Heizstab ersetzt die Ölheizung im Winter nicht. Sinnvoll ist er
  für Grundwärme in der Übergangszeit, zum Verheizen von PV-Überschuss und zum
  Frostschutz.
- **Belastbarer machen:** bisheriger Ölverbrauch pro Jahr (Liter) → echter
  Wärmebedarf (1 l ≈ 10 kWh × 85 % Kesselwirkungsgrad).

### Messaufbau ab 04.10., 22:36

Home-Assistant-Hilfssensoren für die Langzeitmessung:

| Entity | Bedeutung |
|---|---|
| `sensor.heizstab_leistung_gemessen` | Leistung L1 minus 450 W Grundlast, nur wenn L1 > 2 kW (sonst 0) |
| `sensor.heizstab_energie` | Riemann-Integral (links) daraus, kWh gesamt |
| `sensor.heizstab_energie_taglich` | Tageszähler, Reset um 0 Uhr |
| `sensor.backdoor` | Außentemperatur (Fühler unter der Terrasse, Schatten) |
| `sensor.wohnzimmer_essbereich`, `sensor.oben_indoor` | Innentemperatur EG/OG |

Einschränkung: Laufen andere Verbraucher über 2 kW auf L1 (Wasserkocher, Backofen),
werden sie mitgezählt; die Grundlast schwankt zwischen ca. 400 und 600 W.

Firmware v1.4.0 (per OTA aufgespielt, erstmals erfolgreich): Die Stufe wird nach einem
Neustart wiederhergestellt. Heizstab seit 22:36 wieder auf 100 %.

## 2026-10-05 – Erste Nacht im Dauerbetrieb

- Heizstab durchgehend auf 100 % von 22:36 bis 13:44.
- **Energie 0–13:42 Uhr: 35,7 kWh** (Tageszähler), im Mittel 2,6 kW, also praktisch Dauerlauf.
- **Rücklauf (Stundenmaxima):** 23 Uhr 30,3 °C → 0 Uhr 34,9 → 2 Uhr 42,1 → **4–5 Uhr 43,0 °C (Höchstwert)**
  → 5–6 Uhr Abfall auf 31,4 °C → ab 8 Uhr 31–33 °C.
- Vermutung: Nachtabsenkung der Heizkörper-Thermostatköpfe. Nachts sind die Ventile zu,
  wenig Durchfluss, der Rücklauf steigt. Gegen 5 Uhr öffnen sie, kaltes Wasser kommt zurück.
  Der plötzliche Abfall am 04.10. um 23:28 passt ins selbe Muster. Noch zu bestätigen.
- 13:42: Wohnzimmer 22,8 °C, OG 20,1 °C, außen 18,8 °C, PV 3,3 kW (Heizstab läuft teils mit Sonnenstrom).
- Nebenbefund: Die DDNS-Adresse war seit der nächtlichen Zwangstrennung nicht aktualisiert
  (alte IP), daher kein Fernzugriff bis ca. 13:40. Home Assistant hat trotzdem lückenlos aufgezeichnet.
- **13:44: Test mit 25 % (ca. 0,7 kW im Mittel)** gestartet. Das Takten ist im L1-Verlauf sichtbar.

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
- [ ] Fragen an den Heizungsbauer: Durchfluss/Pumpenstufe, Ölkessel
      absperren, Thermostat-Einstellung, Entlüften/Anlagendruck
- [ ] Bisherigen Ölverbrauch pro Jahr erfragen (echter Wärmebedarf)
- [ ] Tatsächliche Heizlast im Normalbetrieb messen (kWh pro Tag) und mit der
      Außentemperatur (sensor.backdoor) und Innentemperatur dazu notieren
- [ ] Automation „Heizstab nur bei PV-Überschuss“
- [ ] Eigenen Strompreis und Einspeisevergütung eintragen
