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
- **25 % reichen nicht:** Der Rücklauf fiel bis 19:25 von ca. 33,7 °C auf 25,7 °C (Stundenmittel,
  gut 1 K pro Stunde). Danach 19:45 auf 75 %, ab 20:07 wieder 100 %.
- **Umwälzpumpe** seit ca. 20:00 an einem Shelly Plug M Gen3: Leistung konstant ca. 35 W.
- **Hydraulik geklärt:** Hinter dem Ölkessel sitzt ein 4-Wege-Mischer, der Kessel wird nicht
  durchströmt. Der Heizstab sitzt im inneren Heizkreis, im Rücklauf von den Heizkörpern zum
  Mischer. Der Rücklauffühler sitzt vor dem Heizstab, der Vorlauffühler dahinter.
- **Heizkörperventile:** Zwei alte Homematic-Thermostate (Küche, Hannah) meldeten 0 %
  Ventilöffnung, die Heizkörper waren aber warm, also offen. Küche: Batterien getauscht und neu
  angelernt, danach kalt (06.10. früh geprüft). Hannah: auf „Aus“ gestellt, Raum von 21,6 °C
  auf 19,3 °C (06.10. 07:50), Batterien werden getauscht. Bis dahin waren die Messungen
  „mit unkontrollierten Heizkörpern“.
- **Temperatur-ESP (Vor-/Rücklauf):** Beim Reparieren des Vorlauffühlers beschädigt, seit
  20:04 keine Wasser-Temperaturen. Ersatz: Wemos D1 mini mit DS18B20-Adapter-Modulen.
- **Tagesbilanz 05.10.:** **50,6 kWh** Heizstab, außen Mittel 14,0 °C (min 8,9 / max 20,9 °C).

## 2026-10-06 – Zweite Nacht

- Heizstab die ganze Nacht auf 100 %. **Energie 0–07:50 Uhr: 10,8 kWh**, also Dauerlauf.
- Außen: Tiefstwert der bisherigen Messung, ca. 9 °C (07:50: 8,8 °C).
- 07:50: Wohnzimmer 21,6 °C, OG 19,2 °C, Hannah 19,3 °C (Ventil zu), Küche 26,9 °C (Heizkörper kalt).
- **07:50: Vorlauf 40 °C**, von Hand am Thermometer an der Heizung abgelesen (Temperatur-ESP defekt).
- Akku leer (5,9 %), der Heizstab läuft morgens mit ca. 3,2 kW Netzbezug.
- Homematic IP: Der WLAN-Access-Point (HmIP-WLAN-HAP) hat seit dem 05.10. früh keine stabile
  Verbindung mehr, die Heizgruppen liefern keine Werte.
- **Tagsüber:** wolkenlos, außen bis ca. 22 °C. Ab ca. 10:30 läuft der Heizstab komplett mit
  PV (12:04: PV 5,35 kW, Heizstab 2,9 kW, Akku lädt mit 1,9 kW, Netz 0).
- **E-Auto:** lädt ab 13:25 durchgehend mit ca. 3,55 kW (einphasig, L2), bis 19:10 gut 20 kWh.
  Dadurch floss der PV-Überschuss ins Auto statt in den Akku, und der Akku war schon um 19:04
  leer (Prognose ohne Auto: voll gegen 17 Uhr, reicht bis ca. 21 Uhr).
- **Temperatur gehalten:** Ab ca. 16 Uhr schaltet der Thermostat des Heizstabs ihn immer wieder
  für 10–20 min ab, die eingestellte Temperatur ist erreicht. **~19:15: Vorlauf 50 °C**
  (Handablesung). Um 50 °C Vorlauf zu halten, brauchte der Heizstab von 16 bis 19 Uhr
  **im Mittel ca. 1,2 kW** (0,87 / 1,75 / 1,09 kWh pro Stunde), bei außen ca. 19–22 °C,
  mit geschlossenen Ventilen in Küche und Hannah.
- 19:12: Wohnzimmer 22,5 °C, OG 21,1 °C (früh 19,2 °C), Küche 22,2 °C, Hannah 20,6 °C.
  Heizstab-Energie bis 19:11: 35,0 kWh.
- **Abend 06.10.:** Drehregler am Heizstab auf „3 von 5“, alle Heizkörper zu außer dem Bad
  (entlüftet). Vorlauf ca. 20 Uhr: **40 °C** (Handablesung). Mit allen Heizkörpern zu stand das
  Wasser fast still (Heizstab nur 40–50 s an), erst mit offenem Bad längere An-Phasen.
  Heizstab-Energie: 21–22 Uhr 0,64 kWh, **22–23 Uhr 0,19 kWh**, 23–24 Uhr 1,66 kWh
  (Heizkörper Klavierzimmer kurz geöffnet, wurde nicht richtig warm).
- 4-Wege-Mischer: keine Steuerung, steht am Anschlag, Ölkessel nicht durchströmt.
- 00:26 (07.10.): Umwälzpumpe Wilo Star-RS 30/4 von Stufe Mitte (gemessen 35–38 W) auf
  max (52,6 W).
- Korrektur zur Nacht 05./06.10.: Der Heizstab lief **nicht** durchgehend. 20–24 Uhr
  Dauerlauf (je ca. 2,65 kWh), 0–5 Uhr Takten (0,2–1,7 kWh/h), 5–8 Uhr wieder Volllast.
  20–8 Uhr gesamt 21,9 kWh.

## Tagesbilanz 04.–09.10.

Energie aus Home Assistant (Tageszähler Heizstab, Wechselrichter). Netzanteil des Heizstabs
geschätzt: Netzbezug des Tages minus ca. 3 kWh Grundbedarf ohne Heizstab (am 06.10. das
E-Auto separat herausgerechnet). Preise angenommen: Netzstrom 0,35 €/kWh, PV-Strom mit
entgangener Einspeisevergütung 0,08 €/kWh.

| Tag | Heizstab | davon Netz | davon PV/Akku | Kosten | Außen Ø (min–max) | WZ Ø | OG Ø |
|---|---|---|---|---|---|---|---|
| 04.10. ab 18 Uhr | 10,6 kWh | 3,7 | 7,0 | 1,84 € | 15,3 (11–22) | 21,2 | 19,2 |
| 05.10. | 50,6 kWh | 35,1 | 15,5 | 13,51 € | 14,0 (8,9–20,9) | 21,9 | 19,7 |
| 06.10. | 39,2 kWh | 14,5 | 24,7 | 7,05 € | 14,7 (8,6–22,6) | 22,3 | 20,1 |
| 07.10. | 26,3 kWh | 13,6 | 12,7 | 5,78 € | 14,6 (9,0–21,8) | 22,3 | 20,6 |
| 08.10. | 27,7 kWh | 18,5 | 9,1 | 7,22 € | 14,9 (11,6–20,6) | 21,8 | 20,9 |
| 09.10. bis 11:23 | 18,6 kWh | 18,6 | 0 | 6,51 € | 10,3 (8,4–12,0) | 21,4 | 21,1 |
| **Summe** | **172,9 kWh** | **104,0** | **68,9** | **41,90 €** | | | |

- Öl-Äquivalent für dieselbe Wärme: ca. 20 l Heizöl (85 % Wirkungsgrad), ca. **33,40 €**.
- PV-Erzeugung: 42,7 / 39,6 / 42,1 / 39,5 / 20,1 kWh (04.–08.10.). Eingespeist wurden trotzdem
  8,1 kWh (05.10.) und **15,2 kWh (07.10.)**: Sonnenstrom ging ins Netz, während der Heizstab
  nachts mit Netzstrom lief. Genau hier setzt die geplante PV-Automation an.
- Umwälzpumpe: 0,9–1,3 kWh pro Tag (ca. 0,30–0,47 €).
- Seit 07.10. pendelt der Heizstab bei 26–28 kWh/Tag, die Räume bleiben dabei warm
  (OG im Tagesmittel von 19,2 auf 21,1 °C gestiegen).
- 09.10. ca. 11:15: Drehregler am Heizstab auf **max** (ca. 75 °C), um die Höchsttemperatur zu
  testen.

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
