# CLAUDE.md

ESPHome-Config (`heizstab.yaml`) für einen Heizstab-Controller auf einem ESP32-2432S028R
(„Cheap Yellow Display“, ILI9341 + XPT2046). Ein Fotek SSR-40DA an GPIO27 regelt einen
Heizlando MDC 230 (3 kW) per Burst-Fire (`slow_pwm`, 60 s Periode, Stufen 0/25/50/75/100 %).
Zusätzlich laufen ein Bluetooth-Proxy und ein BLE-Gerätezähler. Das Repo ist **öffentlich**.

## Hardware-Learnings (CYD)

- **Display und Touch brauchen zwei getrennte SPI-Busse:** Display auf CLK 14 / MOSI 13 /
  MISO 12 (CS 15, DC 2), Touch auf CLK 25 / MOSI 32 / MISO 39 (CS 33, IRQ 36). Backlight hängt an GPIO21.
- **GPIO4 ist die RGB-LED des CYD, kein Reset-Pin.** Ihn nicht als `reset_pin` für das
  Display eintragen.
- **GPIO12 ist ein Strapping-Pin** und braucht deshalb `ignore_strapping_warning: true`.
- **`color_palette: 8BIT` ist nötig,** sonst reicht der RAM nicht für den Framebuffer.
- **Große OTA-Updates scheitern bei laufendem BLE+WiFi am Heap.** Dann per USB flashen.
  Gegenmaßnahmen in der Config: `framework: esp-idf` und ein BLE-Scan mit window < interval
  (300/320 ms). Bei 100 % Scan-Duty-Cycle hat das WLAN keine Sendezeit mehr.
- **GPIO2 (DC) und GPIO15 (CS) sind ebenfalls Strapping-Pins**, deshalb steht auch dort
  `ignore_strapping_warning: true`.
- **Die Touch-Achsen sind durch `rotation: 90` vertauscht** (x = Zeile 0–320, y = Spalte
  0–240, also gestaucht gegenüber den Display-Koordinaten). Die y-Bereiche der Buttons sind
  empirisch ermittelt und dürfen sich nicht überlappen, sonst lösen zwei Buttons gleichzeitig
  aus. Eine Lösung per `transform:` wurde auf der Hardware noch nicht getestet.

## Config-Konventionen

- Die Leistung wird nur über das Script `set_heizstab_level` gesetzt. Es aktualisiert das
  Global, die PWM und die HA-Number gemeinsam. Touch-Buttons und `set_action` rufen nur
  dieses Script auf.
- `api_encryption_key` dient auch für OTA (`ota: encryption: {}`). Ein separates
  OTA-Passwort ist nicht vorgesehen.

## Arbeitsweise

- Änderungen an `heizstab.yaml` immer als **komplette Datei** liefern, keine Teil-Snippets.
- Vor jedem Commit `esphome config heizstab.yaml` zur Validierung ausführen, falls ESPHome
  verfügbar ist.
- `secrets.yaml` ist gitignored. Keine Passwörter, Keys, IP-Adressen oder persönlichen Daten
  committen. Neue Secrets als Platzhalter in `secrets.yaml.example` ergänzen.
- Sicherheit: Thermostat und STB des MDC 230 sind die Schutzebene, das ESP regelt nur die
  Leistung. Keine Änderungen vorschlagen, die diese Annahme aufweichen.
