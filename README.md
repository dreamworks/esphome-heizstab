# ESPHome Heizstab-Controller

ESPHome-Konfiguration für einen Touch-Controller, der die Leistung eines 3-kW-Heizstabs
(Heizlando MDC 230) über ein Solid-State-Relais in fünf Stufen regelt. Bedient wird er über
das 2,8"-Touchdisplay eines ESP32-2432S028R („Cheap Yellow Display“) oder über Home Assistant.

<!-- Foto-Platzhalter: Bild unter docs/foto.jpg ablegen und die nächste Zeile einkommentieren -->
<!-- ![Heizstab-Controller](docs/foto.jpg) -->
> 📷 *Foto folgt*

---

## ⚠️ Sicherheitshinweise

> **Bitte vor dem Nachbau vollständig lesen.**

- **230 V / 3 kW:** Netzspannung und hohe Leistung sind lebensgefährlich. Der Anschluss
  darf **nur durch eine Elektrofachkraft** erfolgen.
- **Den Schutzleiter (PE) niemals über das SSR führen.** Er wird immer direkt mit dem
  Heizstab verbunden.
- **Die Umwälzpumpe muss während des Heizbetriebs laufen.** Der Heizstab braucht einen
  Mindestdurchfluss, sonst überhitzt er.
- **Thermostat und Sicherheitstemperaturbegrenzer (STB) des MDC 230 bleiben die eigentliche
  Schutzebene.** Das ESP32 regelt nur die Leistung. Es ist keine Sicherheitseinrichtung und
  ersetzt weder Thermostat noch STB.
- **Nutzung auf eigene Gefahr.** Es gibt keine Gewährleistung und keine Haftung (siehe
  [LICENSE](LICENSE)).

---

## Funktionen

- Leistungsregelung per **Burst-Fire** (`slow_pwm`, 60 s Periode): ganze Netzhalbwellen
  statt Phasenanschnitt, damit wenig Störungen und kaum Schaltverluste
- Stufen **0 / 25 / 50 / 75 / 100 %** (0 / 750 / 1500 / 2250 / 3000 W)
- Bedienung über **Touch-Buttons** am Display
- Integration in **Home Assistant** über die native API (Number-Entity für die Leistungsstufe)
- Anzeige, ob **Home Assistant verbunden** ist
- Bewusst **ohne Bluetooth** (siehe [Bekannte Eigenheiten](#bekannte-eigenheiten-des-cyd))

## Hardware

| Komponente | Beschreibung |
|---|---|
| ESP32-2432S028R („CYD“) | ESP32 mit 2,8"-Display ILI9341 (320×240) und resistivem Touch XPT2046 |
| Heizlando MDC 230 (Bundle) | Heizstab 3 kW, eingebauter Thermostat 30–75 °C, STB 85 °C |
| Fotek SSR-40DA | Solid-State-Relais (3–32 V DC Steuerung, 24–380 V AC Last), **mit Kühlkörper** |
| Netzteil 5 V | USB-Versorgung für das CYD |

> Das SSR erzeugt bei 3 kW spürbar Abwärme (ca. 1–1,5 W pro Ampere, bei 13 A also ca. 15–20 W).
> Ohne ausreichend dimensionierten Kühlkörper fällt es aus.

## Pinbelegung

| Funktion | GPIO | Hinweis |
|---|---|---|
| Display SPI CLK | 14 | eigener SPI-Bus |
| Display SPI MOSI | 13 | |
| Display SPI MISO | 12 | Strapping-Pin, siehe unten |
| Display CS | 15 | |
| Display DC | 2 | |
| Touch SPI CLK | 25 | zweiter, getrennter SPI-Bus |
| Touch SPI MOSI | 32 | |
| Touch SPI MISO | 39 | nur Eingang |
| Touch CS | 33 | |
| Touch IRQ | 36 | nur Eingang |
| Display-Hintergrundbeleuchtung | 21 | |
| **SSR-Ansteuerung** | **27** | an SSR-Eingang „+“ (3), GND an „−“ (4) |

## Installation

Die Config ist für das **ESPHome-Dashboard in Home Assistant** (ESPHome-Add-on) gedacht.

1. In Home Assistant das **ESPHome-Add-on** installieren und öffnen.
2. Im Dashboard oben rechts unter **Secrets** prüfen, dass `wifi_ssid` und `wifi_password`
   eingetragen sind. Weitere Secrets braucht die Config nicht (Vorlage:
   [secrets.yaml.example](secrets.yaml.example)).
3. **Neues Gerät** anlegen, oder ein bestehendes öffnen, und als Inhalt nur diese paar
   Zeilen eintragen. Die eigentliche Config wird direkt aus diesem Repo geladen, und
   eigene Werte stehen nur in der lokalen Datei:
   ```yaml
   substitutions:
     device_name: heizstab              # Gerätename im Netz
     friendly_name: ESP Heizstab        # Anzeigename in Home Assistant
     temp_entity: sensor.mein_ruecklauf # HA-Entity der Rücklauftemperatur
     # optional: max_temp: "50", resume_temp: "45"

   packages:
     heizstab: github://dreamworks/esphome-heizstab/heizstab.yaml@main
   ```
   Alle verfügbaren Platzhalter stehen oben in [heizstab.yaml](heizstab.yaml) unter
   `substitutions`. Mit `@main` gibt es bei jedem Kompilieren den neuesten Stand.
   Wer eine feste Version will, verweist auf einen Commit oder ein Tag.
4. **Install → Plug into this computer:** Den ersten Flash per USB durchführen. Spätere
   Updates gehen per OTA („Wirelessly“). Wenn ein OTA-Update fehlschlägt, wieder per USB
   flashen (siehe [Bekannte Eigenheiten](#bekannte-eigenheiten-des-cyd)).
5. Das Gerät wird in Home Assistant automatisch erkannt und kann unter
   *Einstellungen → Geräte & Dienste* hinzugefügt werden. Den Schlüssel für die
   verschlüsselte API hinterlegt Home Assistant dabei selbst.

<details>
<summary>Alternative: ESPHome auf der Kommandozeile</summary>

```bash
git clone https://github.com/dreamworks/esphome-heizstab.git
cd esphome-heizstab
cp secrets.yaml.example secrets.yaml   # WLAN-Daten eintragen, Datei ist gitignored
esphome run heizstab.yaml
```
</details>

## Sicherheit im Netzwerk

Die Config kommt bewusst **ohne zusätzliche Secrets** aus, damit sie sich direkt im
Dashboard einsetzen lässt. Das hat folgende Folgen:

- **API:** Sie ist verschlüsselt (`encryption: {}`). Den Schlüssel erzeugt Home Assistant
  beim Einbinden und hinterlegt ihn auf dem Gerät.
- **OTA-Updates sind nicht geschützt.** ESPHome kann den zur Laufzeit hinterlegten
  API-Schlüssel nicht für OTA verwenden. Jedes Gerät im selben Netz kann also neue Firmware
  aufspielen.
- **Der Fallback-Hotspot ist offen.** Findet das ESP sein WLAN nicht, öffnet es einen
  Hotspot ohne Passwort mit Captive Portal. Wer in Funkreichweite ist, kann sich dann
  verbinden und, weil OTA ungeschützt ist, auch Firmware aufspielen.

Wer das absichern möchte, hat zwei Möglichkeiten:

```yaml
# 1. Hotspot mit Passwort
wifi:
  ap:
    password: !secret ap_password

# 2. Fester Schlüssel für API und OTA (erzeugen mit: openssl rand -base64 32)
api:
  encryption:
    key: !secret api_encryption_key
ota:
  - platform: esphome
    encryption: {}   # verwendet den API-Schlüssel
```

Alternativ entfällt der Hotspot ganz, wenn man `ap:` und `captive_portal:` entfernt. Bei
einem neuen WLAN-Passwort muss das Gerät dann per USB neu geflasht werden.

Unabhängig davon gilt: Der Thermostat und der STB des MDC 230 begrenzen die Temperatur
hardwareseitig, auch bei manipulierter Firmware.

## Bedienung

### Am Display
In der unteren Buttonleiste (`0`, `1/4`, `1/2`, `3/4`, `1`) wählst du die Leistungsstufe;
die aktive Stufe ist rot hinterlegt. Außerdem zeigt das Display:

- WLAN-Signalstärke und IP-Adresse
- ob Home Assistant gerade verbunden ist (`verbunden` / `getrennt`)
- die aktuelle Heizstab-Stufe (`AUS` / `25%` … `100%`)

### In Home Assistant
Die Leistungsstufe ist als Number-Entity **„Heizstab Leistung“** (0–100 %, Schrittweite 25)
verfügbar. Damit lässt sich der Heizstab per Dashboard, Automation oder Skript steuern,
z. B. um PV-Überschuss zu nutzen. Eine Änderung am Display wird sofort an Home Assistant
gemeldet und umgekehrt.

### Automatik: Zieltemperatur halten (seit v1.5.0)

Im **Modus „Automatik“** wählt das ESP die Stufe selbst, nach dem Abstand zwischen dem
Rücklauf und der **Zieltemperatur** (25–50 °C):

| Rücklauf unter Ziel | Stufe |
|---|---|
| 4 K oder mehr | 100 % |
| 3 K | 75 % |
| 2 K | 50 % |
| 1 K | 25 % |
| Ziel erreicht | 0 % |

Die Regelung läuft auf dem ESP. Die Rücklauftemperatur kommt aus Home Assistant
(`temp_entity` unter `substitutions`). Tippt man einen Touch-Button oder setzt die
Leistung in Home Assistant, schaltet das ESP auf **„Manuell“** zurück.

### Temperaturstopp (seit v1.5.0)

Erreicht der Rücklauf **50 °C** (`max_temp`), schaltet das ESP das SSR ab, unabhängig
von Modus und Stufe. Unter **45 °C** (`resume_temp`) heizt es weiter. In Home Assistant
zeigt der Binary-Sensor **„Temperaturstopp“** den Zustand an, auf dem Display erscheint
„TEMP-STOPP“.

Wichtig: Stopp und Automatik hängen am Temperaturwert aus Home Assistant. Ohne Verbindung
nutzt das ESP den zuletzt empfangenen Wert. **Thermostat und STB des MDC 230 bleiben die
eigentliche Schutzebene**, den Thermostat am besten auf etwa 50–55 °C stellen.

**Nach einem Neustart oder Stromausfall** stellt das ESP die zuletzt gewählte Stufe
wieder her (seit v1.4.0). Der Heizstab heizt also von selbst weiter. Das setzt voraus, dass
die Umwälzpumpe dauerhaft läuft. Thermostat und STB des MDC 230 bleiben die Schutzebene.
Wer das nicht möchte, setzt in der Config bei `heizstab_level` `restore_value: no`. Dann
startet das ESP immer mit 0 %.

## Funktionsweise der Leistungsregelung

`slow_pwm` schaltet das SSR innerhalb einer Periode von 60 s für einen Anteil ein, der der
gewählten Stufe entspricht. Bei 50 % ist es also 30 s an und 30 s aus. Das SSR-40DA schaltet
im Nulldurchgang, deshalb entstehen kaum Netzrückwirkungen. Durch die thermische Trägheit
des Wassers ergibt sich im Mittel die gewünschte Leistung.

Der eingebaute Thermostat des MDC 230 begrenzt unabhängig davon die Wassertemperatur.

## Bekannte Eigenheiten des CYD

- **Zwei SPI-Busse:** Display und Touch hängen beim ESP32-2432S028R an getrennten SPI-Bussen
  und müssen in ESPHome auch als zwei Busse konfiguriert werden.
- **GPIO4 ist kein Reset-Pin.** An GPIO4 hängt die RGB-LED des Boards. Das Display wird ohne
  Reset-Pin konfiguriert.
- **GPIO12 ist ein Strapping-Pin.** Er wird trotzdem als Display-MISO genutzt, daher steht
  in der Config `ignore_strapping_warning: true`.
- **RAM:** Der Framebuffer passt nur mit `color_palette: 8BIT` in den Speicher.
- **Kein Bluetooth:** Das ESP32 des CYD hat keinen Zusatzspeicher (PSRAM). Bluetooth-Proxy,
  Display-Framebuffer, WLAN und verschlüsselte API zusammen ließen den freien Heap auf rund
  100 Bytes schrumpfen. Die Folge: WLAN-Abbrüche, abgebrochene OTA-Updates, und Home Assistant
  konnte keine verschlüsselte Verbindung aufbauen. Seit v1.3.0 ist Bluetooth deshalb entfernt.
  Für einen Bluetooth-Proxy besser ein eigenes ESP32 ohne Display verwenden.
- **WLAN-Mesh:** Bei mehreren Zugangspunkten mit derselben SSID wählt das ESP beim Start den
  stärksten. Ein Neustart kann deshalb ein deutlich besseres Signal bringen.
- **Wechsel des Frameworks:** Wer von einer älteren `arduino`-Version dieser Config kommt,
  muss einmal per USB flashen, weil sich die Partitionstabelle ändern kann.
- **Touch-Achsen:** Durch `rotation: 90` sind die Touch-Achsen vertauscht
  (x entspricht der Zeile, y der Spalte).

## Lizenz

[MIT](LICENSE). Nutzung auf eigene Gefahr.
