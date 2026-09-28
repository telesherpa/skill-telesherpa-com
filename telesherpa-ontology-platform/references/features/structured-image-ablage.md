# Strukturierte Bildablage

Die strukturierte Bildablage organisiert Bilder (z.B. Fotos von Standorten) in Kategorien pro
Scope. Bilder sind Datei-Objekte, verknüpft mit dem übergeordneten Objekt.

## Konzept

- Jeder Scope hat eine Menge definierter **Kategorien** (z.B. "Aussenansicht",
  "Schlüsseltresor / Position").
- **Wichtig für `upload_file_to_object`:** Der `name`-Parameter (der Kategorie-Dateiname) muss
  **exakt** dem `imagename` der Ziel-Kategorie entsprechen — nur dann landet das Bild in der
  richtigen Kategorie. Ohne exakten Match wird das Bild nicht der Kategorie zugeordnet.
- Nur eine Kategorie ist als Thumbnail markiert — deren Bild erscheint als Vorschau/Titelbild
  des Objekts (z.B. in Karten- oder Listenansichten). Andere Kategorien sind zusätzliche
  Dokumentation, erscheinen aber nicht als Vorschaubild.

## Die Kategorien lassen sich über MCP lesen UND setzen

Dafür gibt es eigene Tools — man muss die Kategorien nicht mehr über die
Weboberfläche pflegen:

| Tool | Zweck |
|---|---|
| `onto_scope_structuredimage_categories` | Kategorien eines Scopes **lesen** (`scope_id`) |
| `onto_scope_save_structuredimage_categories` | Kategorien **schreiben** (`scope_id` + `categories`) |
| `onto_file_structured` | Bildliste **eines Objekts** (welche Kategorie ist gefüllt, welche leer?) |
| `upload_file_to_object` | Bild hochladen (`type: structuredimage`, `name` = imagename) |

Alle Scope-/Datei-Tools sind für die Rolle `telesherpa` sichtbar; das Schreiben braucht
zusätzlich **`can_manage_scope`** auf dem Scope (403 sonst).

### Kategorien lesen

```json
{"name":"onto_scope_structuredimage_categories","arguments":{"scope_id":<SCOPE_ID>}}
```

### Kategorien setzen

```json
{"name":"onto_scope_save_structuredimage_categories","arguments":{
  "scope_id":<SCOPE_ID>,
  "categories":[
    {"imagename":"01_<Motiv>.jpg","caption":"<Bezeichnung>","language":"de",
     "info":"<Beschreibung>","thumbnail":true,"versioned":false},
    {"imagename":"01_<Motiv>.jpg","caption":"<Label>","language":"en",
     "info":"<Description>","thumbnail":true,"versioned":false},
    {"imagename":"02_<Motiv>.jpg","caption":"<Bezeichnung>","language":"de",
     "info":"<Beschreibung>","thumbnail":false,"versioned":false}
  ]
}}
```

Feldregeln je Eintrag:

- **`imagename`** — Pflicht. MUSS dem Dateinamen des Uploads entsprechen (siehe unten).
- **`caption`** — Anzeigename in der Oberfläche/App.
- **`info`** — Fotografier-Hinweis an den Menschen (was soll aufs Bild?).
- **`language`** — `de` / `en`. **Mehrere Einträge mit gleichem `imagename` sind Sprachvarianten**
  derselben Kategorie, keine Duplikate.
- **`thumbnail`** — `true` = dieses Bild ist das Vorschaubild des Objekts. Mehrere `true`
  sind möglich (dann entscheidet die Reihenfolge bzw. das vorhandene Bild).
- **`versioned`** — `true` = mehrere Aufnahmen derselben Kategorie werden als Versionen
  aufbewahrt (z.B. wiederkehrende Prüfungen); `false` = ein Bild je Kategorie.
- **Die Reihenfolge der Liste bestimmt die Anzeige-Reihenfolge.**

**Der Schreibvorgang ersetzt die Liste vollständig** — wer nur eine Kategorie ergänzen will,
liest erst, hängt an und schreibt die ganze Liste zurück. Sonst sind die übrigen Kategorien weg.

## Der Namens-Match ist der entscheidende Punkt

Der Slot-Match läuft über `filename == imagename`. In der Praxis heißt das:

| Datei (Upload-`filename` **und** `name`) | Kategorie-`imagename` | Ergebnis |
|---|---|---|
| `01_<Motiv>.jpg` | `01_<Motiv>.jpg` | ✅ Slot gefüllt |
| `<Motiv>.jpg` | `01_<Motiv>.jpg` | ❌ nur Platzhalter |
| `IMG_4711.jpg` | `01_<Motiv>.jpg` | ❌ nur Platzhalter |

Ein Präfix (`01_`, `02_` …) ist **Teil des Namens** und muss mit. Vor dem Bau einer eigenen
Namenskonvention immer diesen Match gegenprüfen.

## Beispiel: Standort dokumentieren (Agent, ohne Schreibrecht am fremden Scope)

```
onto_scope_structuredimage_categories  scope_id=<scope>     → welche Kategorien gibt es?
onto_file_structured  object_id=<udid>                       → welche ist noch leer?
upload_file_to_object type=structuredimage, name=<Kategorie-Dateiname> → hochladen
```

Liegt der Ziel-Scope nur lesend vor, landet der Upload über die SSR-Kaskade im **eigenen**
Schreib-Scope — die Kategorie-Zuordnung am fremden Objekt funktioniert trotzdem.

## Beispiel: typischer Kategorien-Satz

Ein Kategorie-Satz besteht aus je einer Datei pro Motiv und Sprache (erfundenes Beispiel):

```
01_<Motiv>.jpg         de "<Motiv>"          en "<Motive>"        thumbnail=true
02_<Motiv>.jpg         de "<Motiv>"          en "<Motive>"        thumbnail=true
03_<Motiv>.jpg         de "<Motiv>"          en "<Motive>"        thumbnail=true
```
Alle mit `versioned: false`. Umfangreichere Motiv-Sätze sind möglich.

## Weitere Einstellungen (Scope, nicht Kategorie)

Zwei Scope-Rechte steuern, wer überhaupt strukturierte Bilder ablegen darf:
`can_upload_structuredimage` und `can_upload_file`.

Ein Nutzer ohne dieses Recht bekommt beim App-Upload einen Platzhalter statt eines Fehlers —
bei „mein Bild erscheint nicht" zuerst die Kategorien prüfen (Match!), dann dieses Flag.
