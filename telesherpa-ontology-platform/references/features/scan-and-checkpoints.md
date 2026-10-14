# Scan-Flows: Kontrollpunkt, Einlass, Regal (QR)

**Was du mitnimmst:** wie aus einem gedruckten QR-Code ein Vorgang wird, den die Ontologie selbst
entscheidet — mit der verbindlichen Regel, wann sofort ausgeführt wird und wann der Nutzer
bestätigen muss, und mit dem Nachweis im Aktionsprotokoll.

## Aufbau

1. Das **Objekt trägt den Zustand als Property** (z. B. `checked_at` am Kontrollpunkt).
2. Der Code am Ort oder am Gegenstand enthält die **UDID** des Objekts.
3. Der Client gibt den Scan an den Server; der Server sucht die passenden Actiondefs, prüft deren
   Bedingung und antwortet mit Kandidaten.
4. Ausgeführt wird **sofort oder per Knopf** — der Server entscheidet (siehe Regel unten).
5. Der **Nachweis** entsteht im Aktionsprotokoll `onto_action_logs`, je Ausführung eine Zeile mit
   `action` (Aktionsschlüssel), Objekt, Konto und `scope_id`.

Die Oberfläche ist eine Themenseite: `views/ontouser/scan.php` (Route `GET /ontouser/scan`, Rolle
`member`) lädt `themes/qrscan/scan.js` (Kamera, Ton, Rückmeldung) und
`themes/qrscan/scan_page.js` (Liste, Knöpfe, Zähler; Cache-Buster `?v=<filemtime>`). Der **Vertrag
ist der Server**, nicht die Seite — dieselben Aufrufe funktionieren aus jeder Oberfläche.

## Server-Vertrag (HTTP, Rolle `member`)

| Aufruf | Zweck |
|---|---|
| `POST /onto/object/match_actions` — `{"udid": "<hex32>", "partner": "<hex32>"}` | Welche Aktionen passen zu diesem Scan? **Nur Auskunft**, keine Änderung, kein Credit |
| `POST /onto/object/execute_action` | Führt einen Kandidaten aus (Kosten: `action`) |
| `POST /onto/object/counter` — `{}` | Zähler aus allen sichtbaren Actiondefs (rein lesend) |

`match_actions` liefert je Kandidat: `key`, `label`, `auf_udid` (Objekt, auf dem ausgeführt wird),
`prompt` (Feld, das die UDID des anderen Objekts bekommt), `partner`, `offene` (Pflicht-Prompts
ohne Vorbelegung), `message`, `replace`, `remove`, `ein_code`, `blockiert`, `ausfuehrung`,
`bestaetigen`.

- **Zwei Codes:** `partner` mitsenden (Paket + Regalplatz). Beide Scan-Reihenfolgen werden geprüft —
  wer zuerst gescannt wurde, spielt keine Rolle.
- **Ein Code:** angeboten werden Aktionen **ohne** Verknüpfung (Kontrollpunkt quittieren) —
  erkennbar an `ein_code: true`.
- `blockiert: true` heißt: die Bedingung (`condition`) ist verletzt, `message` erklärt es. Gesperrte
  Kandidaten werden mitgeliefert, nicht verschwiegen — der Nutzer soll sehen, warum nicht.
- `offene` muss der Client als Formular zeigen, **bevor** er ausführt.

## Ausführungsregel (Server, verbindlich)

1. genau **eine** mögliche, nicht lösende `auto`-Aktion → läuft **sofort**
2. genau **ein** Kandidat insgesamt → läuft, **außer** er löst eine Verknüpfung (`remove`)
3. sonst (mehrere Wege **oder** eine lösende Aktion) → **Knöpfe**, `bestaetigen: true`
4. `blockiert` und `remove` laufen **nie** automatisch

Merksatz: *Eine Aktion, die eine Verknüpfung löst oder deren Vorbedingung verletzt, läuft nie
automatisch.* Ausnahmen gibt es nicht, auch nicht bei `scan.ausfuehrung: "auto"`.

## Felder am Actiondef

`object_types`, `scopes`, `roles` wie gewohnt, dazu für den Scan:

- `condition`: `{ "props": [{"key","op","value"}], "sql": "…", "message": "…" }` — **immer
  serverseitig**. Eine Bedingung, die nur die Oberfläche prüft, ist keine Bedingung.
- `prompts[]`: zusätzlich `default_value`; `object_types: ["storage"]` macht aus einem Textfeld eine
  **Objektauswahl** (der Client füllt die UDID ein).
- `action.links[]`: `{ "type", "from", "to", "replace", "remove", "require", "text" }` —
  `require: "absent"` / `"present"` prüft die Verknüpfung, `text` ist die Klartext-Meldung bei
  Verstoß, `replace: "source"` ersetzt eine bestehende Verknüpfung (Paket von Regal A nach B).
- `counter`: `{ "label": "Kontrollen heute", "sql": "SELECT COUNT(*) … {scope_where}" }` — der
  Platzhalter setzt die Lese-Scopes ein. Fällt ein Zähler aus, fehlt er in der Antwort; der Scan
  läuft weiter.
- `scan`: `{ "ausfuehrung": "auto" | "bestaetigen" }` — Vorgabe `auto`, `bestaetigen` erzwingt Knöpfe.
- `feedback`: `{ "scan" | "success" | "error": {"sound","color","mode","haptic","ms"}, "silent":
  bool, "dedupe_ms": int }` — dreistufige Rückmeldung. Der Client trennt die Stufen zeitlich
  (Vorgabe 320 ms, überschreibbar über `abstand_ms`), sonst überlagern sich Ton und Farbe.
  `dedupe_ms` schluckt denselben Scan innerhalb der Frist.

## Objektmodell (erprobtes Beispiel)

| Zweck | Typ | Feldset |
|---|---|---|
| Kontrollpunkt | `checkpoint` | `checkpoint_props`: `code`, `title`, `section`, `tour`, `interval_min`, `checked_at`, `checked_by` |
| Einlass | `ticket` | `ticket_props`: `description`, `event`, `admitted`, `admitted_at`, `left_at` |
| Paket / Lagerplatz | `package`, `storage` | `package_props` (`place`), `storage_props` (`description`) |

- Adressfelder kommen aus dem bestehenden Set `address` (`street`, `postalCode`, `city`, `latitude`,
  `longitude` mit Geocode/Reverse-Layout) — nicht nachbauen.
- Einlass in zwei Aktionen: **einlassen** (`admitted = ja`, `admitted_at = {now}`, Verknüpfung
  `concerns → {user_person}` mit `replace: "source"`) und **verlassen** (`admitted = ""`,
  `left_at = {now}`, dieselbe Verknüpfung mit `remove: true`). Der Zustand `admitted` ist die
  Wahrheit, die Verknüpfung zeigt nur den Träger.
- Anlegen mit `onto_type_create`, `onto_property_def_create`, `onto_property_set_create`;
  **Objekttypen sind Plattform-Schema und damit admin-only** — Kunden-Agenten arbeiten mit den
  vorhandenen Typen (Katalog/Presets) und bauen darauf Aktionen, Ansichten und Reports.
  Feldnamen englisch, Beschriftungen als **i18n-Schlüssel ohne `mo.`-Präfix**. Ohne Export der
  Sprachdateien zeigt die Oberfläche den Rohschlüssel (`mo.Checkpoint`). Vorher prüfen, welche
  Schlüssel es schon gibt (`onto_i18n_matrix`) — `mo.Area` heißt „Areal", nicht „Bereich".

## Nachweis

- Quelle ist `onto_action_logs`. Für den Kontrollnachweis zählt jede Ausführung mit
  `action = 'qr_control'`.
- Die **Bewertung gehört in den Report, nicht in den Scan**: Abstand zur vorigen Kontrolle am
  selben Punkt gegen `interval_min`, dazu die Wegzeit seit der vorigen Kontrolle an einem **anderen**
  Punkt desselben Kontos — auffällig kurze Wegzeiten entlarven Abhaken ohne Ortswechsel.
- Zähler im Actiondef (`counter`) und Report-SQL nutzen beide `{scope_where}`.
- Der Report-Header ist eine **kommagetrennte Zeichenkette** (`explode(',')`), keine JSON-Struktur,
  und darf in einer Überschrift kein Komma enthalten. Filter kommen in der Reihenfolge der
  `search_cols` und wirken **nur per POST**; die JSON-/GET-Route ignoriert sie.
- Ein Report darf `candownload = 1` tragen, dann ist der CSV-Export freigeschaltet.

## Pitfalls

1. **Ein Kode ist nicht der andere.** Eine Aktion mit Verknüpfung braucht zwei Codes; eine ohne
   Verknüpfung wird im Paarmodus nicht angeboten. Wer beides in einer Aktion will, baut zwei
   Actiondefs.
2. **Umbenannte Aktionsschlüssel und Felder reißen die Historie**, wenn die Log-Zeilen (`action`,
   `payload`) und das Report-SQL nicht mitgezogen werden. Der Report zeigt danach 0 Zeilen,
   obwohl die Aktionen im Protokoll stehen.
3. **Plausibilität nicht in den Scan legen.** Der Scan muss schnell bleiben; die Bewertung rechnet
   der Report aus denselben Rohdaten.
4. **Der Client cached.** Nach jeder Änderung an `themes/qrscan/*.js` den Cache-Buster prüfen
   (`?v=<filemtime>`); sonst testet der Nutzer die alte Fassung.
5. **Der Fehlertext gehört dem Nutzer.** `message`/`text` in Klartext formulieren, keine Codes.
