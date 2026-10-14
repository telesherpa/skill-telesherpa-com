# Kontrollgänge per QR aufbauen — Arbeitsanweisung mit Nachweis

**Ziel:** Ein Wach- oder Sicherheitsdienst quittiert seine Rundgänge am Ort per QR-Code, und danach
gibt es einen Nachweis, der auch einer Prüfung standhält — je Kontrolle eine Zeile mit Zeit, Punkt,
Konto und einer Plausibilitätsbewertung.

**Für wen:** Betreiber/Administrator. Objekttypen sind Plattform-Schema (`onto_type_create` ist
admin-only); Kunden-Agenten arbeiten mit den Typen, die da sind, und bauen darauf Aktionen,
Ansichten und Reports.

Das technische Verhalten steht in `references/features/scan-and-checkpoints.md` (Server-Vertrag,
Ausführungsregel, Feldformate). Hier steht die Reihenfolge, in der man so etwas aufbaut.

## 0. Entscheidungen vor dem Bauen (nicht raten)

| Frage | Warum sie vorher geklärt wird |
|---|---|
| Welche Rundgänge (`tour`) und Bereiche (`section`) gibt es? | Aus ihnen entsteht die Liste der Punkte, nicht umgekehrt |
| Wie oft muss jeder Punkt kontrolliert werden (`interval_min`)? | Bestimmt die Bewertung „zu früh / überfällig" im Nachweis |
| Wer darf quittieren? | Rolle und Scope; `PUBLIC` im Scope bedeutet Lesen, **nicht** Mitscannen |
| Was passiert bei einer ausgefallenen Kontrolle? | Heute nur sichtbar im Report. Ein Alarm ist ein eigener Auftrag (Trigger/Function) |
| Muss bewiesen werden, **wer** geprüft hat, oder auch **wo**? | Heute ist die Geräteposition **nicht** Teil des Nachweises. Wer sie braucht, muss das ausdrücklich beauftragen |
| Wieviel Rückmeldung ist zumutbar? | Ton, Farbe und Haptik je Stufe; zu viel Rückmeldung wird weggehört |

## 1. Typ und Felder anlegen (Admin)

```
onto_type_create        name="checkpoint", label="mo.Checkpoint",
                        description="Kontrollpunkt fuer den Rundgang: QR-Tag am Ort"
onto_property_def_create key="code",         value_type="string",   label="mo.Code"
onto_property_def_create key="title",        value_type="string",   label="mo.Title"
onto_property_def_create key="section",      value_type="string",   label="mo.Section"
onto_property_def_create key="tour",         value_type="string",   label="mo.Patrol"
onto_property_def_create key="interval_min", value_type="int",      label="mo.Interval_Min"
onto_property_def_create key="checked_at",   value_type="datetime", label="mo.Checked_At"
onto_property_def_create key="checked_by",   value_type="string",   label="mo.Checked_By"
onto_property_set_create key="checkpoint_props", label="mo.Checkpoint_Props",
                        def_ids=[…], type_ids=[<Typ-ID von checkpoint>]
```

Regeln dazu:

- **Feldnamen englisch, Beschriftung als i18n-Schlüssel** (`mo.…`). Existiert der Schlüssel nicht,
  zeigen Oberfläche und Introspektion den Rohschlüssel. Erst `onto_i18n_index`/`onto_i18n_matrix`
  lesen, dann `onto_i18n_create`; die Annahme/den Export macht der Betreiber.
- **Adresse nicht nachbauen:** das Set `address` (`street`, `postalCode`, `city`, `latitude`,
  `longitude` mit Geocode- und Reverse-Layout) existiert und wird an den Typ gebunden.
- `checked_at`/`checked_by` sind **Zustandsfelder**: sie stehen am Objekt, damit eine Liste „wann
  zuletzt geprüft" ohne Log-Auswertung funktioniert. Der Verlauf bleibt trotzdem im Protokoll.
- Pflichtfelder (`required`) sparsam setzen — ein Pflichtfeld blockiert jede Ausführung.

## 2. Punkte anlegen und Codes drucken

```
onto_object_create      type="checkpoint", name="KP-01 Haupteingang",
                        props={code:"KP-01", title:"Haupteingang", section:"Erdgeschoss",
                               tour:"Rundgang Nord", interval_min:60}
```

- Der QR-Code enthält die **UDID** des Objekts (32 Hex-Zeichen), sonst nichts. Kein Klartext, keine
  URL mit Parametern: die UDID ist der Schlüssel, die Bedeutung steht in den Properties.
- Codes **haltbar** drucken: ein abgerissener oder überklebter Code ist der häufigste Ausfall.
- Ein Punkt, der nie kontrolliert wird, gehört nicht in den Rundgang — sonst bewertet der Nachweis
  dauerhaft „überfällig".

## 3. Aktion bauen

```
onto_actiondef_save   key="qr_control", label="Kontrolle quittieren",
                      object_types=["checkpoint"], scopes=[<Kundenscope>], roles=["member"],
                      body={"action":{"type":"update",
                                      "props":{"checked_at":"{now}","checked_by":"{user_email}"}},
                            "scan":{"ausfuehrung":"auto"},
                            "counter":{"label":"Kontrollen heute","sql":"… {scope_where}"},
                            "feedback":{…}}
```

- **Keine Verknüpfung** in dieser Aktion: ein Kontrollpunkt hat kein Gegenüber, also genügt ein
  Scan, und die Aktion darf sofort laufen (`scan.ausfuehrung: "auto"`).
- Eine Aktion, die eine Verknüpfung **löst** (z. B. Paket aus dem Regal), läuft **nie** automatisch —
  das erzwingt der Server, unabhängig von `ausfuehrung`.
- `condition` immer serverseitig formulieren (z. B. „nur wenn der Auftrag offen ist"), mit
  `message` in Klartext für den Nutzer.
- Rückmeldung dreistufig: Scan angenommen, Ergebnis, Fehler. Zu jeder Stufe Ton, Farbe, Haptik und
  Dauer; `dedupe_ms` gegen Doppelscans.

## 4. Abnehmen (am laufenden Dienst, nicht am Schreibtisch)

1. **Introspektion:** `onto_type_show` am Typ — Felder und Aktionsschlüssel müssen erscheinen,
   Beschriftungen in der Zielsprache.
2. **Ernstfall:** je Rundgang einen echten Scan ausführen (`QR-Code → Aktion`).
   Danach am Objekt prüfen: `checked_at` gesetzt, `checked_by` = Konto.
3. **Protokoll:** `onto_action_logs` muss je Ausführung eine Zeile mit Aktionsschlüssel, Objekt,
   Konto und `scope_id` tragen.
4. **Ausführungsregel:** eine lösende Aktion muss Knöpfe zeigen, eine harmlose sofort laufen.
5. **Gegenprobe Negativfall:** zweimal derselbe Code innerhalb der Dedupe-Frist darf nicht doppelt
   buchen; ein unbekannter Code darf keine Aktion anbieten.
6. **Nachweis:** Report öffnen und gegen die eigenen Scans lesen.

## 5. Nachweis (Report „Kontrollen")

Je Zeile: Zeit, Kontrollpunkt, Scope, Konto, Ergebnis, Abstand zur vorigen Kontrolle am selben
Punkt, Soll aus `interval_min`, Bewertung, voriger Punkt, Wegzeit, Rundgang.

- Bewertung im SQL, **nicht im Scan**: `erste Kontrolle` (kein Vorgänger), `zu frueh` (< Soll),
  `ueberfaellig` (> 2 × Soll), sonst `ok`. Der Faktor ist eine Vereinbarung mit dem Auftraggeber —
  er gehört schriftlich festgehalten, nicht stillschweigend gesetzt.
- **Wegzeit** = Minuten seit der letzten Kontrolle desselben Kontos an einem **anderen** Punkt.
  Auffällig kleine Werte (unter zwei Minuten) sind der beste Hinweis auf Abhaken ohne Ortswechsel —
  deshalb steht die Spalte im Nachweis und nicht in einer Sonderauswertung.
- Filter: Scope, Kontrolleur (E-Mail), Zeitraum. Filterwerte gehen **nur per POST** in der
  Reihenfolge der `search_cols`; die JSON-/GET-Route ignoriert sie.
- `candownload = 1` gibt den CSV-Export frei (für Prüfungen und Akten).

## 6. Missbrauch im Blick behalten

Der Nachweis ist nur so gut wie seine Nebenrechnung. Deshalb von Anfang an mitliefern:

- Wegzeit-Spalte (Abhaken ohne Ortswechsel),
- Zähler im Actiondef („Kontrollen heute") als Sofort-Kontrolle für den Schichtleiter,
- `scope_id` im Protokoll, damit die Zeilen dem Kunden zuordenbar bleiben,
- bei Bedarf ein zweiter Code je Punkt (Gegenstand + Ort), wenn der Vorgang eine Verknüpfung
  braucht.

Was der Nachweis **nicht** kann, und das gehört in die Übergabe: er beweist Zeit, Konto, Objekt und
Reihenfolge — nicht die Anwesenheit am Ort. Für Anwesenheit braucht es die Geräteposition (eigener
Auftrag, siehe `references/features/scan-and-checkpoints.md`).
