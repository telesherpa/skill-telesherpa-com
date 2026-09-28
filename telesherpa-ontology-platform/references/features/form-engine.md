# Formularengine

Die Formularengine erlaubt strukturierte Datenerfassung über Formulare, deren Antworten als
Ereignis-Objekte in der Ontologie landen.

## Konzept

- **Formular** = Objekttyp `form` mit `formdef`-JSON (Felddefinitionen).
- **Antwort** = eigenständiges Ereignis-Objekt, Feldwerte als Properties.
- **Links:** `usesForm` (Formular→Antwort), `performs` (Person→Antwort), `concerns` (Antwort→Zielobjekt).
- **Props + Scope erben den FORMULAR-Scope.** Feld-`scope_id` = optionale Sichtbarkeit (fail-closed).

## MCP-Tools (Agenten-Workflow)

Fürs Ausfüllen per Agent (nicht App/Web): `list_formdefs` → `prepare_form_answer` →
`submit_form_answer` (ein- oder mehrfach mit `status: draft`) → `submit_form_answer`
(`status: sent`) → optional `show_form_answer` zur Kontrolle.

- **Parametername ist überall `form_udid`, nicht `formdef_udid`** — gilt für
  `prepare_form_answer`, `submit_form_answer`, `show_form_answer` und
  `resolve_form_answer`, obwohl das zugrunde liegende Objekt vom Typ `form` ist
  (formdef-JSON als Property).
- **`submit_form_answer` UND `show_form_answer` brauchen beide `form_udid` UND
  `answer_udid`** (zweistellige Route, bewusst kein automatisches Auflösen über den
  `usesForm`-Link). Nur die `answer_udid` bekannt (z.B. aus einer Antwort-Liste)?
  Erst `resolve_form_answer(answer_udid)` aufrufen, um die passende `form_udid` zu holen.
- `submit_form_answer`s `values`-Objekt landet als `answer`-Property je Feld-Key aus
  `show_formdef`; Subform-Felder bekommen eine LISTE von Zeilen-Objekten als Wert.

## Pitfalls

- Formular-Antworten sind Ereignis-Objekte (Zustandsbeschreibung zu einem Zeitpunkt), NICHT
  Mutationen auf dem Ziel-Objekt.
- Formübergreifende Aggregation läuft über den Link zur Person, nicht über ein gemeinsames Objekt.
