# Sensor-Zeitreihen

Zeitreihen sind der dritte Datentyp neben Objekt-Eigenschaften und Beziehungen: **viele
Messpunkte über die Zeit** an einem Objekt. Typische Fälle: Fahrzeug-Telemetrie (Position,
Geschwindigkeit), Pegelstände, Temperaturkurven, Flottenbewegung.

## Das Modell in drei Sätzen

- Ein **Objekt** hat eine oder mehrere **Reihen**. Eine Reihe ist ein Strom von Punkten
  `(ts, value_float, value_str, lat, lng)`.
- Eine Reihe hat **keine eigene udid**. Ihre Identität ist das Paar `(object_udid, series_key)`.
- Der `series_key` ist **derselbe String** wie der Key des zugehörigen Property. Das Property
  ist der **Zeiger** auf die Reihe (Def-Feld `computed = {"series": true}`) und trägt selbst
  keinen Wert — der letzte Wert steht als `series_ts`/`series_lat`/`series_lng` daneben.

## Die zwei Tools

| Tool | Wofür |
|---|---|
| `onto_series_range` | **Verlauf**: die Punkte zwischen zwei Zeitpunkten (Kurve, Tagesroute) |
| `onto_series_last` | **Jetzt**: die aktuellen Werte mehrerer Objekte/Reihen in EINEM Aufruf |

Welche Reihen ein Objekt hat, steht in `onto_object_show` im Feld `series_keys`.

### `onto_series_range`

Pflicht: `object_udid` + `series_key`.

**`resolution` ist der Kniff.** `0` = jeder Rohpunkt. `>0` = Raster in Sekunden (Minimum 60).

- Tagesroute → `300` (5 Minuten): eine handliche Punktzahl, Form bleibt erkennbar.
- Messkurve → `900` oder `3600`.
- Ohne Raster kommen **alle** Rohpunkte. Bei feiner Auflösung sind das schnell sehr viele, und
  die Antwort ist auf `MAX_RANGE_POINTS` begrenzt — `count` kleiner als erwartet heißt: Fenster
  verkleinern oder gröber rastern.

Antwort: `{count, resolution, points[{ts, value_float, value_str, lat, lng}]}`. Bei einem
`geo`-Sensor tragen `lat`/`lng` die Position, `value_*` ist dann `null`.

### `onto_series_last`

`objects` und `series` sind **Listen** (max 1000 Objekte). Damit ist eine ganze Flotte in einem
Aufruf versorgt — statt N Einzelabfragen.

## Zeit: alles UTC

`points[].ts` und `series_ts` sind **UTC in ISO 8601 mit `Z`** — `2026-10-06T23:09:00Z`. Auch
`from`/`to` werden als UTC gelesen. **Nicht selbst umrechnen:** wer UTC erst in Ortszeit
verwandelt, um zwei Werte zu vergleichen, baut genau den Fehler ein, den die Umrechnung
vermeiden sollte. Vergleiche UTC mit UTC.

Die **Web-Ansicht** zeigt dagegen Betrachterzeit. `01:09` in der Oberfläche und `23:09` in der
API sind derselbe Zeitpunkt (Europe/Berlin = UTC+2 im Sommer) — zwei Stunden Zonenunterschied,
kein Fehler.

## Echte Use-Cases (nachgemessen)

### 1. „Wo stehen alle Fahrzeuge jetzt?" — Flotte in EINEM Aufruf

**Nachgemessen** in einem Scope mit **10 Objekten** (Typen `car`, `form`, `invoice`, `file`,
`trigger`, `damageReport`), Telemetrie-Reihen `location` (geo) und `speed`:

```json
{"objects": ["<udid1>", "…", "<udid10>"], "series": ["location", "speed"]}
```

Ergebnis: `count: 2`, also **zwei Werte** — ein Objekt mit zwei Reihen:

```
location  ts=2026-10-06T23:09:00Z  lat=52.502494  value_float=null
speed     ts=2026-10-06T23:09:00Z  lat=null      value_float=26.2
```

**Lernpunkt — die stille Falle:** 10 Objekte gefragt, 2 Werte bekommen. Das ist **kein
Fehler**: neun Objekte haben in diesen Reihen schlicht keinen Punkt. Wer „Anzahl Antwort =
Anzahl Objekte" erwartet, hält eine korrekte Antwort für kaputt und sucht an der falschen
Stelle. Die Antwort ist eine Liste *vorhandener Messwerte*, keine Objektliste.

Ebenso wichtig: **eine leere Liste ist ein gültiges Ergebnis.** Ein Fahrzeug, das gerade nichts
gesendet hat, taucht nicht auf — es ist nicht verschwunden.

### 2. Tagesroute eines Fahrzeugs

1. `onto_series_range`, `series_key: "location"`, `from`/`to` auf den Tag, `resolution: 300`
2. Die Punkte tragen `lat`/`lng` → direkt in ein Karten-Polygon
3. Längere Zeiträume gröber rastern (`900`, `3600`), sonst greift `MAX_RANGE_POINTS`

Zum Maßstab: dieselbe Reihe hatte in einem Fenster von 2,5 Stunden **22 Punkte bei
`resolution: 300`** — der Verlauf bleibt lesbar, die Antwort bleibt klein.

**Lernpunkt:** `resolution` ist die Antwort auf „die Antwort ist zu groß" — nicht das Fenster
immer weiter verkleinern.

### 3. Messkurve ausdünnen statt abschneiden

Ein Sensor liefert im 1-Minuten-Takt, ein Chart braucht einen Überblick:
`resolution: 900` → 15-Minuten-Mittel. Die Kurve zeigt den Verlauf, nicht das Rauschen.
Das Raster rechnet **serverseitig** — die Ausdünnung kostet keinen zusätzlichen Transfer.

## Was schiefgeht (und warum es stumm ist)

- **Leere Antwort ist kein Fehler.** Siehe Use-Case 1 — der häufigste Fehlschluss.
- **`series_key` vertauscht.** Der Property-Key ist richtig; wer die udid der Reihe sucht,
  findet nichts — eine Reihe hat keine.
- **Fremder Scope → 403.** Die Sichtbarkeit richtet sich nach dem Scope des **Property**, nicht
  des Objekts. Das Tool ist sichtbar, die Ausführung wird trotzdem abgelehnt (fail-closed).
- **Rohpunkte über einen langen Zeitraum.** Ohne `resolution` kommt alles — bis die Begrenzung
  greift und `count` kleiner ausfällt als erwartet.
- **`ts` als Ortszeit missverstanden.** Der Wert endet auf `Z`; er ist UTC. Nicht umrechnen,
  nur anzeigen — oder anzeigen lassen.
