# Referenz-Ontologien und Standard-Mapping

**Für wen:** Ein Agent, der einem Unternehmen Objekttypen und Properties konfiguriert — und
wissen muss, wie das Datenmodell der Plattform zu etablierten Standards steht.

**Wozu:** Kunden kommen aus SAP, ServiceNow, Planon, Salesforce. Wenn ein Begriff
wiedererkennbar ist, verkürzt das die Migration. Diese Datei ist die Übersetzungstabelle.

**Wichtig — Rollen:** Objekttypen, Property-Sets und Property-Defs anzulegen ist
**`admin`-Sache** (`onto_type_create`, `onto_property_set_create`, `onto_property_def_create`
sind `roles=["admin"]`). Ein normaler Firmen-Agent kann das Schema **lesen**, aber nicht
erweitern. Was er tun kann: die **vorhandenen** Typen und Properties vollständig nutzen.

---

## 1. Der Katalog-Gedanke

Die Plattform stellt einen **großen, standardkonformen Katalog** bereit, aus dem der Kunde
auswählt — statt dass jeder Kunde sein eigenes Modell erfindet. Das ist der Unterschied zu
SAP/ServiceNow: dort bekommt man ein Modell, das man erst aufwendig anpassen muss. Hier ist die
Auswahl die Konfiguration.

**Beim Konfigurieren deshalb:** bevor du einen eigenen Typ vorschlägst, prüfe, ob es im Katalog
schon einen standardkonformen gibt. Ein selbst erfundener Typ ist die zweitschlechteste Lösung;
die schlechteste ist zwei ähnliche Typen nebeneinander.

---

## 2. Wie das Modell der Plattform aussieht

**Der Katalog umfasst zwei Stände:**
- **Bestandstypen** — gewachsene Objekttypen
- **Katalogtypen (Familie Raum/Standort)** — standardkonform ergänzt:
  `site`, `floor`, `zone`, `space`, `outdoorArea`, `land`

| Objekttyp | Entspricht (Standard) |
| --- | --- |
| `building` | Brick `Building` · REC `Building` · ISA-95 `Site`-Ebene |
| `site` | Brick `Site` · ISA-95 `Site` · REC `RealEstate` |
| `room` | Brick `Space` · REC `Room` · ISA-95 `Work Center` |
| `space` | Brick `Space` · REC `Space` |
| `floor` | Brick `Floor` (**Etage**) · REC `Level` |
| `zone` | Brick `Zone` (HVAC/Lighting/Energy/Fire) |
| `outdoorArea` | Brick `Outdoor_Area` · REC `OutdoorSpace` |
| `land` | REC `Land` |
| `machine` | Brick `Equipment` · REC `Device` · ISA-95 `Equipment` |
| `tool` | ISA-95 `Equipment` (Werkzeug) |
| `person` | Brick `Occupant` · REC `Agent`/`Person` · ISA-95 `Personnel` |
| `car` | ISA-95 `Equipment` (Fahrzeug) · FIWARE `Vehicle` |
| `aed` | Brick **`Automated_External_Defibrillator`** (eigene Klasse!) |
| `baseStation` | TM Forum `Resource` · SAREF4CITY |
| `radioTower` | TM Forum `Resource` · SAREF4CITY |
| `serviceVisit` | FIWARE `Service` · TM Forum `Service` |
| `serviceOrder` | TM Forum `Order` · ISA-95 `Process Segment` |
| `inspection` | TM Forum `Service` (Prüfung) |
| `changeRequest` | OSLC Change Management |
| `orderItem` | ISA-95 `Material` · TM Forum `Product` |
| `file` | REC `Information` |
| `comment` | — |
| `form`, `function`, `trigger`, `actiondef` | Plattform-Mechanik, kein Standard-Pendant |

**Belegt:** Brick führt **`Automated_External_Defibrillator`** (Klasse `Safety_Equipment`) —
die Plattform ist in diesem Punkt bereits standardkonform. Das ist ein Verkaufsargument,
kein Zufall. **Achtung:** die Klasse heißt **nicht** `AED` — dieser Kurzname existiert in Brick
nicht. In ODTB (REC 3.3) fehlt eine Defibrillator-Klasse ganz.

---

## 3. Was in den Standards existiert und hier (noch) fehlt

**Familie Raum/Standort ist angelegt.**
`site` · `floor` · `zone` · `space` · `outdoorArea` · `land` — mit Property-Sets und
standardkonformen Labels (`mo.Site`, `mo.Floor_Level`, `mo.Zone`, `mo.Space`,
`mo.OutdoorArea`, `mo.Land`).

**i18n-Falle bei den Labels (wichtig):**

- Das Typ-Label ist `mo.X`, der **Übersetzungsschlüssel ist `X` — ohne Präfix.** Wer den
  Schlüssel mit `mo.` anlegt, erzeugt einen Eintrag, den niemand findet — der Typ zeigt dann
  den Roh-Schlüssel im Dropdown.
- **`Site` ist bereits belegt** (deutsch „Gelände", aus einem früheren Modell). `create`
  lehnt einen existierenden Key ab. Der Objekttyp `site` nutzt daher den **bestehenden**
  Wert; soll er „Liegenschaft" heißen, ist das eine **Änderung** am Bestands-Key (mit
  Nebenwirkung auf bestehende Daten) — bewusst zu entscheiden, nicht nebenbei.
- `Zone`, `Space`, `OutdoorArea`, `Land`, `Floor_Level` waren frei.

**Status:** Typen sind `active` und über `onto_type_show` vollständig abrufbar. Die deutschen
Texte stehen in der DB; die 15 Übersetzungen (en/fr/it) warten auf `review` + `export` —
das sind **Mensch-Schritte**, nicht Agent-Schritte.

**Noch offen:**

| Lücke | Standard-Begriff | Heutiger Zustand |
| --- | --- | --- |
| **Anlage als Oberbegriff** | Brick `Equipment`, REC `Device` | nur `machine`/`tool` — kein gemeinsamer Oberbegriff |
| **Sensor** | REC `Sensor`, Brick `Point` | fehlt (`machine` ist Anlage, kein Messpunkt) |
| **Messwert** | Brick `Point`/`Setpoint`, REC `Observation` | `samples`, `range` als Properties auf `radioTower` |
| **Zone** | Brick `Zone` | **angelegt** (Katalogtyp, muss pro Kunde freigeschaltet werden) |

**Konsequenz für die Konfiguration:** Etagen und Zonen gibt es jetzt als Katalogtypen — sie
müssen aber pro Kunde **freigeschaltet** werden (siehe Abschnitt 4b). Sie sind nicht
automatisch in jedem Kunden sichtbar.

---

## 4. Das Geodaten-Muster (Pflicht für Standorte)

**Das ist die Falle, die jeden Agenten trifft.** Ein Standort ohne Koordinaten ist nicht
kartenfähig — und die UI bietet einen Geocoding-Knopf nur an, wenn die Daten da sind.

**Für jeden kartenfähigen Typ gehören diese fünf Properties dazu:**

```
latitude · longitude · city · postalCode · street
```

Vorhanden und über `onto_type_show` sichtbar bei: `building`, `car`, `person`, `radioTower`,
`aed` (korrigiert).

**Werkzeuge:** `onto_object_geocode` mit `direction: geocode` (Adresse → Koordinaten) oder
`direction: reverse` (Koordinaten → Adresse). Mit `apply: true` wird direkt geschrieben.

**Regel:** Erst Adresse, dann geocodieren, dann prüfen ob `latitude`/`longitude` tatsächlich
geschrieben wurden. Nicht annehmen — `apply: true` kann stillschweigend nichts tun.

---

## 4b. Katalogtypen freischalten — PRESETS

**Das Problem:** Objekttypen sind **global** definiert. Ein großer Katalog würde jedem Kunden
alles zeigen.

**Die Lösung: Presets.** Der Kunde wählt **Bündel**, nicht einzelne Typen.

```
Preset-DEFINITION (global, von Telesherpa gepflegt):
  Name + Liste der enthaltenen Objekttypen

Preset-AUSWAHL (pro Kunde):
  Auswahl der Preset-Namen für den eigenen Scope
```

**Mehrere Presets sind möglich** — die Typ-Mengen werden **vereinigt**. Damit lässt sich die
Plattform für mehrere Geschäftsfälle gleichzeitig nutzen (z. B. Funknetz **und** Fuhrpark).

**Verfügbare Presets:**

| Preset (interner Name) | Typen |
| --- | --- |
| `basis` | building, room, person, file |
| `gebaeudeverwaltung` | building, room, floor, zone, space, site, land, outdoorArea, machine, aed, person, file |
| `funknetz` | radioTower, baseStation, site, person, file |
| `feldservice` | machine, tool, aed, person, serviceOrder, building, room, file |
| `fuhrpark` | car, person, file, serviceOrder |
| `vollstaendig` | alle 18 fachlichen Typen (ohne Plattform-Mechanik) |

**Wert-Format der Preset-Definition:** `{"label":"mo.Preset_X","types":[...]}`.
Ein reines Array (Altform) wird weiterhin gelesen — das Label fällt dann auf den
internen Namen zurück. Der interne Name (`funknetz`) ist stabil; nur das Label ist
übersetzbar.

**Labels sind i18n-Keys — und brauchen `export`.** Ist das Label eines Typs zum Beispiel
`mo.Preset_Telecom`, dann ist der i18n-Key `Preset_Telecom` **ohne Präfix** (siehe
Abschnitt 3, i18n-Falle). Solange die Übersetzungen nicht exportiert sind, zeigt die
Oberfläche den Roh-Key (`mo.Preset_Telecom`) statt der Übersetzung — das ist kein
Fehler, sondern der offene Export-Schritt.

**Verhalten:**
- **Kein Eintrag** für einen Scope → **alle** aktiven Typen sichtbar (Bestandsverhalten).
  Das ist Absicht: die Funktion kann keine bestehende Installation lahmlegen. Ein Kunde,
  der nichts wählt, merkt nichts.
- **Eintrag vorhanden** → nur die Typen der gewählten Presets erscheinen.
- Mehrere Scopes eines Users werden vereinigt (additiv).

**Wo es wirkt:** in der Objektansicht (Typ-Filter) und in der App bei der Typ-Auswahl.
Gefiltert über die serverseitige Typ-Auflösung bzw. das App-Pendant.

**Zwei Wege, dieselbe Funktion — Mensch und Agent sind gleichgestellt:**

| Weg | Endpoint |
| --- | --- |
| **Mensch** (Browser) | in der Weboberfläche: Katalog-Auswahl mit Checkboxen und Speichern |
| **Agent** (MCP) | `onto_catalog_presets`, `onto_catalog_index`, `onto_catalog_apply`, `onto_catalog_clear` |

Rechte: **scope-basiert** (`can_manage_scope`), FAIL-CLOSED, kein
Admin-Bypass. Wer einen Scope verwalten darf, darf dessen Presets setzen.

**Wo der Einstieg im Menü sitzt:** unter **Einstellungen** (nicht im Usermenü, nicht im
Admin-Bereich) — es ist eine Konfiguration des Kunden für seinen eigenen Scope, keine
tägliche Arbeit. Das Menü ist Konfiguration, kein Code.
Ein Eintrag ist `{"label":"mo.Catalog","url":"<Pfad>","required_role":"..."}`.

**Begriffe (verbindlich):**

- **Menü-Label:** `mo.Catalog` → „Katalog" (kurz, damit die Sidebar ruhig bleibt)
- **Seitentitel:** `mo.Catalog_Title` → „Objekttypen-Katalog" (präzise, eigenes i18n-Key!)
- **Seiten-Überschriften:** `mo.Catalog_Presets`, `mo.Catalog_Selection`, `mo.Catalog_Hint`

Warum **nicht** „Ontologie-Katalog": „Ontologie" benutzt die Plattform nur als Kopf/Titel,
nie als Arbeitsbegriff des Kunden. Und „Katalog" allein ist unspezifisch — es gibt daneben
Link-Definitionen, Property-Definitionen und Property-Sets. „Objekttypen-Katalog" ist
eindeutig und reiht sich sauber neben „Typen verwalten" ein.

**Typ-Namen im Katalog werden als Label gezeigt**, nicht technisch: `Gebäude (building)`,
`Person (person)`, `Funkmast (radioTower)`. Sonst widerspricht der Katalog der
Admin-Typenliste, die ebenfalls Label + Code zeigt. Der Helfer liefert dafür
`types_detail` (Array aus `{name,label}`) zusätzlich zu `types` (Array von Namen).

**Sichtbarkeit:** Der Menü-Eintrag ist für alle mit Rolle `telesherpa` sichtbar — das ist
praktisch jeder Kunde. Das ist vertretbar, weil die Rechte pro Scope geprüft werden:
ohne `can_manage_scope` ist die Seite **lesbar**, aber nicht änderbar.

**Ehrlich dazu:** Wer keinen Scope verwaltet, sieht eine Seite, auf der er nichts ändern
kann. Für einen echten Self-Service-Kunden ist das richtig (er verwaltet seinen Scope).
Für Fremd-Kunden ohne eigenen Scope ist der Eintrag Rauschen — dann wäre `required_role`
oder eine Scope-Bedingung im Menü der Hebel, nicht der Controller.

**Die Presets pflegt Telesherpa, nicht der Kunde.** Neue Presets sind ein Produktentscheid.
Der Kunde wählt nur aus, was da ist.

**Properties werden NICHT separat gefiltert.** Wer einen Typ freischaltet, bekommt seine
Properties mit — sie hängen am Typ. Das Filter-Dropdown der Objektliste zeigt weiterhin alle
Defs: ein Objekt kann Properties tragen, die gar nicht im Schema stehen (Formular-
felder werden als Properties geschrieben, ohne Def). Eine Filterung würde diese Werte
unsichtbar machen.

### i18n-Review im UI: Trefferflächen

Die Annehmen/Ablehnen-Buttons der Review-Queue waren `btn-xs` mit **nur Icon** — rund 22×18 px,
direkt nebeneinander. Auf dem Telefon führt das zu Fehlgriffen (echt passiert: „Ablehnen"
statt „Annehmen" bei `Catalog_Title`/fr_FR). Regel für mobile Aktionen:

- **Immer Text zusätzlich zum Icon** — ein Icon allein ist keine Beschriftung.
- **Mindestens 44 px Trefferhöhe** (hier `padding:12px; font-size:15px`).
- **Auf schmalen Schirmen gestapelt** (`flex-direction:column`, volle Breite, `gap:10px`),
  damit zwei gegensätzliche Aktionen nie nebeneinander liegen.
- Die Beschriftungen kommen aus bestehenden Keys (`mo.Accept` = „Akzeptieren",
  `mo.Reject` = „Ablehnen") — keine neuen Keys für vorhandene Begriffe erfinden.

**Ablehnen ist kein Datenverlust:** ein abgelehnter Vorschlag bleibt als Zeile mit
`status='rejected'` erhalten, der freigegebene Bestand wird nicht angetastet. Ein erneutes `translate` mit
demselben Key/Locale legt einen neuen `pending`-Vorschlag an (der Review-Endpoint nimmt nur
`pending` an). Es ist also reparabel — aber nur, wenn man es bemerkt.

**Falle bei `i18n create`:** Der Endpoint lehnt belegte Keys ab (`Key existiert bereits`) und
es gibt **keinen** Update-Endpoint. Wer Menü-Label und Seitentitel ändern will, braucht
deshalb **zwei getrennte Keys** (`mo.Catalog` fürs Menü, `mo.Catalog_Title` für die Seite) —
nicht denselben Key zweimal belegen.

**Wichtig für den Agenten:** Fehlt ein Katalogtyp im Dropdown, obwohl er `status='active'`
hat, ist sein Preset für diesen Kunden nicht freigeschaltet. Das ist Konfiguration, kein Bug —
nicht durch Anlegen eines zweiten, ähnlichen Typs umgehen. Stattdessen das passende Preset
freischalten.

---

## 5. Sets und Schema sind zwei Quellen — die wichtigste Falle

Die Plattform zeigt Properties auf **zwei** Wegen, und sie waren historisch nicht deckungsgleich:

| Weg | Quelle | Wer sieht es |
| --- | --- | --- |
| **Schema-Introspektion** | `onto_property_def_index` | der **Agent** über `onto_type_show` |
| **Objekt-Ansicht** | `onto_property_set_index` (+ `onto_property_def_index`) | der **Mensch** in der UI |

**Zwei Zustände, die es zu verstehen gilt:**

- **`display_mode='auto'`** — das Set wird in der Objektansicht automatisch gruppiert angezeigt
- **`display_mode='manual'`** — nur in Interfaces/Cards sichtbar
- **Properties in keinem Set** — erscheinen in der UI als flache Liste unterhalb der Sets
  (in der Oberfläche: „nicht zugeordnete Felder")

**Wichtig (schon einmal aufgetreten):** Properties können in der Oberfläche sichtbar sein,
aber im Agenten-Schema fehlen — dann legt ein Agent Objekte ohne diese Werte an.

**Regel:** Wenn ein Property in der UI erscheint, aber in `onto_type_show` fehlt (oder
umgekehrt), ist das ein **Zuordnungsfehler** — melde ihn, umgehe ihn nicht durch Raten.

---

## 6. Property-Typen und was sie können

Die UI bietet: `string` · `int` · `float` · `datetime` · `bool` ·
`options`.

- **`options`** braucht JSON: `[{"value":"1","label":"Ja"},{"value":"0","label":"Nein"}]`
- **`datetime`** kennt relative Defaults: `0` (heute), `+30`, `-365` (Tage)
- **`computed`** macht ein Feld berechnet statt manuell (JSON-Formel).
  Beispiel aus dem Bestand: `serviceStatusComputed`, `maintenanceOverdue`
- **`required`** ist derzeit selten gesetzt — die meisten Felder sind optional

**Labels** sind `mo.*`-Keys: sie laufen über die Sprachdateien des Servers (4 Sprachen),
**nicht** über die i18n-Verwaltung der Plattform. Neue Labels anlegen: `create` + `translate`
(Agent), `review` + `export` (Mensch).

---

## 7. Standard-Herkunft und Lizenz (wichtig für Kompatibilitäts-Aussagen)

Verifiziert an den LICENSE-Dateien der Repositories bzw. den Ontologie-Dateien:

| Standard | Lizenz | Nutzbar als |
| --- | --- | --- |
| **Brick Schema** | **BSD-3-Clause** (Brick Consortium, Inc.) | Klassen/Properties übernehmen, Attribution |
| **RealEstateCore** | **BSD-3-Clause** (RealEstateCore Consortium) | Klassen/Properties übernehmen, Attribution |
| **Azure opendigitaltwins-building** | **MIT** (Microsoft Corporation) | Klassen übernehmen, Attribution |
| **FIWARE Smart Data Models** | **CC BY 4.0** — belegt über die Gruppen-Repos und die `contribution-agreement.md` der FIWARE-Organisation (nicht Teil dieses Skills). **Einschränkung:** die einzelnen `dataModel.*`-Repos haben keine eigene LICENSE.md (GitHub meldet `NOASSERTION`), und die Badge im Umbrella-README ist im HTML auskommentiert | Entitäten übernehmen, **Namensnennung nötig** |
| **SAREF (ETSI)** | **ETSI Software License = BSD-3-Clause** — via `dcterms:license` in jeder TTL. **Achtung:** der normative Spezifikationstext (TS 103 264 PDF) steht **nicht** darunter („© ETSI 2025. All rights reserved."), dort kommt kein Lizenzbegriff vor | Ontologie-IRIs/Serialisierungen frei, **Spezifikationstext nicht** |
| **ISA-95 (IEC 62264)** | **proprietär** (Norm 454–470 USD). **Frei:** die B2MML-XSDs von MESA International (Attribution genügt) | Normtext nicht, XSD-Schemata ja |
| **TM Forum SID** | **proprietär** für die Models Suite (nur Mitglieder). **Frei:** ITU-T M.3190 (frei abrufbar) und die **Open APIs (Apache 2.0)** | SID-Begriffe über ITU/Open APIs, Models Suite nicht |
| **OSLC** | **CC BY 4.0 + Apache 2.0** (OASIS OSLC Open Project) | frei, kein Login |
| ServiceNow CSDM · SAP ODM · Planon | proprietär | nur als Referenz lesen, **nicht** übernehmen |
| Deloitte `sap-ontology` | **CC BY-SA 4.0** | **Vorsicht:** ShareAlike — abgeleitete Werke müssen gleich lizenziert sein |

**Regel:** Bei einer Kompatibilitäts-Aussage nach außen immer die Lizenz prüfen. „Wir sind
kompatibel zu X" ist eine andere Aussage als „wir haben X übernommen". Bei Brick und REC die
Konsortien namentlich nennen — Attribution ist Lizenzbedingung.

---

## 7b. Größenordnung der Standards (für Erwartungsmanagement)

| Standard | Umfang | Fassung |
| --- | --- | --- |
| Brick Schema | **1.469 Klassen**, 82 ObjectProperties + 8 DatatypeProperties | 1.5.0 |
| RealEstateCore | **270 eigene Interfaces** (+ 1.094 eingebettete Brick) | 4.1 |
| Azure opendigitaltwins-building | **766 Interfaces** | REC 3.3 (älterer Stand) |
| FIWARE Smart Data Models | **82 Repos, 1.118 Entitätstypen** | Stand 2026-07-19 |
| SAREF Core | 30 Klassen · 62 ObjectProperties | 4.1.1 (TS 103 264) |
| SAREF4BLDG | 65 Klassen · 81 DatatypeProperties | 2.1.1 |
| ISA-95 / IEC 62264 | 8 Teile + TR95.01 | — |

**Das ist die wichtigste Zahl in dieser Datei:** Brick allein hat 1.469 Klassen. Ein 1:1-Mapping
ist deshalb **kein** gangbarer Weg — es würde den Katalog unbrauchbar machen. Der Katalog greift
gezielt die Klassen heraus, die einen realen Anwendungsfall tragen.

---

## 7c. Namensfallen (vor dem Anlegen prüfen!)

**`floor` — die gefährlichste Falle:**

| Standard | `Floor` bedeutet |
| --- | --- |
| **Brick** | **Etage** (`Floor` unter `Location`, mit `Basement`, `Parking_Level`, `Rooftop`) |
| **Azure ODTB** | **Bodenplatte** (`Floor` unter `BuildingComponent`) — **nicht** die Etage! |
| **RealEstateCore** | kennt `Floor` gar nicht — dort heißt es **`Level`** (`levelNumber`) |

Wer „Floor" sagt und Etage meint, muss `floor` anlegen und in der Doku **explizit** als Etage
deklarieren. Wer das nicht tut, erbt die ODTB-Mehrdeutigkeit.

**Weitere Abweichungen zwischen den Standards:**

- `Wing` (Gebäudeflügel) existiert **nur in Brick** — nicht in REC, nicht in ODTB
- `Outdoor_Area` nur in Brick; REC/ODTB nennen es **`OutdoorSpace`**
- `Site` fehlt in ODTB (vorhanden in Brick und REC 4.1)
- `Land` nur in REC 4.1, nicht in ODTB
- **Messwerte sind fundamental verschieden modelliert:** Brick nutzt `Point`-Objekte
  (`Sensor`, `Setpoint`, `Command`, `Status`, `Alarm`, `Parameter`) mit externer
  Timeseries-Referenz; REC/ODTB nutzen **`Capability`** als Fähigkeit plus separate
  **`ObservationEvent`**-Objekte. Ein Katalog, der nur einen der beiden Wege abbildet, ist
  für den anderen Standard nicht anschlussfähig.

**Telekom-Lücke:** Für `baseStation`/`radioTower` liefern **beide** Standards fast nichts —
nur `ICT_Equipment` (Brick) bzw. `ICTEquipment` (ODTB) mit `Server`, `Gateway`, `Router`,
`WirelessAccessPoint`. Mobilfunk-spezifische Klassen fehlen. Das ist **kein** Versäumnis des
Katalogs, sondern eine Lücke der Standards — nach außen so benennen.

---

## 8. Quellen

- Brick Schema — `github.com/BrickSchema/Brick`, `bricksrc/location.py`, `bricksrc/equipment.py`
- RealEstateCore — `github.com/RealEstateCore/rec`, Doku `doc.realestatecore.io/3.2/core.html`
- Azure open digital twins building — `github.com/Azure/opendigitaltwins-building` (MIT)
- FIWARE Smart Data Models — `fiware.org/smart-data-models`
- SAREF — `saref.etsi.org`
- ISA-95 / IEC 62264 · TM Forum SID · OSLC — Standard-Dokumentation
