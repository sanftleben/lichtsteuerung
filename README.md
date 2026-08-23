# Home Assistant Blueprint – Bewegungs-/Präsenzgesteuerte Beleuchtung

Steuert eine oder mehrere `light`-Entities anhand von Bewegung, Präsenz und optional der Außenhelligkeit.

Die Helligkeit wird abhängig von der Tageszeit aus vier frei konfigurierbaren Zeitprofilen gewählt.

## Import

> **Hinweis:** Den Link im Button musst du auf die Raw-URL deiner Blueprint-Datei in deinem GitHub-Repository anpassen.

[![In Home Assistant importieren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/sanftleben/lichtsteuerung/refs/heads/main/lichtsteuerung.yml)

Alternativ kann der Blueprint über **Einstellungen → Automationen & Szenen → Blueprints → Blueprint importieren** importiert werden.

## Funktionen

- Mehrere Bewegungs- und Präsenzsensoren
- Eine oder mehrere Leuchten
- Vier frei konfigurierbare Tageszeit-/Helligkeitsprofile
- Individuelle Helligkeit von 1–100 %
- Einstellbare Ausschaltverzögerung
- Optionale zusätzliche Home-Assistant-Bedingungen
- Optionaler Außen-Helligkeitssensor in Lux
- Außenhelligkeit kann das Licht einschalten, wenn jemand im Raum ist
- Außenhelligkeit schaltet das Licht niemals aus
- Das Licht wird erst ausgeschaltet, wenn **alle** Bewegungs-/Präsenzsensoren inaktiv sind
- Neue Bewegung/Präsenz während der Ausschaltverzögerung startet die Automation neu

## Logik

### Bewegung / Präsenz

Ein `on`-Event eines der konfigurierten Bewegungs-/Präsenzsensoren schaltet das Licht sofort ein.

Die aktuelle Uhrzeit bestimmt dabei die Helligkeit.

Die Außenhelligkeit spielt bei diesem Trigger keine Rolle.

Das bedeutet beispielsweise:

```text
Tagsüber + Bewegung
→ Licht 100 %

Nachts + Bewegung
→ Licht 10 %
```

### Außenhelligkeit

Der Außenhelligkeitssensor ist ein zusätzlicher Trigger.

Beispiel:

```text
Außenhelligkeit < 100 lx
+
mindestens ein Bewegungs-/Präsenzsensor = on
→ Licht EIN
```

Dadurch funktioniert auch dieser Fall:

```text
14:00
Person bereits im Wohnzimmer
Präsenz = on
Außenhelligkeit = 500 lx
→ Licht bleibt aus

15:30
Wolken / Dämmerung
Außenhelligkeit fällt unter 100 lx
Präsenz = on
→ Licht geht an
```

Umgekehrt wird das Licht **nicht ausgeschaltet**, wenn es draußen wieder heller wird.

Das Ausschalten erfolgt ausschließlich über die Bewegungs-/Präsenzsensoren und die eingestellte Ausschaltverzögerung.

## Hysterese

Für die Außenhelligkeit können zwei Schwellen eingestellt werden:

```text
Einschalten:        100 lx
Rückkehr zu hell:   150 lx
```

Die Rückkehr-Schwelle sollte höher als die Einschalt-Schwelle gewählt werden.

Damit wird verhindert, dass die Automation bei kleinen Schwankungen der Außenhelligkeit ständig zwischen hell und dunkel wechselt.

### Beispiel

```text
120 lx
→ noch hell

99 lx
→ dunkel genug → Licht kann eingeschaltet werden

105 lx
→ bleibt im dunklen Bereich

140 lx
→ bleibt im dunklen Bereich

151 lx
→ wieder hell
```

Die Außenhelligkeit schaltet das Licht dabei nicht aus.

## Bewegungs-/Präsenzsensoren

Es können mehrere Sensoren ausgewählt werden.

Beispiel:

```text
binary_sensor.wohnzimmer_bewegung
binary_sensor.wohnzimmer_prasenz
```

Alle Sensoren werden gemeinsam betrachtet.

Das Licht wird erst für die Ausschaltverzögerung freigegeben, wenn **alle** Sensoren `off` sind.

Beispiel:

```text
Bewegung       ON
Präsenz        ON
               ↓
Bewegung       OFF
Präsenz        ON
               ↓
Keine Abschaltung
               ↓
Präsenz        OFF
               ↓
Ausschaltverzögerung
               ↓
Licht AUS
```

## Ausschaltverzögerung

Die Ausschaltverzögerung beginnt erst, wenn alle Bewegungs-/Präsenzsensoren `off` sind.

Beispiel:

```text
Ausschaltverzögerung = 2 Minuten
```

Wenn innerhalb dieser Zeit wieder Bewegung oder Präsenz erkannt wird, startet die Automation neu und das Licht bleibt an.

## Tageszeitprofile

Es stehen vier frei konfigurierbare Profile zur Verfügung.

Beispiel:

| Profil | Start | Helligkeit |
|---|---:|---:|
| 1 | 00:00 | 10 % |
| 2 | 06:00 | 40 % |
| 3 | 08:00 | 100 % |
| 4 | 18:00 | 50 % |

Damit ergibt sich:

```text
00:00 – 05:59 → 10 %
06:00 – 07:59 → 40 %
08:00 – 17:59 → 100 %
18:00 – 23:59 → 50 %
```

Profil 1 sollte idealerweise um `00:00` beginnen.

Die Startzeiten sollten chronologisch sortiert werden.

## Beispielkonfiguration

Für ein Wohnzimmer:

```text
Bewegung:
  binary_sensor.wohnzimmer_bewegung

Präsenz:
  binary_sensor.wohnzimmer_prasenz

Außenhelligkeit:
  sensor.aussen_helligkeit

Leuchten:
  light.wohnzimmer_decke
  light.wohnzimmer_stehlampe

Ausschaltverzögerung:
  3 Minuten

Außenhelligkeit:
  Einschalten unter 100 lx
  Rückkehr zu hell ab 150 lx

Helligkeit:
  00:00 → 10 %
  06:00 → 40 %
  08:00 → 100 %
  18:00 → 50 %
```

## Verhalten im Alltag

### Szenario 1 – Bewegung am Tag

```text
Bewegung erkannt
→ Licht 100 %
```

### Szenario 2 – Bewegung in der Nacht

```text
Bewegung erkannt
→ Licht 10 %
```

### Szenario 3 – Es wird dunkel, Person ist bereits im Raum

```text
Präsenz = ON
Außenhelligkeit fällt unter 100 lx
→ Licht EIN
```

### Szenario 4 – Es wird dunkel, niemand ist im Raum

```text
Präsenz = OFF
Außenhelligkeit fällt unter 100 lx
→ nichts passiert
```

### Szenario 5 – Person kommt erst später in einen dunklen Raum

```text
Außenhelligkeit = 50 lx
Präsenz = OFF

Person kommt in den Raum
Präsenz = ON
→ Licht EIN
```

### Szenario 6 – Außen wird wieder heller

```text
Außenhelligkeit > 150 lx
→ Licht bleibt unverändert
```

Das ist absichtlich so. Ein hellerer Außenwert soll niemals eine laufende Beleuchtung abschalten.

## Voraussetzungen

- Home Assistant
- Blueprint-Unterstützung
- Bewegungs-/Präsenzsensoren als `binary_sensor`
- Zu steuernde Leuchten als `light`
- Optional: Außenhelligkeitssensor mit Lux-Wert und `device_class: illuminance`

## Datei

Die Blueprint-Datei ist:

```text
motion_presence_light.yaml
```

Sie kann beispielsweise unter folgendem Pfad abgelegt werden:

```text
/config/blueprints/automation/custom/motion_presence_light.yaml
```

Danach den Blueprint-Bereich in Home Assistant öffnen und eine neue Automation daraus erstellen.
