# CLAUDE.md

ESPHome-Config (`heizstab.yaml`) für einen Heizstab-Controller auf einem ESP32-2432S028R
(„Cheap Yellow Display“, ILI9341 + XPT2046). Ein Fotek SSR-40DA an GPIO27 regelt einen
Heizlando MDC 230 (3 kW) per Burst-Fire (`slow_pwm`, 60 s Periode, Stufen 0/25/50/75/100 %).
Seit v1.3.0 ohne Bluetooth (siehe unten). Das Repo ist **öffentlich**.

## Hardware-Learnings (CYD)

- **Display und Touch brauchen zwei getrennte SPI-Busse:** Display auf CLK 14 / MOSI 13 /
  MISO 12 (CS 15, DC 2), Touch auf CLK 25 / MOSI 32 / MISO 39 (CS 33, IRQ 36). Backlight hängt an GPIO21.
- **GPIO4 ist die RGB-LED des CYD, kein Reset-Pin.** Ihn nicht als `reset_pin` für das
  Display eintragen.
- **GPIO12 ist ein Strapping-Pin** und braucht deshalb `ignore_strapping_warning: true`.
- **`color_palette: 8BIT` ist nötig,** sonst reicht der RAM nicht für den Framebuffer.
- **Kein Bluetooth auf diesem Board einbauen.** Ohne PSRAM passen BLE-Stack (Proxy),
  8-Bit-Framebuffer, WLAN und Noise-API nicht gemeinsam in den RAM. Mit Bluetooth lief der
  Heap auf ~124 Bytes leer: WLAN-Abbrüche, OTA-Abbrüche, Noise-Handshake scheitert
  (`HANDSHAKESTATE_READ_FAILED`, HA meldet `invalid_psk`). Ohne Bluetooth: DRAM 26 %,
  Image 0,95 MB statt 1,38 MB.
- **Erstes Flashen nach Arduino → esp-idf und bei kaputtem WLAN nur per USB** über
  web.esphome.io (CH340-Treiber nötig, Datenkabel). Bei mehreren Mesh-APs mit derselben
  SSID nimmt das ESP beim Start den stärksten; ein Neustart kann das Signal verbessern.
- **GPIO2 (DC) und GPIO15 (CS) sind ebenfalls Strapping-Pins**, deshalb steht auch dort
  `ignore_strapping_warning: true`.
- **Die Touch-Achsen sind durch `rotation: 90` vertauscht** (x = Zeile 0–320, y = Spalte
  0–240, also gestaucht gegenüber den Display-Koordinaten). Die y-Bereiche der Buttons sind
  empirisch ermittelt und dürfen sich nicht überlappen, sonst lösen zwei Buttons gleichzeitig
  aus. Eine Lösung per `transform:` wurde auf der Hardware noch nicht getestet.

## Config-Konventionen

- **Das Repo ist unabhängig von einer bestimmten Installation.** Keine privaten Gerätenamen,
  Entity-IDs, IPs, MAC-Adressen oder Messdaten in Repo-Dateien. Installationsspezifisches
  steht als Platzhalter unter `substitutions`. Der Nutzer überschreibt die Werte in seiner
  lokalen Dashboard-Datei, die `heizstab.yaml` als GitHub-Package einbindet (siehe README).

- Die Leistung wird nur über das Script `set_heizstab_level` gesetzt. Es aktualisiert das
  Global, die PWM und die HA-Number gemeinsam. Touch-Buttons und `set_action` rufen nur
  dieses Script auf.
- Die Config wird im **ESPHome-Dashboard von Home Assistant** genutzt und braucht nur die
  Secrets `wifi_ssid` und `wifi_password`. Es sollen keine weiteren Secrets dazukommen.
- Die API ist verschlüsselt (`encryption: {}`), den Key hinterlegt HA zur Laufzeit. OTA ist
  deshalb ungeschützt, denn ein Laufzeit-Key kann nicht für OTA genutzt werden. Der offene
  Fallback-Hotspot (`ap: {}`) ist eine bewusste Entscheidung des Nutzers. Die Abwägung steht
  in der README unter „Sicherheit im Netzwerk“.
- Seit v1.4.0 wird die Stufe gespeichert (`restore_value: yes`) und per `on_boot` wieder
  gesetzt. Das hat der Nutzer ausdrücklich so gewünscht: Der Heizstab soll nach Neustart
  oder Stromausfall von selbst weiterheizen.
- Versionierung nach SemVer in `esphome: project: version`. Bei jeder Änderung der Config
  die Version erhöhen.

## Arbeitsweise

- Änderungen nie direkt auf `main` pushen: Branch anlegen, Pull Request öffnen, Merge erst
  nach Freigabe durch den Nutzer.
- Änderungen an `heizstab.yaml` immer als **komplette Datei** liefern, keine Teil-Snippets.
- Vor jedem Commit `esphome config heizstab.yaml` zur Validierung ausführen, falls ESPHome
  verfügbar ist.
- `secrets.yaml` ist gitignored. Keine Passwörter, Keys, IP-Adressen oder persönlichen Daten
  committen. Neue Secrets als Platzhalter in `secrets.yaml.example` ergänzen.
- Sicherheit: Thermostat und STB des MDC 230 sind die Schutzebene, das ESP regelt nur die
  Leistung. Keine Änderungen vorschlagen, die diese Annahme aufweichen.
