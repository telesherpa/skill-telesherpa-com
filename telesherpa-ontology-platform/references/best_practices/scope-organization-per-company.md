# Scope-Organisation für Firmen (Agent → Mitarbeiter)

**Status:** Empfohlenes Muster für neue Firmen-Scope-Setups im Self-Service.
**Frage:** Wie organisiert man Scopes so, dass Mitarbeiter genau das dürfen, was sie sollen —
welche Scopes legt der Firmenagent an, was kommt rein, wer bekommt welche Rechte?

---

## 1. Der Kern: Rechte laufen über Scopes, nicht über Rollen

Auf der Plattform gibt es keine frei definierbaren Datenzugriffs-Rollen. Alle Plattformnutzer
teilen dieselbe Grundrolle; **wer was darf, ergibt sich ausschließlich aus den
Scope-Mitgliedschaften:**

- `can_read` — lesen
- `can_write` — schreiben
- `can_manage_scope` — verwalten (einladen, Rechte setzen, Sichtbarkeit)

Es gibt also nur *eine* Achse für Rechte-Unterscheidung — und das ist der Scope.

**Konsequenz:** Wer A darf und B nicht, muss über Scopes getrennt werden. Es gibt keine zweite
Achse, über die man das ausdrücken könnte.

**Und daraus folgt das eigentliche Problem:** Schreibrecht auf einen Scope gilt für **alle
Objekte** dieses Scopes, ohne Ausnahme. Ein Scope pro Firma heißt: alle Mitarbeiter dürfen alles.

## 2. Der Ausweg: Lesen und Schreiben sind ZWEI Achsen

Man muss nicht alle in denselben Scope stecken. Zugriff wird auf getrennten Wegen gegeben:

| Achse | Wirkung |
| --- | --- |
| **Schreiben** (`can_write`) | schreiben auf alle Objekte des Scopes |
| **Lesen** (`can_read`) | lesen aller Objekte des Scopes |
| **Verwalten** (`can_manage_scope`) | Owner: einladen, Rechte setzen, Sichtbarkeit |
| **Buchbarkeit** | wer den Scope selbst buchen darf (`read_allowed_for`) |
| **Opt-out** | Firma schaltet einzelne Aktionen ab |
| **Feld-Ebene** | einzelne Felder nur für bestimmte Scope-Halter |

Ein Objekt liegt in **genau einem** Scope. Aber **lesbar** kann ein Scope für Halter anderer
Scopes sein. Also: **Schreiben ist exklusiv, Lesen ist teilbar.** Das ist die ganze Konstruktion.

## 3. Der eigentliche Mechanismus: der Zwei-Phasen-Commit ist deine Objekt-Berechtigung

Das ist der wichtigste Punkt. Es gibt bereits einen Commit beim Absenden:

| Phase | Antwort-Scope | Wer darf schreiben |
| --- | --- | --- |
| **Entwurf** (`status=draft`) | Schreib-Scope des Ausfüllenden | der Mitarbeiter |
| **Absenden** (`status=sent`) | Ziel-Scope der Aktion (`target_scope`) | nur, wer dort schreiben darf |

Der Mitarbeiter hat auf dem **Ziel-Scope kein Schreibrecht** — deshalb kann er die fertige
Antwort nach dem Absenden nicht mehr ändern, obwohl er sie erstellt hat. **Das ist die
Berechtigung**, ohne eine einzige Objekt-ACL: die Antwort *wandert* beim Absenden aus seinem
Schreib- in einen Lese-Scope.

Der Commit läuft serverseitig aus der Konfiguration (nie aus dem Request) und verschiebt in
einer Transaktion Objekt + Properties + Links + Dateien + Subform-Zeilen. **Wichtig:** ohne die
Datei-Kaskade sieht der Kunde die Antwort ohne Unterschrift/Fotos.

**`target_scope` ist eine Entscheidung, keine Pflicht** — siehe Abschnitt 8.

## 4. Vorschlag: DREI Scopes pro Firma (Grundmuster, nicht starr)

```
<firma>_stammdaten   Stammdaten: Standorte, Kundenobjekte, Anlagen
<firma>_arbeit       Arbeitsbereich: Entwürfe, laufende Besuche
<firma>_archiv       Ziel-Scope: abgeschlossene Antworten
```

Warum genau diese drei — und nicht einer oder zehn:

- **`stammdaten`** — die Objekte, *an* denen gearbeitet wird. Muss von allen gelesen werden können
  (der Außendienst muss den Standort sehen), soll aber nicht von allen geändert werden.
- **`arbeit`** — hier leben **Entwürfe**. Schreiben ja, aber niemand außerhalb sieht Halbfertiges.
  Getrennt von `stammdaten`, damit ein Entwurf **nie** versehentlich im Kundenbestand auftaucht.
- **`archiv`** — hier landen abgeschlossene Antworten. Nur Backoffice/Agent schreiben hier.

**Das ist ein Grundmuster für den Ein-Team-Fall, keine feste Zahl.** Wenn zwei Teams sich nicht
gegenseitig in Entwürfen sehen sollen, braucht jedes Team einen eigenen `arbeit`-Scope — die Formel
ist eigentlich **1× `stammdaten` + 1× `archiv` + N× `arbeit`** (eins pro Team mit
Entwurfs-Trennung), nicht starr drei. **Schreib-Granularität = Scope-Granularität.**

**Das Standard-Kontingent an Scopes ist bewusst knapp bemessen** — eine Firma mit einem Team kommt
damit aus. Wächst die Firma auf mehrere Teams, wird mehr benötigt; zusätzliche Scopes sind über
Credits erhältlich, alternativ gibt es ein Flat-Modell ohne Obergrenze. Wer an das Kontingent
stößt, hat also nicht einen Fehler gefunden, sondern den Punkt erreicht, an dem sich die Nutzung
bezahlt macht — das ist beabsichtigtes Design, kein Bug, den man umgehen müsste.

## 5. Was trägt welche Achse — die vier Ebenen

| Ebene | Wo es hingehört |
| --- | --- |
| Sichtbarkeit der Formular-Definition | `stammdaten` (oder `+PUBLIC` bei generischer Vorlage) |
| Erlaubter Objekttyp | `["building"]` (bzw. der Standort-Typ) |
| Sichtbarkeit des Buttons | `stammdaten` — Button nur an diesen Objekten |
| Datenhoheit der Antwort | `archiv` oder „Scope des übergeordneten Objekts" |

Beispiel einer Aktion:
```json
{
  "object_types": ["building"],
  "scopes": [<stammdaten-id>],
  "action": {
    "type": "form_answer",
    "form": "<form-udid>",
    "target_scope": <archiv-id>
  }
}
```

**Wichtig:** Die Button-Sichtbarkeit (`scopes`) ist **keine Sicherheitsgrenze**. Sie filtert nur,
*wo ein Button angezeigt wird*; wer eine Aktion auslöst, wird zusätzlich am Scope des Zielobjekts
geprüft. Wer eine echte Grenze braucht, muss sie dort ziehen — nicht über die Button-Anzeige.

## 6. Wer bekommt welche Rechte — die drei Muster

Der Agent ist **Scope-Owner** (`can_manage_scope=1`) und verteilt Rechte auf drei Wegen:

**(a) Invite-Codes — gerichtet, für bekannte Personen**
`create_invite_code(scope_id, can_read, can_write, can_manage_scope, allowlist, max_uses, expires_at)`
- Code-Format `XXXX-XXXX`, abtippbar (ohne 0/O/1/I/L). **Nicht geheim** — zum Weitergeben gedacht.
- Allowlist per `email` / `domain` / `glob` (z.B. Domain der Firma = alle Mitarbeiter).
- Rechte sind **additiv** (`max(code, bestehend)`) — ein Code kann nie entziehen.
- `revoke` verhindert nur neue Einlösungen; wer schon drin ist, bleibt.
- **Ein Code = ein Scope.** Braucht ein Mitarbeiter Rechte auf mehreren Scopes (z.B. Außendienst:
  `stammdaten`-read + `arbeit`-write), löst er dafür mehrere Codes ein — einen pro Scope.

**(b) Sichtbarkeit — Selbstbuchung, für offene Gruppen**
Owner setzt den Scope auf öffentlich/buchbar → er erscheint in der öffentlichen Liste →
Mitarbeiter bucht sich per `assign_public` **nur Lesen**. Schreiben ist so nicht erreichbar —
das ist Absicht.

**(c) Admin — für Fälle, die Self-Service nicht deckt**
Mehr Scopes als das Kontingent, oder ein Scope ohne Owner.

### Rechte-Matrix aufbauen — per MCP

Die Zuordnung läuft über Invite-Codes; der Agent als Owner (`can_manage_scope=1`) legt sie an
und teilt sie aus:

```
create_invite_code   scope_id=<stammdaten>  can_read=1  can_write=0
create_invite_code   scope_id=<arbeit>      can_read=1  can_write=1
create_invite_code   scope_id=<archiv>      can_read=1  can_write=0
```

Drei Codes für den Außendienst — einer pro Scope, weil **ein Code genau einen Scope** vergibt.
Für offene Gruppen (wechselnde Subunternehmer) den Domain-Eintrag in die Allowlist setzen oder
die Adressliste als XLSX über `upload_code_allowlist` hochladen (ersetzt die `email`-Einträge
dieses Codes vollständig, `domain`/`glob` bleiben).

Nachkontrolle jederzeit mit `list_members` (wer ist drin, welche Flags) und
`list_invite_codes` (welche Codes sind noch gültig). **`toggle_member_flag` ändert nur EIN
Recht** — `remove_member` löscht die ganze Zuordnung.

### Konkrete Rechte-Matrix (Empfehlung)

**Firmen-Agent / Disposition**
- `stammdaten` — write + manage
- `arbeit` — write + manage
- `archiv` — write + manage

**Backoffice / Innendienst**
- `stammdaten` — write
- `arbeit` — read (sieht laufende Aufträge)
- `archiv` — write (wertet abgeschlossene aus)

**Außendienst / Mitarbeiter**
- `stammdaten` — **read** (sieht den Standort, ändert ihn nicht)
- `arbeit` — **write** (legt Entwürfe an, lädt Unterschrift hoch)
- `archiv` — read (sieht eigene abgeschlossene Besuche)

**Nur-Leser / Auswertung**
- `stammdaten` — read
- `arbeit` — kein Zugriff
- `archiv` — read

Wichtig für den Außendienst: `stammdaten` **nur read** zu geben ist der Hebel. Genau dort sitzen die
Standorte — und niemand soll die Stammdaten ändern, während er einen Besuch dokumentiert.

## 6b. Bilder als Teil des Aufbaus (strukturierte Bildablage)

Gehört zu jedem Standort-Setup dazu und wird **vor** dem ersten Feldbesuch eingerichtet, weil
die App dann genau die Motive abfragt, die das Backoffice braucht:

```
onto_scope_structuredimage_categories       scope_id=<stammdaten>  → was gibt es schon?
onto_scope_save_structuredimage_categories  scope_id, categories=[...]
```

Empfehlung für ein Standort-Setup: 4–6 Motive, jeweils `de`+`en` als Sprachvarianten desselben
`imagename`, und **genau eines mit `thumbnail: true`** (das wird das Vorschaubild in Karten und
Listen). Motive, die sich bei jedem Besuch wiederholen, bekommen `versioned: true`.

**Der Dateiname ist der Schlüssel:** der Fotografierende (Mensch in der App) lädt unter dem
Namen hoch, der als `imagename` in der Kategorie steht. Passt der Name nicht exakt, erscheint
nur der Platzhalter — ohne Fehlermeldung. Details und ein vollständiges Beispiel:
`references/features/structured-image-ablage.md`.

## 7. Feinjustierung: Felder und Buttons

**Einzelne Felder** — einem Feld lässt sich ein Scope zuordnen, sodass es nur für Halter dieses
Scopes sichtbar ist (beim Rendern *und* beim Absenden geprüft). Damit geht z.B.: Kostenfelder nur
für Backoffice sichtbar.
```json
{ "key": "kosten", "widget": "float", "label": "Kosten", "scope_id": <archiv-id> }
```

**Einzelne Aktionen** — die Firma kann einzelne Aktionen am Scope abwählen (Opt-out), ohne dass
sich an der Konfiguration selbst etwas ändert.

**Papierkorb-Falle:** `toggle_member_flag` ändert **ein** Recht, `remove_member` löscht die **ganze**
Zuordnung. Wer „nur Schreiben wegnehmen" will, darf nicht `remove_member` benutzen — sonst ist auch
das Leserecht weg.

## 7b. Zwei Automatiken, die sich fast immer lohnen

Regeln (Trigger) und Functions sind **kein Extra** — sie ersetzen die Rückfrage per Telefon.
Beides lässt sich per MCP anlegen (`onto_trigger_create`, `onto_function_create`), im
Ziel-Scope mit `can_manage_scope=1`.

**1. Eskalation bei kritischem Befund** (Muster ist auf der Plattform live):

```json
{"on":"insert","object_types":["inspection"],"scope_id":<arbeit>,
 "condition":{"props":[{"key":"zustand","op":"eq","value":"kritisch"}]},
 "effects":[{"type":"create_object","object_type":"changeRequest",
             "name":"Eskalation: {object_name}",
             "props":{"status":"proposed","text":"Automatisch aus dem Standort-Check erzeugt."},
             "links":[{"linktype":"proposesChange","to":"{object_udid}"}]}]}
```

**2. Kennzahl aus zwei Feldern** (Function, per `run_function` in einer Regel aufrufbar):

```json
{"inputs":[{"key":"kmStand","type":"int","required":true}],
 "body":{"if":[{"\u003e":[{"field":["kmStand"]},10000]},"faellig","ok"]},
 "effects":[],"timeout_ms":2000}
```

**Zwei Regeln für den Betrieb** (beide sonst als Fehler erlebt):
- **`condition.changed` immer setzen.** Ein Formular-Submit schreibt mehrere Felder = mehrere
  Ereignisse; ohne `changed` feuert die Regel pro Feld (real: 1 Vorgang → 4 Effekte).
- **Der Dispatcher muss laufen.** Je einer für Ereignisse und für Zeitpläne. Einer deckt den
  anderen nicht ab.

Vollständig, inklusive Effekt-Typen und Operator-Liste:
`references/features/automation-and-functions.md`.

## 8. Der ehrliche Rest

- **Kein Objekt-Level-Recht.** Wer Schreibrecht auf einem Scope hat, darf *alle* Objekte darin
  ändern. Eine Ausnahme pro Objekt gibt es nicht. Wer das braucht, muss feiner schneiden
  (mehr Scopes).
- **`can_write` impliziert nicht `can_read`.** Beide Rechte getrennt setzen, sonst sieht der
  Mitarbeiter seine eigenen Entwürfe nicht.
- **Der Antwort-Scope erbt beim Entwurf** den Schreib-Scope. Ohne Ziel-Scope (`target_scope`) bleibt
  die Antwort dort liegen — dann sieht das Backoffice sie nie.
  **`target_scope` ist eine bewusste Entscheidung, keine Pflicht in jedem Fall:** Sinnvoll, wenn
  Ersteller (Entwurf) und Bearbeiter der fertigen Antwort unterschiedliche Rollen/Personen sein
  sollen (Außendienst schreibt, Backoffice wertet aus). In einem Drei-Personen-Betrieb, wo jeder
  alles bearbeiten darf (z.B. Malerbetrieb ohne Rollentrennung), ist Weglassen valide — keine
  Lücke, sondern bewusster Verzicht auf die Trennung. Nur klar sein, WARUM man sich entscheidet —
  nicht aus Versehen vergessen.
- **Sichtbarkeit ist Buchbarkeit, nicht Leserecht.** Wer lesen darf, entscheidet `can_read`.
  Verwechslung führt zu „Scope sichtbar, Objekte nicht".
- **Ein Scope kann verwaisen** (kein aktiver Manager). Verwaiste Scopes lassen sich gezielt
  auflisten, damit eine Firma nicht ohne Owner festsitzt.

## 9. Zusammenfassung in einem Satz

Nicht ein Scope pro Firma, sondern **mindestens zwei**: einer, in dem gearbeitet wird (Write für
Mitarbeiter), und einer, in dem Ergebnisse liegen (Write nur für Backoffice) — der Wechsel passiert
automatisch beim Absenden über `target_scope`, wenn ihr euch bewusst dafür entscheidet. Lesen wird
über `can_read` geteilt, Schreiben über Scope-Grenzen getrennt.

## 10. Entscheidungspunkte — was der Agent festlegen muss

Die Scope-Struktur erschließt sich aus dem Geschäft, nicht aus der Plattform. Vier Punkte
muss der Agent **bewusst** entscheiden und in der Übergabe-Notiz festhalten:

| Entscheidung | Woran es hängt |
| --- | --- |
| `target_scope` ja/nein | Sind Ersteller (Entwurf) und Bearbeiter (fertige Antwort) verschiedene Personen? |
| Anzahl `arbeit`-Scopes | Wie viele Teams dürfen sich **nicht** gegenseitig in Entwürfe sehen? |
| Scope öffentlich (`PUBLIC`) | Soll Fremdnutzung automatisch Credits gutschreiben (Marketplace)? |
| Rechte-Matrix | Wer liest Stammdaten, wer schreibt wo, wer verwaltet? |

Details zu `target_scope` in Abschnitt 8 — es ist eine **Entscheidung, keine Pflicht**. Ein
weggelassenes `target_scope` ist gültig, aber nur als bewusste Wahl: die Antwort bleibt dann im
Schreib-Scope, und das Backoffice sieht sie nie. Der Unterschied zwischen „bewusst weggelassen"
und „vergessen" zeigt sich erst Monate später — deshalb gehört die Begründung schriftlich in die
Übergabe-Notiz (`references/best_practices/first-30-minutes.md`).
