# Regeln & Functions — Automatik auf der Plattform

Die Plattform hat einen **Automatik-Layer**: Regeln (im System „Trigger" genannt) reagieren auf
Änderungen an Objekten, Functions sind benannte Berechnungen/Effekt-Bündel, die Regeln aufrufen
können. Beides sind **Ontologie-Objekte**, kein separates Schema.

> **Regeln und Functions lassen sich auch über MCP ANLEGEN.** Lesen, Ausführen
> **und Schreiben** gehen über MCP (`onto_trigger_create/_update/_delete`,
> `onto_function_create/_update/_delete`). Frühere Fassungen dieser Referenz sagten „anlegen nur
> über HTTP/Admin" — das ist überholt.

## Die Bausteine

| Objekttyp | ID | Definition liegt als | Zweck |
|---|---|---|---|
| `trigger` | 25 | Property `triggerdef` | Regel: „wenn X passiert, tu Y" |
| `function` | 26 | Property `functiondef` | benannte Berechnung + Effekte |

Wie bei `actiondef` und `formdef` gibt es **keinen eigenen Schema-Eintrag** dafür — bewusst.
Die Automatik liest genau diese Property; die GUI schreibt nichts anderes.

**Jede Objekt-Schreibung erzeugt ein Ereignis** (Art der Änderung, betroffenes Objekt, Feld,
alter und neuer Wert, Auslöser). Der Abarbeiter arbeitet diesen Puffer ab.

## Ablauf

```
Objekt wird geschrieben
    → der Server legt ein Ereignis an
        → der Abarbeiter laeuft (kurzer Takt) und zieht passende Regeln
            → lädt aktive trigger-Objekte, die passen (event + object_type + Feld)
                → prüft condition (deklarativ, KEIN SQL)
                    → führt effects aus: set_property | create_object | create_link | run_function
                        → Effekt erzeugt Event → nächste Runde (max_depth 3)
```

Zwei Betriebsarten — **zwei Timer, nicht einer**:

| Betriebsart | Auslöser | Abarbeiter | Takt |
|---|---|---|---|
| `on_change` (insert/update/delete) | Objekt-Änderung | eigener Abarbeiter | kurz |
| `schedule` (zeitgesteuert) | Zeitplan | eigener Abarbeiter | länger |

Jeder Pfad hat **seinen eigenen** Abarbeiter — einer deckt den anderen **nicht** ab. Läuft nur
der zeitgesteuerte, sieht man ereignisgesteuerte Regeln nie feuern.

Status und Rückstand sind abfragbar (offen / gesamt / fehler / ok / skip). Der Status-Endpunkt
ist nicht von aussen erreichbar.

## Aufbau einer Regel (`triggerdef`)

```json
{
  "on": "update",
  "object_types": ["radioTower"],
  "scopes": [<SCOPE_ID>],
  "enabled": true,
  "condition": {
    "changed": ["status"],
    "props": [ { "key": "status", "op": "eq", "value": "defect" } ]
  },
  "effects": [
    { "type": "set_property", "key": "priority", "value": "high" },
    { "type": "run_function", "function": "wartung_faellig" }
  ]
}
```

**Erlaubte `on`-Werte:** `insert`, `update`, `delete`, `schedule`.

**Erlaubte `effects[].type`:** `set_property`, `create_object`, `create_link`, **`run_function`**.
`notify` und `webhook` sind **absichtlich nicht erlaubt** — sie würden still wirkungslos bleiben.
Der Validator lehnt sie beim Speichern mit Klartext ab.

**`condition.changed`** — nur feuern, wenn sich eines dieser Felder geändert hat.
Bei einem Vorgang, der mehrere Felder schreibt (z.B. Formular-Submit), ist das die
wirksamste Bremse gegen Mehrfachfeuer.

**`condition.props[]`** — Wertvergleich gegen den **aktuellen Objektzustand** (nicht gegen das
Event). Operatoren: `eq`, `neq`, `in`, `gt`, `gte`, `lt`, `lte`, `empty`, `notempty`, `matches`.
Fail-closed: ein unbekannter Operator gilt als **nicht erfüllt** (nicht als „immer wahr").

## Regel/Function über MCP anlegen (der heutige Weg)

```
onto_function_index  → gibt es die Function schon? (Name oder udid)
onto_trigger_index   → welche Regeln laufen im Scope?
```

**Function anlegen** — `onto_function_create`:

```json
{"name":"onto_function_create","arguments":{
  "name":"wartung_faellig",
  "scope_id":<SCOPE_ID>,
  "body":{
    "inputs":[{"key":"kmStand","type":"int","required":true}],
    "body":{"if":[{"\u003e":[{"field":["kmStand"]},10000]},"faellig","ok"]},
    "effects":[],
    "timeout_ms":2000
  }
}}
```

**Regel anlegen** — `onto_trigger_create`:

```json
{"name":"onto_trigger_create","arguments":{
  "name":"meine_regel",
  "scope_id":<SCOPE_ID>,
  "body":{
    "on":"insert",
    "object_types":["inspection"],
    "condition":{"props":[{"key":"zustand","op":"eq","value":"kritisch"}]},
    "effects":[{"type":"create_object","object_type":"changeRequest","name":"Eskalation: {object_name}",
                "props":{"proposedKey":"zustand","text":"Automatisch erzeugt"},
                "links":[{"linktype":"proposesChange","to":"{object_udid}"}]}]
  }
}}
```

Rückgabe: `{udid, key, scope_id}` (der `udid` ist die Kennung für `_update`/`_delete`).

- **`object_types` ist Pflicht und darf nicht leer sein** — die Server-Validierung lehnt eine
  leere Liste mit „ohne Objekttyp wirkt die Regel nie" (400) ab, und jeder genannte Typ muss
  aktiv vorhanden sein.
- **`scopes` wird von der Anlage aus `scope_id` gefüllt** — der Aufrufer muss es nicht mitgeben.
  Die Liste ist die Erlaubnis für die Ausführung, siehe Sicherheit.
- `scope_id` ist Pflicht; du brauchst dort **`can_manage_scope`** (403 sonst). Der
  oeffentliche Scope nur durch Admin.
- `key` wird auf `[a-zA-Z0-9_]` bereinigt und muss eindeutig sein (sonst 409).
- `_delete` ist ein **Soft-Delete** (`status=deleted`) — umkehrbar.
- **`_create` und `_update` sind Schema-Schreibvorgänge**: sie ändern, was die Plattform
  automatisch tut. Schreibrecht (`can_write`) genügt bewusst **nicht**.

Platzhalter in Werten: `{object_udid}`, `{object_name}`, `{value_new}`, `{value_old}`.

## Aufbau einer Function (`functiondef`)

```json
{
  "inputs": [
    { "key": "objekt", "type": "object",   "required": true },
    { "key": "schwelle", "type": "int",    "required": false }
  ],
  "body": { "cat": ["Wartung fällig: ", { "field": "name" }] },
  "effects": [ { "type": "set_property", "key": "note", "value": "faellig" } ],
  "timeout_ms": 2000
}
```

- **`inputs[].type`:** `object`, `string`, `int`, `float`, `bool`, `datetime`, `list`.
- **`body`** ist ein **AST für den Ausdrucks-Parser** — es gibt keinen zweiten
  Interpreter. Die Validierung läuft beim Speichern; ein unbekannter Operator ist ein
  **Fehler beim Anlegen**, nicht erst beim Ausführen.
- **`timeout_ms`:** 100–30000, sonst Fehler.
- **Konvention:** der Eingabeschlüssel `objekt` trägt die Ziel-UDID
  (siehe `onto_function_run`: `inputs` = `{key: wert}`).

### Operator-Katalog (Whitelist)

Alles außerhalb dieser Liste wird beim Speichern abgelehnt.

- **Auswahl/Daten:** `var`, `source`, `field` (Alias für `var`)
- **Graph-Navigation:** `link`, `linklist` — über Links auf verknüpfte Objekte zugreifen
- **Aggregation über Listen:** `count_of`, `sum_of`, `max_of`, `min_of`
- **Logik:** `and`, `or`, `!`, `if` (2–3 Argumente)
- **Vergleich:** `==`, `!=`, `===`, `!==`, `<`, `<=`, `>`, `>=`, `in`, `notin`,
  `empty`, `notempty`, `matches`
- **Rechnen:** `+`, `-`, `*`, `/`, `%`, `min`, `max`, `abs`, `round`
- **Text:** `cat`, `lower`, `upper`, `len`, `trim`
- **Formularspezifisch:** `sum_over`, `count_over`

Der AST hat **immer** die Form `{"operator":[argumente]}` — also `{"-":[a,b]}` für a minus b,
nicht `{"op":...}`.

`link`/`linklist` nehmen ein **Konfigurationsobjekt**, keinen Ausdruck:
```json
{ "link": { "type": "performs", "dir": "in", "prop": "kmReading" } }
```
Ein Tippfehler im Linktyp fällt beim Speichern auf, wenn der Aufrufer die gültigen Linktypen
mitgibt (z.B. aus den Link-Definitionen) — sonst still ein leerer Wert.

## Sicherheit (fail-closed, alle geprüft)

- **Sichtbarkeit pro Zeile** über die Scope-Verwaltung — kein Admin-Bypass.
- **Ziel-Scope muss verwaltet werden** (`can_manage_scope=1`); der öffentliche Scope nur Admin.
  Geprüft wird serverseitig pro Scope, fail-closed.
- Der Ziel-Scope ist **die Erlaubnis**: er bestimmt, auf welche Objekte die Regel wirken darf.
  Wird er zu weit gefasst, greift die Regel überall in diesem Bereich. Eng fassen.
- **`max_depth` (Default 3)** gegen Endlosschleifen (Effekt erzeugt Event erzeugt Effekt).
- **Dedupe 60 s** gegen Wiederholungsfeuer — verglichen wird `(Trigger, Objekt, Feld, **Wert**)`,
  über einen Fingerabdruck. Der **Wert** muss mit verglichen werden: sonst werden echte
  Wertewechsel verworfen.
- **Rollen-Whitelist pro Definition** (Default `telesherpa`).
- **Automatik kostet KEINE Credits** — `charge_credits` läuft nur bei Auslösung durch einen Nutzer.
- **Der Endpunkt ist nicht öffentlich:** er ist von aussen nicht erreichbar, Aufruf intern.

### Ein Effekt-Typ außerhalb der Whitelist wird beim Speichern abgelehnt
Kein stiller No-Op. Umgekehrt: eine `run_function`-Referenz auf eine nicht existierende Function
wird ebenfalls beim Speichern abgewiesen, damit eine Regel nicht feuert und nichts tut.

## Zugriffswege — der wichtige Unterschied

| Weg | Lesen | Schreiben |
|---|---|---|
| **MCP** | `onto_trigger_index`, `onto_function_index`, `onto_function_run` | `onto_trigger_create/_update/_delete`, `onto_function_create/_update/_delete` |
| **HTTP (Web/Admin)** | in der Weboberfläche unter Automatik | dieselbe Funktion wie die MCP-Tools |

Beide Wege enden auf **derselben** Funktion — MCP ist der Wrapper darum. Ein Agent braucht
die Weboberfläche damit **nicht mehr**. Die sechs Schreib-Tools sind für die Rolle `telesherpa`
sichtbar (kein Admin nötig); die echte Sperre ist `can_manage_scope` pro Ziel-Scope.

**Die Tool-Namen enden auf `_create`/`_update`/`_delete`, NICHT `_save`.** Ein Client, der nach
`_save` sucht, findet nichts und meldet fälschlich „Tool fehlt".

Ausführen geht über MCP: `onto_function_run` mit `function` (UDID **oder** Name) und `inputs`.

## Beispiel: eine Regel mit `create_object` und Link

`onto_trigger_index` listet die vorhandenen Regeln, `onto_function_run` (oder ein Lauf über den
Abarbeiter) zeigt, was passiert ist. Muster für `create_object` mit Link:

```json
{"on":"insert","object_types":["inspection"],"scopes":[<SCOPE_ID>],
 "condition":{"props":[{"key":"zustand","op":"eq","value":"kritisch"}]},
 "effects":[{"type":"create_object","object_type":"changeRequest",
             "name":"Eskalation: kritischer Zustand {object_name}",
             "props":{"proposedKey":"zustand","status":"proposed",
                      "text":"Automatisch vom Trigger erzeugt: Standort-Check {object_name} wurde im strukturierten Feld 'zustand' als 'kritisch' eingestuft."},
             "links":[{"linktype":"proposesChange","to":"{object_udid}"}]}]}
```

## Pitfalls

1. **`on_change` braucht den Abarbeiter — und der ist eigenständig.** Eine Regel wird nicht
   ausgeführt, nur weil ein Objekt geschrieben wird. Läuft der Abarbeiter nicht, sammeln sich
   Ereignisse und nichts passiert. Bei „meine Regel tut nichts" zuerst den Status prüfen.
2. **Formular-Antworten erzeugen Ereignisse** (Auslöser „Formular"). Bei einem NEUEN
   Schreibweg immer ein echtes Ereignis erzeugen und prüfen, dass es ankommt (nicht per
   SQL-INSERT nachbauen — ein selbstgebautes Ereignis prüft nur die eigene Annahme).
3. **Mehrfachfeuer pro Vorgang (der „4x-Bug").** Ein Submit schreibt N Felder → N Events.
   `condition.props` wird gegen den AKTUELLEN Objektzustand geprüft, nicht gegen das Event — also
   sieht jedes der N Events das fertige Objekt. Die Dedupe greift nicht (verschiedene `key_name`
   = verschiedene Hashes). **Abhilfe:** `condition.changed` setzen — dann greift ein Gate im
   Abarbeiter und dieselbe Submission erzeugt nur einen Effekt statt vier.
4. **Der Scope ist die Berechtigung, nicht die Heimat.** Eine Regel wirkt nur auf Objekte, deren
   Scope in der Erlaubnisliste der Regel steht. Wird ein Objekt in einen anderen Scope
   verschoben, greift die Regel dort nicht mehr.
5. **Dedupe-Fenster 60 s.** Zwei echte Änderungen desselben Feldes auf denselben Wert
   innerhalb 60 s → nur eine Ausführung. Bei einem erwarteten zweiten Lauf auf einen
   **anderen Wert** ändern.
6. **`set_property` in Effekten nutzt `props` (Objekt), NICHT `key`/`value`.** Ein Body mit
   key/value wird von der Validierung nicht als falsch erkannt, tut aber nichts.
7. **`create_link` braucht `linkvalue`** — fehlt er, scheitert der **ganze Effekt**, nicht nur
   der Link. Bei `create_object.links` sind beide Schlüsselnamen gültig (`type` und `linktype`);
   jeder übersprungene Link wird im Action-Log vermerkt.
