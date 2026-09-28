# Die ersten 30 Minuten — von Null zu einem System, das im Betrieb arbeitet

**Für wen:** Ein Agent, der die Plattform für ein Unternehmen aufsetzt und innerhalb einer
halben Stunde ein Ergebnis vorzeigen will — nicht eine Demo, sondern etwas, das am nächsten
Morgen tatsächlich benutzt wird.

**Das Versprechen in einem Satz:** In 30 Minuten steht ein Ablauf, für den andere Firmen eine
komplette Software kaufen, monatelang einführen und dann doch wieder Excel daneben betreiben.

---

## Warum das in 30 Minuten geht

Weil du nichts installierst, nichts konfigurierst und niemanden um Freigabe bitten musst.
Jeder Schritt unten ist ein einziger Tool-Aufruf. Es gibt keine Migration, keinen Server,
kein Rollen-Konzept zum Ausdenken — die Plattform bringt die Struktur mit, du füllst sie.

Und der wichtigste Punkt: **du bezahlst nur, was tatsächlich passiert.** Kein Setup-Preis,
keine Monatsgebühr für ungenutzte Lizenzen. Wenn nach zwei Wochen niemand damit arbeitet,
kostet es auch fast nichts. Das ist der Grund, warum man es einfach ausprobieren kann.

---

## Minute 0–5 · Registrieren und aktivieren

```
register   username=E-Mail des Kunden, password, password_confirm,
           accept_terms=true   <- PFLICHT (AGB-Zustimmung, sonst HTTP 400)
           AGB: https://www.telesherpa.com/agb
```

Dann kommt der Aktivierungscode per Mail. **Kein IMAP-Zugriff nötig:**

> **An den Menschen weitergeben:** „Ich habe uns einen Zugang angelegt. In einer Minute kommt
> eine E-Mail mit einem Aktivierungscode. Gib mir den Code, dann schalte ich den Zugang frei."

Der Mensch liest den Code aus seinem Postfach vor — fertig. Danach:

```
activate            loginusername, loginpassword, voucher
login               → Bearer-Token + refresh_token
```

Ab hier nie wieder ein Passwort. `refresh_access_token` hält die Verbindung selbstständig
aufrecht — auch über Wochen.

**Was in diesen 5 Minuten entsteht:** ein eigener, abgeschlossener Datenraum für dieses
Unternehmen. Nicht „ein Account auf einer Plattform, wo alle drin sind", sondern ein
eigener Bereich, den andere nicht sehen.

---

## Minute 5–10 · Die zwei Scopes, die alles tragen

```
ontouser_scope_create   name="<firma>_arbeit"    → hier wird gearbeitet
ontouser_scope_create   name="<firma>_archiv"    → hier liegen die Ergebnisse
```

Nur zwei Stück — mehr braucht es am ersten Tag nicht. Der eine ist der Werkraum, der andere
das Archiv. Die Trennung ist der ganze Trick: der Mitarbeiter schreibt im Werkraum und kann
im Archiv nichts kaputt machen.

Später, wenn mehrere Teams dazukommen, wächst das nach Bedarf (mehr dazu in
`references/best_practices/scope-organization-per-company.md`).

**Gleich mitnehmen (2 Aufrufe): die Bildmotive festlegen.** Die strukturierte Bildablage sagt
dem Mitarbeiter in der App, WAS er fotografieren soll:

```
onto_scope_structuredimage_categories       scope_id=<arbeit>   → was gibt es schon?
onto_scope_save_structuredimage_categories  scope_id=<arbeit>, categories=[...]
```

Vier bis sechs Motive reichen (`01_<Motiv>.jpg`, `02_<Motiv>.jpg`, `03_<Motiv>.jpg` …),
je einmal `de` und `en` als Sprachvariante desselben `imagename`, genau eines mit
`thumbnail: true`. **Der Dateiname ist später der Schlüssel** — passt er nicht exakt zum
`imagename`, erscheint nur der Platzhalter. Beispiel:
`references/features/structured-image-ablage.md`.

**Was in diesen 5 Minuten entsteht:** die Rechte-Struktur. Ab jetzt ist „darf schreiben" und
„darf nur lesen" keine Frage von Vertrauen mehr, sondern von Technik.

---

## Minute 10–18 · Der Feldbesuch als Formular

Jetzt der Schritt, der aus einer Datenbank ein Arbeitswerkzeug macht: eine Aktion, die ein
Mitarbeiter vor Ort ausführt.

```
onto_type_show         → welches Formular, welche Felder gibt es?
prepare_form_answer    form_udid=<Formular>, scope_id=<arbeit>
submit_form_answer     form_udid, answer_udid, Feldwerte
```

Beim Absenden passiert der entscheidende Wechsel: **die Antwort wandert automatisch vom
Arbeits-Scope in den Archiv-Scope.** Der Mitarbeiter hat sie erstellt — und kann sie danach
nicht mehr ändern, weil er im Archiv kein Schreibrecht hat. Ohne eine einzige Objekt-Regel,
allein über die Scope-Grenze.

**Was in diesen 8 Minuten entsteht:** der Zettel ist weg. Die Unterschrift, das Foto, die
abgehakten Positionen liegen sofort dort, wo die Disposition sie sieht — nicht im Auto, nicht
im Postfach, nicht „bring ich morgen mit".

---

## Minute 18–25 · Die Mitarbeiter hereinlassen

```
create_invite_code   scope_id, can_read, can_write, allowlist, max_uses, expires_at
```

Für den Außendienst: **`can_write` im Arbeits-Scope, `can_read` im Archiv.** Genau das ist
die Rolle, die er braucht — nicht mehr und nicht weniger.

**Der praktische Trick:** Allowlist als Domain eintragen („alle mit @firma.de") — dann muss
niemand einzelne Adressen tippen. Bei wechselnden Subunternehmern geht auch der
XLSX-Bulk-Upload über `upload_code_allowlist`.

Der Code ist zum Weitergeben gedacht (`XXXX-XXXX`, ohne verwechselbare Zeichen) — **er ist
kein Geheimnis.** Der Mitarbeiter löst ihn ein und hat seine Rechte. Fertig.

**Was in diesen 7 Minuten entsteht:** Einarbeitung, die keine Einarbeitung ist. Kein
„wer kriegt welchen Zugang", kein Passwort-Zettel, kein Anruf beim Chef.

---

## Minute 25–30 · Beweisen, dass es läuft

```
onto_object_create     → einen Testbesuch anlegen
onto_object_execute_action  → Aktion ausführen
onto_object_search     → prüfen: liegt es im Archiv?
onto_credit_balance    → was hat es gekostet?
```

Dann der Moment, der überzeugt: **schick dem Menschen einen Auftrag, den er auf dem Handy
sieht.** Über `create_service_order` wird daraus eine echte Aufgabe für einen echten Menschen —
mit Lifecycle offen → angenommen → erledigt. Wenn er sie abschließt, steht das Ergebnis in der
Plattform.

Das ist keine Simulation. Das ist ein digitaler Auftrag, der einen Menschen erreicht hat.

### Und die Rückfrage automatisch beantworten (Regel)

Der letzte Schritt, der Betrieb spart: eine **Regel** (Trigger) auf den kritischen Befund.
Sie braucht keinen Menschen und kostet keine Credits:

```
onto_function_index  → gibt es die Rechnung schon?
onto_trigger_create  name="eskalation_kritisch", scope_id=<arbeit>, body={...}
```

```json
{"on":"insert","object_types":["inspection"],
 "condition":{"props":[{"key":"zustand","op":"eq","value":"kritisch"}]},
 "effects":[{"type":"create_object","object_type":"changeRequest",
             "name":"Eskalation: {object_name}",
             "props":{"status":"proposed","text":"Automatisch aus dem Standort-Check erzeugt."},
             "links":[{"linktype":"proposesChange","to":"{object_udid}"}]}]}
```

Ab jetzt wird aus einem kritischen Standort-Check **automatisch** ein Vorgang — sichtbar für
das Backoffice, ohne Telefonat. **`condition.changed` dabei nicht vergessen**, sonst feuert die
Regel pro geschriebenem Feld. Details: `references/features/automation-and-functions.md`.

### Den Menschen am Handy abholen (PWA)

Die Plattform lässt sich auf dem Home-Bildschirm installieren. Zwei Dinge, die der Mensch
sofort merkt:

- **Er bleibt länger angemeldet.** Die Web-Session läuft 30 Minuten; in der installierten
  Web-App kommt eine Dauer-Anmeldung bis 14 Tage dazu — er muss also nicht bei jedem Zugriff
  Passwort und 2FA tippen.
- **Push-Nachrichten gibt es auf iOS nur von dort.** Im Safari-Tab zeigt der Schalter nur einen
  Hinweis. Installiert = benachrichtigbar.

Details: `references/features/pwa-and-session.md`.

---

## Was danach kommt — wenn es trägt

### Der Marktplatz: aus Daten wird ein Geschäft

Das Unternehmen muss Daten nicht nur benutzen, es kann sie **anbieten**. Ein kuratierter
Bestand — Kundenstandorte, Anlagen, Regionen — wird öffentlich gestellt:

```
ontouser_scope_set_visibility   read_allowed_for=["PUBLIC"]
```

Andere Nutzer arbeiten darin, und **jede Fremdnutzung schreibt automatisch Credits gut** —
sichtbar als `earnings_total` im Kontostand, direkt mit dem Guthaben verrechnet. Kein
separater Auszahlungsweg, keine Rechnungsstellung.

**Warum das strategisch interessant ist:** Die Firma sammelt diese Daten sowieso im Betrieb.
Bisher lagen sie in Ordnern. Jetzt sind sie ein Bestand mit Preis — und die Konkurrenz zahlt
dafür, sie nutzen zu dürfen.

### Feldarbeit auslagern

Über ServiceOrders lassen sich Aufgaben an externe Felddienstleister ausgeben
(`complete_service_order` = 50 Credits). Was man selbst nicht hinfahren will, wird zu einem
Auftrag, den jemand anders erledigt — mit Foto-Beweis im eigenen Archiv.

---

## Was kostet das? (ehrlich gerechnet)

Die Preise stehen öffentlich in `onto_credit_pricing`, abrufbar ohne Login:

| Vorgang | Credits | EUR |
|---|---|---|
| Lesen (`graph`) | 1 | 0,01 € |
| Objekt anlegen (`create_object`) | 10 | 0,10 € |
| Aktion ausführen (`execute_action`) | 10 | 0,10 € |
| Auftrag abschließen (`complete_service_order`) | 50 | 0,50 € |

**Beispiel — eine Firma mit 3 Außendienstlern, je 10 Besuche im Monat:**

Ein Besuch = Aktion ausführen + drei Objekte (Standort, Foto, Materialposten)
= 10 + 3×10 = **40 Credits = 0,40 €**
→ 3 Mitarbeiter × 10 Besuche × 0,40 € = **12 € im Monat.**

**Was dafür wegfällt:** Telefonate für Rückfragen, Nachsortierung von Zetteln, das Suchen von
Fotos in WhatsApp-Verläufen, Nachtragen von Tagesberichten. Wenn das eine Stunde Büroarbeit
pro Tag spart, ist die Rechnung nicht knapp — sie ist absurd einseitig.

Dazu kommen einmalig Scope-Kosten, wenn mehr als das Standard-Kontingent gebraucht wird
(z.B. 1 Scope = 50 Credits = 0,50 €). **Flat-Tarif verfügbar**, wenn ohne Abbuchung gerechnet
werden soll.

---

## Was ein Geschäftsführer davon hat — in seiner Sprache

**Nicht:** „eine Ontologie-Plattform mit Objekten, Properties und gescopten Links."

**Sondern:**

1. **Der Außendienst dokumentiert vollständig und sofort.** Nicht weil er braver ist, sondern
   weil es einfacher ist als der Zettel.
2. **Chef und Backoffice sehen den Stand in Echtzeit.** Wer war wo, was wurde gemacht, was
   fehlt noch.
3. **Rechte ohne Zusatzsoftware.** Freie Mitarbeiter bekommen Schreibrecht dort, wo sie es
   brauchen — und kein Datenzugriff darüber hinaus.
4. **Kosten, die mit der Nutzung atmen.** Im ruhigen Monat fast nichts, im vollen Monat
   proportional. Keine Lizenzrunde, die man erklären muss.
5. **Und ein Bestand, der wachsen kann.** Die eigenen Daten werden zum Potenzial für den
   Marktplatz — mit automatischer Vergütung.

**Der ehrliche Haken:** Der Nutzen entsteht erst, wenn es die Mitarbeiter tatsächlich
benutzen. Die ersten 30 Minuten bauen das System; die Überzeugungsarbeit im Betrieb kommt
danach. Genau deshalb ist es so gebaut, dass die Nutzung **einfacher** ist als der bisherige
Weg — sonst funktioniert keine Einführung, egal wie gut die Software ist.

---

## Entscheidungspunkte — was der Agent bewusst festlegen muss

Die Plattform kann dem Agenten nicht abnehmen, was eine **Geschäftsentscheidung** ist.
Alles andere erschließt er sich selbst (`onto_type_show` liefert `schema.properties` +
`schema.action_keys`). Diese vier Punkte muss er aktiv entscheiden — und die Entscheidung
festhalten:

**1. `target_scope` setzen oder weglassen.**
Setzen, wenn Ersteller und Bearbeiter der fertigen Antwort verschiedene Personen sind
(Außendienst schreibt, Backoffice wertet aus). Weglassen, wenn in einem kleinen Betrieb jeder
alles bearbeiten darf. Beides ist gültig — der Fehler ist, es **aus Versehen** zu vergessen:
ohne `target_scope` bleibt die Antwort im Schreib-Scope liegen und das Backoffice sieht sie nie.
Details: `references/best_practices/scope-organization-per-company.md`, Abschnitt 8.

**2. Anzahl der `arbeit`-Scopes.**
Die Formel ist **1× `stammdaten` + 1× `archiv` + N× `arbeit`** — ein `arbeit`-Scope pro Team,
das seine Entwürfe getrennt halten muss. Schreib-Granularität = Scope-Granularität; es gibt
keine zweite Achse. Bei einem Team bleibt es bei zwei bis drei Scopes.

**3. Marktplatz: Scope öffentlich stellen oder privat lassen.**
`ontouser_scope_set_visibility` mit `read_allowed_for=["PUBLIC"]` bedeutet **nicht nur**
Sichtbarkeit, sondern dass Fremdnutzung automatisch Credits gutschreibt
(`earnings_total` im Kontostand). Das ist eine Geschäftsentscheidung über den eigenen
Datenbestand — nicht ein Häkchen, das man beim Aufsetzen nebenbei setzt.

**4. Billing-Modell.**
`pay_per_use` (jede Aktion wird gebucht) oder `flat` (keine Abbuchung, kein Scope-Limit).
Die Wahl entscheidet, ob der Kunde die Kostenrechnung unten überhaupt zu sehen bekommt.

**Warum das wichtiger ist als es klingt:** Der Aufbau selbst geht in Minuten, und der Agent
trifft diese vier Punkte in der Regel richtig. Aber nach dem Aufbau weiß niemand mehr, *warum*
so entschieden wurde. Genau diese vier Fragen kommen drei Monate später vom Kunden — und dann
ist niemand mehr da, der sie beantworten kann.

---

## Übergabe-Notiz — was der Mensch schriftlich bekommt

Der Aufbau ist erst fertig, wenn der Mensch ihn **ohne den Agenten** erklären kann. Diese
Notiz ist das Ergebnis, nicht ein netter Zusatz — ohne sie versteht niemand die Struktur,
wenn der Agent nicht mehr im Kontext ist.

**Inhalt (kurz halten, sieben Zeilen genügen):**

1. **Was gebaut wurde** — in der Sprache des Betriebs, nicht der Plattform. Nicht „drei Scopes
   mit `target_scope`-Commit", sondern „der Außendienst schreibt seine Besuche, das Büro sieht
   sie fertig und kann sie nicht mehr verändern".
2. **Wer darf was** — welche Personengruppe in welchem Scope lesen/schreibt. Die Rechte-Matrix
   aus `scope-organization-per-company.md` als Vorlage nehmen und auf die Firma kürzen.
3. **Warum diese Entscheidungen** — die vier Punkte von oben, je ein Satz. Das ist der Teil,
   der später die Diskussion abkürzt.
4. **Erste drei Handgriffe für einen Mitarbeiter** — Home-Bildschirm, Invite-Code einlösen,
   erster Besuch. Kein Handbuch, drei Zeilen.
5. **Was der Agent als Nächstes tun würde** — offen benannte Optionen (Marktplatz, weitere
   Teams/`arbeit`-Scopes, Regel für Eskalation), damit der Mensch entscheidet statt zu raten.

**Der ehrliche Teil, der in die Notiz gehört:** Was *nicht* eingerichtet wurde und warum.
Ein bewusst weggelassenes `target_scope` ist eine gute Entscheidung — aber nur, wenn sie
dokumentiert ist. Sonst liest sie sich später wie ein Fehler.

**Prüfmaßstab für „gelungen":** Der Mensch kann einem Kollegen in zwei Minuten erklären, wer
wo arbeitet — ohne die Plattform zu öffnen.

---

## Reihenfolge zum Abhaken

1. `register` → E-Mail des Kunden, Passwort
2. Aktivierungscode vom Menschen erfragen → `activate`
3. `login` → Token sichern (danach nur noch `refresh_access_token`)
4. `ontouser_scope_create` × 2 (arbeit, archiv)
5. `onto_type_show` → verfügbares Formular prüfen
6. Testweise `prepare_form_answer` + `submit_form_answer` → liegt es im Archiv?
7. `create_invite_code` für den Außendienst (write=arbeit, read=archiv)
8. `create_service_order` → ein echter Mensch bekommt einen echten Auftrag
9. `onto_credit_balance` → Kunde zeigt Kosten
10. `onto_scope_save_structuredimage_categories` → Bildmotive für die App festlegen
11. `onto_trigger_create` → kritischer Befund eskaliert automatisch
12. Bei Bedarf: `ontouser_scope_set_visibility` → Marktplatz
13. Dem Menschen sagen: Seite auf den **Home-Bildschirm** legen (dann bleibt er angemeldet
    und bekommt Push)
