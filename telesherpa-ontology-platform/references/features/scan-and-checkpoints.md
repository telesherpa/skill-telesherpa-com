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

Die mitgelieferte Oberfläche bedient die Route `GET /ontouser/scan` (Rolle `member`) — Kamera, Ton,
Rückmeldung, Ergebnisliste und Knöpfe, mit Cache-Buster an den eigenen Dateien. Der **Vertrag ist
der Server**, nicht die Seite — dieselben Aufrufe funktionieren aus jeder Oberfläche.

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

## Anwendungsfall: Event-Einlasskontrolle (Typ `ticket`)

Am Eingang wird das gedruckte Ticket gescannt. Ein **Code genügt** — es gibt kein Gegenüber-Objekt,
denn der Gast muss nichts vorzeigen außer dem Ticket.

| Teil | Konfiguration (erprobt) |
|---|---|
| Aktion | `qr_admit`, Label „Einlass: Ticket einlassen", `object_types: ["ticket"]`, Rolle `member` |
| Bedingung | `admitted ne "ja"` → Meldung **„Dieses Ticket wurde schon eingelassen"** |
| Wirkung | `admitted = "ja"`, `admitted_at = {now}`, Verknüpfung `concerns` von `{object_udid}` nach `{user_person}` mit `replace: "source"` (wer das Ticket gerade hält) |
| Zähler | **„Schon eingelassen"**: aktive Tickets mit `admitted = "ja"` |
| Rückmeldung | drei Stufen, `dedupe_ms: 2000` (derselbe Code binnen 2 s bucht nicht doppelt) |
| Gegenstück | `qr_leave`, Label „Auslass: Ticket verlassen": Bedingung `admitted eq "ja"` → **„Dieses Ticket ist nicht auf der Veranstaltung"**; setzt `admitted = ""`, `left_at = {now}` und löst die Verknüpfung (`remove: true`, `require: "present"`) |

Was daraus folgt — und warum es so gebaut ist:

- **Die Bedingung ist die Kontrolle, nicht die Oberfläche.** Der zweite Scan wird serverseitig
  abgewiesen und sagt in Klartext, warum. Am Eingang ist das der wichtigste Fall: er kommt
  fünfhundertmal am Abend vor.
- **Der Zustand `admitted` ist die Wahrheit**, die Verknüpfung zeigt nur, wer das Ticket zuletzt
  hatte (`replace: "source"` — der letzte Halter gewinnt).
- **Zähler zählen Zustand, nicht Ereignisse.** „Schon eingelassen" ist die Zahl der Tickets im
  Zustand eingelassen. Wer „aktuell drin" will, muss die Ausgelassenen abziehen (`left_at`
  berücksichtigen) — sonst steht am Abend die Gesamtzahl aller Einlässe da.
- **`qr_leave` läuft nie automatisch**: es löst eine Verknüpfung, also gibt es Knöpfe. `qr_admit`
  läuft bei einem Code sofort (nur `remove` erzwingt Bestätigung, `replace` nicht — wer eine
  bewusste Tipp-Bestätigung will, setzt `scan.ausfuehrung: "bestaetigen"`).
- **Ein Ticket ohne Veranstaltungsbezug ist wertlos.** `event` gehört gefüllt, sonst zählt der
  Zähler Tickets aus allen Veranstaltungen zusammen.
- **Was der Nachweis nicht kann:** er zeigt Ticket, Zeit und Konto — nicht, dass die Person
  anwesend war, und nicht, dass niemand mit zwei Tickets zweimal eintritt. Wer das braucht,
  arbeitet mit Person statt Ticket (eigener Auftrag).

## Anwendungsfall: Chaotische Lagerhaltung (Typen `package`, `storage`)

Kein fester Platz: **das Paket bekommt seinen Platz in dem Moment, in dem es hingestellt wird.**
Der Platz ist ein Objekt (`storage`) mit eigenem aufgeklebten Code — eine Regalnummer, ein
Bodenfeld, eine Kiste.

| Teil | Konfiguration (erprobt) |
|---|---|
| Einchecken | `qr_shelf_in`, Label „Regal: Paket einchecken", `object_types: ["package"]`, Prompt `place` (`required`, `object_types: ["storage"]` → **Objektauswahl**, der Client füllt die UDID ein) |
| Verknüpfung | `concerns` von `{object_udid}` nach `{place}`, `replace: "source"`, `require: "absent"` → Meldung **„Das Paket ist an diesem Platz schon eingecheckt"** |
| Auschecken | `qr_shelf_out`: setzt `place = ""` und löst dieselbe Verknüpfung (`remove: true`, `require: "present"`) → **„Das Paket ist an diesem Platz nicht eingecheckt"** |

Was daraus folgt:

- **Zwei Codes, aber die Reihenfolge ist frei.** Der Client kann Paket zuerst und Platz zuerst
  scannen — der Server prüft beide Wege und legt fest, worauf ausgeführt wird. Wer als erstes
  scannt, ist keine Bedienregel, die man dem Lager erklären muss.
- **Der Regalcode ist die Adresse.** Es gibt keine Stammdatenpflege vorab: Platzobjekt anlegen,
  Code drucken, ankleben. Deshalb funktioniert „chaotisch" überhaupt — das Regal muss nicht in
  einer Liste stehen, es muss nur einen Code tragen.
- **`require: "absent"` / `"present"` machen den Fehler sichtbar.** Doppelte Einbuchung oder
  Ausbuchung am falschen Regal enden in einem Satz, nicht in einem stillen Datenfehler.
- **Auschecken läuft nie automatisch** (`remove`) — dort wird bewusst getippt. Einchecken darf
  nach dem Prompt sofort laufen.
- **Ein Zustand, nicht zwei.** Aktuell führt die Einbuchung nur die Verknüpfung, die Ausbuchung
  räumt zusätzlich das Feld `place` am Paket. Das ist ein Widerspruch in der Quelle der Wahrheit:
  entweder Verknüpfung **oder** Feld pflegen. Ein Zustand, den man doppelt führt, fällt irgendwann
  auseinander. (Erbe: das Testpaket trägt noch den alten Wert `platz_udid` — nicht deklariert,
  Aufräumen wäre eine Datenänderung.)
- **Kein Platz pro Paket im Modell.** Die Aussage „Paket P liegt in Kiste K" steckt in der
  Verknüpfung, nicht in einer Liste. Auswertungen („wie viele Pakete in dieser Kiste") zählen über
  die Verknüpfung; ein Zähler auf der Regalseite ist noch nicht gebaut.

## Nachweis

- Quelle ist `onto_action_logs`. Für den Kontrollnachweis zählt jede Ausführung mit
  `action = 'qr_control'`.
- Die **Bewertung gehört in den Report, nicht in den Scan**: Abstand zur vorigen Kontrolle am
  selben Punkt gegen `interval_min`, dazu die Wegzeit seit der vorigen Kontrolle an einem **anderen**
  Punkt desselben Kontos — auffällig kurze Wegzeiten entlarven Abhaken ohne Ortswechsel.
- Zähler im Actiondef (`counter`) und Interface-SQL setzen **`{scope_where}`** ein; die Plattform
  ersetzt den Platzhalter dort mit den Lese-Scopes des Aufrufers (fail-closed: ohne Lese-Scope
  bleibt `AND 1=0`).
- **Im Report-SQL wird `{scope_where}` NICHT ersetzt.** Steht der Platzhalter in der Report-Abfrage,
  bricht die Abfrage mit einem SQL-Syntaxfehler ab; fehlt jede Scope-Bedingung, wertet der Report
  **über alle Scopes** aus. Wer einen Report baut, schreibt den Scope darum ausdrücklich hin — die
  Plattform scoped nicht für ihn.
- Filterwerte sind **Fremdeingabe**. Für die eingebauten Filter behandelt die Plattform sie
  serverseitig korrekt; für **selbst geschriebenes** SQL (Report-Abfrage, Interface-SQL) gilt das
  nicht automatisch — dort müssen Spaltennamen aus der eigenen Definition stammen und Werte über die
  Escape-Funktion der Datenbank laufen.
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
4. **Der Client cached.** Nach jeder Änderung an einer Client-Datei den Cache-Buster prüfen
   (`?v=<Dateizeit>`); sonst testet der Nutzer die alte Fassung.
5. **Der Fehlertext gehört dem Nutzer.** `message`/`text` in Klartext formulieren, keine Codes.
6. **`replace` erzwingt keine Bestätigung — nur `remove` tut das.** Wer beim Einlass einen bewussten
   Tipp will, verlässt sich nicht auf die Verknüpfungsart, sondern setzt `scan.ausfuehrung:
   "bestaetigen"`. Umgekehrt gilt: eine Aktion ohne `remove` läuft bei genau einem Kandidaten
   sofort — auch wenn sie eine bestehende Verknüpfung ersetzt.
7. **Derselbe Zustand an zwei Stellen ist ein Fehler mit Ansage.** Verknüpfung **und** Feld
   (`place`) gleichzeitig zu führen, klingt harmlos und fällt erst auf, wenn die beiden
   auseinanderlaufen. Eine Quelle der Wahrheit festlegen, die andere nur lesen.
