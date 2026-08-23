# Bewegungs-/Präsenzgesteuerte Beleuchtung für Home Assistant

Home-Assistant-Blueprint zur Steuerung einer oder mehrerer Leuchten anhand von Bewegungs- und Präsenzsensoren.

Die Helligkeit wird abhängig von der aktuellen Uhrzeit automatisch aus vier frei konfigurierbaren Zeitprofilen ausgewählt.

## Import

[![In Home Assistant importieren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=[DEINE_GITHUB_RAW_URL_HIER](https://raw.githubusercontent.com/sanftleben/lichtsteuerung/refs/heads/main/lichtsteuerung.yml))

Alternativ kann der Blueprint über:

**Einstellungen → Automationen & Szenen → Blueprints → Blueprint importieren**

importiert werden.

## Funktionen

* Mehrere Bewegungs- und Präsenzsensoren
* Eine oder mehrere Leuchten
* Vier frei konfigurierbare Zeitprofile
* Individuelle Helligkeit pro Zeitprofil
* Frei konfigurierbare Ausschaltverzögerung
* Zusätzliche Bedingungen möglich
* Das Licht wird erst ausgeschaltet, wenn **alle** Sensoren inaktiv sind
* Neue Bewegung/Präsenz während der Ausschaltverzögerung hält das Licht aktiv
* Helligkeit wird nur beim Einschalten gesetzt

## Beispiel

Eine typische Konfiguration könnte so aussehen:

| Zeit          | Helligkeit |
| ------------- | ---------: |
| 00:00 – 06:00 |       10 % |
| 06:00 – 08:00 |       50 % |
| 08:00 – 18:00 |      100 % |
| 18:00 – 00:00 |       50 % |

Damit wird beispielsweise nachts nur mit 10 % Helligkeit eingeschaltet, während tagsüber die volle Helligkeit verwendet wird.

## Verhalten

Angenommen, zwei Sensoren sind konfiguriert:

* Bewegungsmelder
* Präsenzsensor

Sobald einer der Sensoren `on` meldet:

1. Die Automation wird ausgelöst.
2. Die aktuelle Uhrzeit wird bestimmt.
3. Die entsprechende Helligkeit wird ausgewählt.
4. Die konfigurierten Leuchten werden eingeschaltet.
5. Die Automation wartet, bis **alle** Sensoren `off` sind.
6. Die Ausschaltverzögerung beginnt.
7. Wird während dieser Zeit erneut Bewegung oder Präsenz erkannt, wird die Automation neu gestartet.
8. Erst wenn weiterhin alle Sensoren `off` sind, werden die Leuchten ausgeschaltet.

### Beispiel

```text
Bewegung erkannt
       │
       ▼
   Licht AN
       │
       ▼
Alle Sensoren OFF?
   │           │
  Nein        Ja
   │           │
   │           ▼
   │     Ausschaltverzögerung
   │           │
   │           ▼
   │      Neuer Trigger?
   │       │         │
   │      Ja        Nein
   │       │         │
   └───────┘         ▼
                  Licht AUS
```

## Installation

### Variante 1 – Import Button

Oben auf **In Home Assistant importieren** klicken.

Home Assistant öffnet anschließend direkt den Blueprint-Import.

### Variante 2 – Manuell

Die Blueprint-Datei herunterladen und nach:

```text
/config/blueprints/automation/
```

kopieren.

Zum Beispiel:

```text
/config/blueprints/automation/my_blueprints/motion_presence_light.yaml
```

Danach in Home Assistant zu:

**Einstellungen → Automationen & Szenen → Blueprints**

gehen.

Falls der Blueprint nicht sofort erscheint, die Automationen-/Blueprint-Seite neu laden.

## Konfiguration

Beim Erstellen einer Automation aus dem Blueprint stehen folgende Optionen zur Verfügung.

### Bewegungs-/Präsenzsensoren

Hier können beliebig viele `binary_sensor`-Entities ausgewählt werden.

Unterstützt werden insbesondere:

* Bewegungssensoren (`motion`)
* Präsenzsensoren (`occupancy`)
* Präsenzsensoren (`presence`)

Alle ausgewählten Sensoren werden gemeinsam berücksichtigt.

**Wichtig:** Das Licht wird erst ausgeschaltet, wenn alle ausgewählten Sensoren `off` sind.

### Leuchten

Eine oder mehrere `light`-Entities können ausgewählt werden.

Beispielsweise:

```text
light.wohnzimmer
light.stehlampe
light.deckenlampe
```

### Ausschaltverzögerung

Legt fest, wie lange nach dem Ausschalten aller Sensoren gewartet wird.

Beispiel:

```text
2 Minuten
```

Erkennt ein Sensor während dieser Zeit erneut Bewegung oder Präsenz, bleibt das Licht an und die Verzögerung beginnt anschließend erneut.

### Zusätzliche Bedingungen

Optional können zusätzliche Home-Assistant-Bedingungen definiert werden.

Beispiele:

* Nur wenn jemand zuhause ist
* Nur wenn eine bestimmte Tür geschlossen ist
* Nur wenn ein anderer Schalter aktiviert ist
* Nur zu bestimmten Zeiten

## Helligkeitsprofile

Die Automation besitzt vier Zeitprofile.

Jedes Profil besteht aus:

* Startzeit
* Helligkeit in Prozent

Beispiel:

```text
Profil 1
00:00 → 10 %

Profil 2
06:00 → 50 %

Profil 3
08:00 → 100 %

Profil 4
18:00 → 50 %
```

Die Automation verwendet immer das zuletzt gestartete Profil.

Dadurch ergibt sich:

```text
00:00 – 05:59 → Profil 1
06:00 – 07:59 → Profil 2
08:00 – 17:59 → Profil 3
18:00 – 23:59 → Profil 4
```

### Reihenfolge der Profile

Die Startzeiten sollten chronologisch sortiert sein:

```text
Profil 1 < Profil 2 < Profil 3 < Profil 4
```

Idealerweise beginnt Profil 1 bei:

```text
00:00
```

Damit ist der komplette Tag abgedeckt.

## Beispielkonfiguration

Für eine typische Wohnzimmerbeleuchtung:

```text
Sensoren:
  - binary_sensor.wohnzimmer_bewegung
  - binary_sensor.wohnzimmer_prasenz

Leuchten:
  - light.wohnzimmer_decke
  - light.wohnzimmer_stehlampe

Ausschaltverzögerung:
  3 Minuten

Profil 1:
  00:00
  10 %

Profil 2:
  06:00
  40 %

Profil 3:
  08:00
  100 %

Profil 4:
  18:00
  50 %
```

Ergebnis:

* Nachts → dezentes Licht
* Morgens → reduzierte Helligkeit
* Tagsüber → volle Helligkeit
* Abends → reduzierte Helligkeit

## Hinweise

Die Helligkeit wird beim Einschalten anhand der **aktuellen Uhrzeit** bestimmt.

Die Automation verändert die Helligkeit einer bereits eingeschalteten Leuchte nicht nachträglich.

Das ist beabsichtigt: Wenn sich die Helligkeit während einer laufenden Präsenzphase verändert, wird das Licht nicht plötzlich dunkler oder heller.

Bei einem erneuten Trigger wird die Automation jedoch erneut gestartet und die Helligkeit entsprechend der aktuellen Uhrzeit gesetzt.

## Voraussetzungen

* Home Assistant mit Blueprint-Unterstützung
* Bewegungs- oder Präsenzsensoren als `binary_sensor`
* Zu steuernde Leuchten als `light`

## Lizenz

Frei verwendbar und anpassbar für die eigene Home-Assistant-Installation.
