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
- **Bluetooth-Proxy** für Home Assistant
- Zähler der in Reichweite sichtbaren **BLE-Geräte** auf dem Display

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
3. **Neues Gerät** anlegen, oder ein bestehendes öffnen, und den Inhalt von
   [heizstab.yaml](heizstab.yaml) komplett in den Editor kopieren. Unter `esphome:` ggf.
   `name` und `friendly_name` anpassen.
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
- Anzahl der BLE-Geräte, die in den letzten 30 s gesehen wurden, und den Namen des zuletzt
  gesehenen Geräts
- die aktuelle Heizstab-Stufe (`AUS` / `25%` … `100%`)

### In Home Assistant
Die Leistungsstufe ist als Number-Entity **„Heizstab Leistung“** (0–100 %, Schrittweite 25)
verfügbar. Damit lässt sich der Heizstab per Dashboard, Automation oder Skript steuern,
z. B. um PV-Überschuss zu nutzen. Eine Änderung am Display wird sofort an Home Assistant
gemeldet und umgekehrt.

Nach einem Neustart des ESP steht die Leistung immer auf 0 %.

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
- **OTA bei BLE + WiFi:** Bei aktivem Bluetooth-Proxy wird der Heap knapp. Größere
  OTA-Updates brechen deshalb ab. In dem Fall per USB flashen. Die Config nutzt deshalb das
  sparsamere Framework `esp-idf` und ein BLE-Scanfenster von 300 ms pro 320 ms, damit
  WLAN und Bluetooth sich das Funkmodul teilen können.
- **Wechsel des Frameworks:** Wer von einer älteren `arduino`-Version dieser Config kommt,
  muss einmal per USB flashen, weil sich die Partitionstabelle ändern kann.
- **Touch-Achsen:** Durch `rotation: 90` sind die Touch-Achsen vertauscht
  (x entspricht der Zeile, y der Spalte).

## Lizenz

[MIT](LICENSE). Nutzung auf eigene Gefahr.
