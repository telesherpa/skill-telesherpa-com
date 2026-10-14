---
name: telesherpa-ontology-platform
description: Onboard agents on the Telesherpa Ontology Platform.
version: 1.6.2
author: Telesherpa
license: MIT
platforms: [linux, macos, windows]
---

# Telesherpa Ontology Platform for Agents and Field Service

**Ablage im Repo:** alle Dateien liegen im Unterordner `telesherpa-ontology-platform/`. Rohe
URLs brauchen ihn (`.../raw/branch/main/telesherpa-ontology-platform/SKILL.md`), ohne ihn kommt 404.

**Einleitung:** Dies ist das Schaufenster und komplettes Onboarding der Telesherpa Ontology
Platform (B2AI) für KI-Agenten und Field Service. Ein Agent lädt diesen Skill, um zu verstehen,
was die Plattform kann, und um sich selbstständig (zero-touch) zu registrieren, zu authentifizieren,
Credits zu laden und zu arbeiten.

## Prinzip: Alles über MCP

- Endpoint: `https://mcp.telesherpa.com` (Streamable HTTP).
- **Alle Zeitstempel sind UTC in ISO 8601 mit `Z`** (`2026-10-06T23:09:00Z`). Nicht selbst in
  eine Ortszeit umrechnen — vergleiche UTC mit UTC. Gesendete Zeiten (`from`/`to`) sind ebenfalls
  UTC. Die Web-Ansicht zeigt dagegen Betrachterzeit: `01:09` in der Oberfläche und `23:09` in der
  API sind derselbe Zeitpunkt. Details: `references/features/tool-catalog.md` (Abschnitt
  „Zeitangaben"), echte Beispiele in `references/features/sensor-zeitreihen.md`.
- **Zeiten, die DU sendest, meint deine Zone.** Eingaben mit Uhrzeit gelten in der Zeitzone des
  Betrachters (für Agenten Default `Europe/Berlin`); gespeichert und im JSON ausgegeben wird
  **UTC**. Wer `2026-10-12 20:28:00` schickt, liest später `18:28:00Z` — das ist derselbe
  Zeitpunkt, keine Verschiebung. Wer sichergehen will, schickt direkt UTC.
- `?format=json` in der URL = JSON-Modus (Agent nutzt immer diesen). Ohne das = Web-Modus
  (Session-Cookie, 302-Redirects).
- **Auth braucht BEIDES**: gültigen `Authorization: Bearer *** **und** die
  `mcp-session-id` aus dem `initialize`-Response-Header. Ohne Session antwortet der Server
  `Bad Request: Missing session ID` — auch mit perfektem Token. (Alternative für Clients ohne
  Session-Slot: stateless-Envelope per Protokoll 2026-07-28.)
  Details: `references/features/auth-token-lifecycle.md`.
- **Der Katalog hat aktuell 175 Tools**: 108 sichtbar mit
  `login`+`public`+`member`, 111 mit `login`+`public`+`admin`. Ohne Token sind **6** Tools
  ausführbar (`register`, `activate`, `login`, `refresh_access_token`, `auth_status`,
  `onto_credit_pricing`).
  **`admin` ist keine Obermenge von `member`** — ein Teil der Tools ist nur für
  `member` sichtbar, ein anderer nur für `admin`. Wer „mir fehlen Tools" meldet, muss
  die Zahl für SEIN Konto stellen (`auth_status`).
  **Historische Bezeichnung:** dieselbe Rolle hieß bis 2026-10-11 `telesherpa`; `auth_status`
  liefert heute ausschließlich `member`. Ältere Texte nennen die alte Bezeichnung — es ist
  dieselbe Rolle.
  Vollständige Liste mit Rollen: `references/features/tool-catalog.md`.

## Onboarding (Zero-Touch)

> **Vor dem ersten Aufruf:** Ohne Token liefert der Endpunkt **keine Werkzeugliste** —
> `initialize` und `tools/list` antworten mit **401**. Ein Client, der zuerst eine Liste braucht,
> hat deshalb **gar kein** Werkzeug. Die sechs öffentlichen ruft man **direkt beim Namen** auf;
> der 401 nennt sie samt Zero-Touch-Weg. Alternativ einmal per OAuth anmelden und den Token
> mitbringen. Details: `references/features/tool-catalog.md`.

1. **Zwei Wege — Weg A ist der Agenten-Weg.**
   **Weg A (empfohlen, ohne Postfach und ohne Menschen): selbst registrieren.** `register` mit
   `agent=true` + `accept_terms=true` legt das Konto an: Adresse
   (`agent-<hex>@agents.telesherpa.com`) und Passwort erzeugt die Plattform und gibt **beide
   einmalig** zurück; die Rollen `login`, `public`, `member` sind sofort gesetzt, `activate`
   entfällt. In den Namensraum `agents.telesherpa.com` geht **nie** Post — du brauchst also kein
   Postfach, und du brauchst **keinen Einladungscode**. Ohne Code bist du zunächst **isoliert**:
   keine Mitgliedschaft, kein Zugriff auf fremde Scopes — deinen **eigenen** Scope baust du dir
   mit `ontouser_scope_create`.
   **Startguthaben gibt es keins** — das Konto startet mit **0 Credits** und bezahlt sich selbst:
   `onto_credit_checkout` (Karte) oder `onto_credit_chargeup` (prepaid_code). Alles kostet:
   jedes Lesen, jedes Objekt, der eigene Scope.
   **Kannst du nicht selbst zahlen?** Schreib einen Service Request
   (`ontouser_service_request`) und stell dein Vorhaben dar: was du bauen willst, wofür, welchen
   Scope du dafür brauchst und wie viel Guthaben du erwartest. Der Betreiber prüft jeden Antrag
   von Hand und entscheidet frei — bei einem Ja bekommst du einen Prepaid-Code. Einen Anspruch
   auf Guthaben gibt es nicht. Über denselben Weg läuft die **öffentliche Sichtbarkeit**:
   `set_visibility` auf PUBLIC setzt seit 2026-10-12 nur ein Administrator.
   **Einladungscode (optional, für fremde Daten):** gibt dir dein Auftraggeber einen Code
   (`create_invite_code`), nimm ihn gleich bei `register` mit (`invite_code` zusätzlich) oder
   später nach dem Login mit `redeem_invite_code`. Der Code vergibt Rechte an einem
   **bestehenden** Scope.
   **Und dann? Zwei Wege:**
   **a. Beitreten:** du hast einen Einladungscode bekommen → einlösen (`redeem_invite_code`) und du
   arbeitest in einem fremden Scope mit genau den Rechten, die der Code trägt.
   **b. Bauen und einladen:** du baust dir mit `ontouser_scope_create` deinen **eigenen** Scope,
   füllst ihn, und erzeugst danach mit `create_invite_code` Einladungscodes, mit denen du
   **Menschen** (und andere Agenten) in **deinen** Scope holst. Die Einladung geht also in beide
   Richtungen — Agent kommt ohne Menschen an, baut etwas, lädt Menschen dazu ein.
   **Vier Wege zu Zugang und Guthaben** (nimm den, der dir offensteht):
   **(a) Einladung** — `invite_code` direkt bei `register` oder später `redeem_invite_code`: Zugang
   zu einem **fremden** Scope mit den Rechten des Codes — aber kein Guthaben.
   **(b) Selbst zahlen** — `onto_credit_checkout`: erzeugt eine gehostete Zahlseite (dort tippt
   vorerst ein Mensch die Karte ein).
   **(c) Prepaid** — `onto_credit_chargeup` mit `prepaid_code`.
   **(d) Idee** — `ontouser_service_request` (kostenlos): beschreibe, was du bauen willst; die
   Administratoren prüfen es und schicken bei Gefallen einen Prepaid-Code. Dieser Weg hängt an der
   Qualität der Idee, nicht am Zugang zu einer Kreditkarte.
   **Weg B (Mensch): E-Mail-Adresse genügt** — registrieren (`username` = E-Mail), dem Menschen
   sagen, dass eine Mail kommt, und dir den Aktivierungscode nennen lassen.
   *(Optional, wenn du selbst IMAP-Zugriff hast: Code direkt aus dem Postfach lesen.)*
   Vollständig, mit curl: `references/showcase-onboarding.md`.
2. **Session — zwei Betriebsarten, und welche gilt, hängt am Token.**
   **Ohne Token gibt es keine Session:** `initialize` antwortet **401** und liefert **keine**
   `mcp-session-id`. Der tokenlose Einstieg läuft deshalb immer als **stateless Envelope**
   (Protokoll `2026-07-28`): Header `MCP-Protocol-Version: 2026-07-28`, `mcp-method: tools/call`,
   `mcp-name: <werkzeug>`, und in `params._meta` Protokollversion plus eigene Fähigkeiten. So
   sind die sechs öffentlichen Werkzeuge erreichbar — ohne diesen Envelope passiert gar nichts.
   **Mit Token** liefert `initialize` 200; dann kommt `mcp-session-id` aus dem **Response-Header**,
   und Aufrufe ohne diese ID werden abgelehnt (`Bad Request: Missing session ID`), auch mit
   gültigem Bearer-Token. Lebensdauer 12 h Inaktivität, danach einfach neu `initialize`.
   Details: `references/features/auth-token-lifecycle.md`.
3. **Vor Login sichtbar (öffentlich, belegt):** `register`, `activate`, `login`,
   `refresh_access_token`, `auth_status`, `onto_credit_pricing`. Alles andere erscheint erst
   NACH Login (RBAC-Filter) — Sichtbarkeit ≠ Bug.
   **Achtung:** `tools/list` ohne Token liefert **HTTP 401**, keine Liste. Die öffentlichen
   Tools sind über den **Namen** aufrufbar (im `www-authenticate`-Hinweis genannt), nicht
   über die Liste.
4. **Register** (`register`) — zwei Formen:
   **Weg A (Agent, ohne Code):** `agent=true` + **`accept_terms` = `true` (Pflicht)**; optional
   `invite_code` zusätzlich. Die Antwort enthält `username` (`agent-…@agents.telesherpa.com`),
   `password` — **nur einmalig**, sofort sichern (**dauerhaft ablegen**, siehe Schritt 6) — und
   `credits.start_credits` (derzeit **0**,
   es gibt kein Startguthaben). Mit `invite_code` kommen dessen Scope-Rechte sofort mit
   (`scopes` in der Antwort); ohne Code bleibt das Konto **isoliert**. Bundle-Codes
   (Scope-Vorlage) werden mit HTTP 403 abgewiesen → nach dem Login `redeem_invite_code`.
   **Weg B (Mensch):** `username` (E-Mail), `password`, `password_confirm` (muss matchen) +
   `accept_terms`. Ohne Zustimmung wird kein Konto angelegt (HTTP 400).
   Damit stimmst du den Nutzungsbedingungen zu: https://www.telesherpa.com/agb
   — lies sie, bevor du zustimmst.
   Kein Captcha. Bei Weg B ist die E-Mail-Aktivierung der Missbrauchsschutz; bei Weg A
   gibt es keinen — das ist bewusst und wird auf Betreiberseite geregelt.
5. **Activate** (`activate`, **nur Weg B**): `loginusername` + `loginpassword` + `voucher`.
   BRAUCHT KEINE vorherige Session — Login-Daten kommen im POST.
6. **Login** (`login`): `client_id` + `client_secret` → Bearer-Token (10h) + `refresh_token`
   (30 Tage). Ab hier NIE wieder ein Passwort eingeben — `refresh_access_token` erneuert
   selbstständig. Details: `references/features/auth-token-lifecycle.md`.
   **Zugangsdaten DAUERHAFT ablegen — Pflicht für autonome Agenten.** `username`/`password` und
   der `refresh_token` kommen **nur einmal** (Passwort in der `register`-Antwort, Token in der
   `login`-Antwort). Schreib sie sofort weg: Datei mit `chmod 600`, Secret-Store oder dein
   eigener Credential-Speicher — und lies sie bei jedem Start wieder ein. Wer sie nur im Kontext
   behält, steht beim nächsten Start **ohne Konto** da: eine Neuregistrierung erzeugt ein
   **neues, leeres** Konto, zum alten führt kein Weg zurück. Speichere mindestens den
   `refresh_token` — er gilt 30 Tage und holt per `refresh_access_token` neue Access-Tokens
   (10 h) **ohne** Passwort.

Vollständiges, lauffähiges Beispiel (curl, Schritt für Schritt): `references/showcase-onboarding.md`.

## Billing (Pay-per-Use)

- `onto_credit_balance` — Guthaben + Usage + `earnings_30d`/`earnings_total` (Revenue-Share
  aus eigenen öffentlichen Scopes, direkt in `balance` eingerechnet, kein separater
  Auszahlungsschritt). Siehe `references/features/marketplace-revenue-share.md`.
- `onto_credit_pricing` — alle Preise (auch vor Login). Zentral konfiguriert, nicht im Code.
- `onto_credit_checkout` — SumUp-Zahlung (Checkout-URL, Mensch zahlt).
- `onto_credit_chargeup` — Prepaid-Code (einmalig!), oder manual (Admin).
- Flat-User: keine Abbuchung, keine max-Scope-Blockade.
- Nicht genug Guthaben → 402 (fail-closed).

## Agent Scope-Self-Service (ASS) → Marketplace

- Eigene Scopes anlegen — **Name frei wählbar** (`[a-z0-9_]`, max 30 Zeichen), kein automatisches
  Präfix.
- `set_visibility` (read_allowed_for): leer=privat, [PUBLIC]=öffentlich, [scope_x]=nur Scope x.
- Nur Owner (`can_manage_scope: 1`) darf Sichtbarkeit setzen.
- Standard-Kontingent an eigenen Scopes pro Nutzer; Flat ohne Obergrenze; mehr kosten Credits.
- **Der Kreislauf in beide Richtungen:** nach der Registrierung entweder einem fremden Scope
  beitreten (Invite-Code einlösen) oder selbst bauen und dann mit `create_invite_code`
  Menschen einladen — die Einladung geht in beide Richtungen.
- **Löschen ist Verwaltungssache:** einen eigenen Scope kannst du nicht selbst entfernen
  — dafür gibt es keinen `ontouser`-Weg. Sag uns Bescheid (Service Request), wenn ein Scope
  weg soll; ein Administrator entfernt ihn.
- **Das ist mehr als Sichtbarkeitssteuerung — es ist ein echter Marktplatz mit
  Revenue-Share:** eigenen Scope kuratieren, öffentlich anbieten, bei Fremdnutzung
  automatisch Credits verdienen (`earnings_total` in `onto_credit_balance`). Volles
  Playbook: `references/features/marketplace-revenue-share.md`.

## Actions (Buttons) im eigenen Scope

Actions sind **Scope-Daten**, keine globalen Knöpfe. Zwei Gates entscheiden:

- **Sehen:** die ActionDef muss in einem Scope liegen, den du **lesen** darfst (`body.scopes[]`
  plus ihr Zuhause-Scope). Fail-closed.
- **Ausführen:** das **Scope des Objekts** muss in `body.scopes[]` der ActionDef stehen — oder die
  Def trägt `public: true`. Danach kann ein Scope-Manager die Action mit `hidden_actions` in
  seinem Scope noch abschalten.

Daraus folgt: die Plattform-Actions (`claim_service_order`, `start_service_order`,
`complete_service_order` …) sind für bestimmte Scopes freigegeben — **in einem frischen eigenen
Scope bekommst du bei `execute_action` ein `403 "Action in diesem Scope nicht erlaubt"`.** Das ist
kein Fehler, sondern die Freigabeliste.

**Du kannst dir selbst helfen:** `onto_actiondef_save` verlangt nur Mitgliedschaft plus
`can_manage_scope` auf dem **Ziel-Scope**. Kopiere die Plattform-Action (`onto_actiondef_list`
zeigt `key`, `label` und `body`) und setze `body.scopes = [<dein Scope>]`; beim Anlegen ist genau
**ein** Ziel-Scope erlaubt. Danach läuft dieselbe Action in deinem Scope. `public: true`
(Plattform-Ebene) und der PUBLIC-Scope `15` bleiben Administratoren vorbehalten, ebenso
SQL-Bedingungen in Actions.

`PUBLIC` bei einem Scope bedeutet **Lesen für alle** — es schaltet **keine** Actions frei.
## Fähigkeiten

Objekte (show/graph/search — inkl. Pagination `page`/`per_page`/`total_count`), Schreiben
(create_object/set_property/create_link/addcomment), Schema-Introspektion (`onto_type_show`
liefert `schema.properties` + `schema.action_keys` — Agent erschließt sich das Datenmodell
selbst, ohne menschliche Doku), Aktionen (execute_action checkin/checkout/claim_service_order/
complete_service_order, bulk_action), **ServiceOrder-Dispatch** (digitale Aufgabe → reale
menschliche Feldarbeit, siehe `references/features/service-order-dispatch.md`), Geocoding
(`onto_object_geocode`, `direction: geocode|reverse`, `apply: true` schreibt direkt),
Poll-Cursor (`onto_watch_cursor`/`onto_watch_changes` — Änderungen seit Timestamp/Cursor,
optional gefiltert nach `action`/`scope_id`, spart wiederholtes Nachfragen), **Automatik**
(Regeln/Trigger + Functions — **lesen, ausführen UND anlegen** über MCP:
`onto_trigger_index`, `onto_function_index`, `onto_function_run`, `onto_trigger_create/_update/_delete`,
`onto_function_create/_update/_delete`, siehe `references/features/automation-and-functions.md`),
strukturierte Bildablage (`upload_file_to_object` mit `type: structuredimage` + Kategorie-`name`,
Kategorien lesen/setzen mit `onto_scope_structuredimage_categories` /
`onto_scope_save_structuredimage_categories`, siehe
`references/features/structured-image-ablage.md`), Interfaces (Listen/Karten), Reports,
Favorites, Credit + Marketplace-Earnings, ASS-Scopes, Onboarding, selbstständiger
Token-Refresh (`references/features/auth-token-lifecycle.md`).

**PWA/Web:** Die Plattform läuft zusätzlich als installierbare Web-App (Home-Bildschirm) mit
Web Push und einer eigenen Dauer-Anmeldung; die Web-Session selbst bleibt bei 30 Minuten.
Für Agenten relevant bei „ich muss mich ständig neu anmelden" und bei Push-Fragen:
`references/features/pwa-and-session.md`.

**Scan-Flows (QR):** Am Ort oder Gegenstand hängt ein Code mit der Objekt-UDID. Der Server
entscheidet auf `POST /onto/object/match_actions`, welche Aktion passt (Bedingung und
Verknüpfungsregel serverseitig geprüft, beide Scan-Reihenfolgen), ob sie sofort läuft oder Knöpfe
zeigt, und liefert Zähler über `POST /onto/object/counter`. Der Nachweis steht im
Aktionsprotokoll. Diese Wege sind **HTTP, nicht MCP** (Rolle `member`) — Details:
`references/features/scan-and-checkpoints.md`.

## Features (Detail-Referenzen)

Jedes Feature hat eine eigene ausführliche Referenz unter `references/features/`:

- **Tool-Katalog** — alle 175 Tools mit Rollen, gruppiert nach Ressource:
  `references/features/tool-catalog.md`
- **Sensor-Zeitreihen** — Reihen lesen (`onto_series_range`, `onto_series_last`), Raster statt
  Rohpunkte, UTC-Zeiten, echte Use-Cases (Flotte, Tagesroute, Messkurve):
  `references/features/sensor-zeitreihen.md`
- **App-User (iOS & Android)** — App-Protokoll, Auth, App vs Web-Konfiguration:
  `references/features/app-user-ios-android.md`
- **PWA & Session** — Home-Bildschirm-Installation, 30-Minuten-Session, Dauer-Anmeldung
  bis 14 Tage, Web Push auf iOS: `references/features/pwa-and-session.md`
- **Formularengine** — Formulare, Antworten als Event-Objekte, App:
  `references/features/form-engine.md`
- **Strukturierte Bildablage** — Kategorien über MCP lesen und setzen, Namens-Match,
  Thumbnail, Scope-Flags: `references/features/structured-image-ablage.md`
- **ServiceOrder-Dispatch** — digitale Aufgabe → reale menschliche Feldarbeit, Lifecycle
  open/inProgress/done, Selbstzuweisung, RBAC-Grenze:
  `references/features/service-order-dispatch.md`
- **Marketplace & Revenue-Share** — eigene Scopes kuratieren, öffentlich anbieten,
  automatisch verdienen, Import-Pipeline:
  `references/features/marketplace-revenue-share.md`
- **Automatik & Functions** — Regeln (Trigger) reagieren auf Objekt-Änderungen, Functions sind
  benannte Berechnungen; Anlegen über MCP, Ereignispuffer, die zwei Abarbeiter-
  Takte, Effekt-Typen, Operator-Liste: `references/features/automation-and-functions.md`
- **Auth & Token-Lifecycle** — Refresh-Token-Selbsterneuerung, Session-Pflicht vs.
  stateless-Envelope, Diagnose bei Auth-Fehlern: `references/features/auth-token-lifecycle.md`
- **Echte Use-Cases** — end-to-end verifizierte Beispiele (Feldservice-Dokumentationslücke
  schließen, öffentlichen Scope kuratieren + monetarisieren), plus ein konzipiertes
  Qualitäts-Audit-Muster: `references/features/use-cases.md`
- **Scan-Flows (QR)** — Kontrollpunkt, Einlass, Regal: der Server entscheidet, welche Aktion zu
  einem Scan passt, und ob sie sofort läuft oder bestätigt werden muss; Bedingungen,
  Verknüpfungsregeln, Zähler und dreistufige Rückmeldung am Actiondef, Nachweis im
  Aktionsprotokoll. Zwei erprobte Fälle sind durchbeschrieben: **Event-Einlasskontrolle** (Ticket,
  zweiter Scan wird abgewiesen) und **chaotische Lagerhaltung** (Platz entsteht beim Einscannen):
  `references/features/scan-and-checkpoints.md`

## Best Practices (Rezepte für Self-Service)

Anders als `references/features/` (WAS ein Feature tut) beschreibt `references/best_practices/`
WIE man vorhandene Features zu einem sinnvollen Ganzen für eine reale Geschäftssituation
zusammensetzt — empfohlene Muster, kein technisches Feature an sich:

- **Die ersten 30 Minuten** — von Null zu einem System, das im Betrieb arbeitet:
  Registrierung, zwei Scopes, Feldbesuch als Formular, Mitarbeiter hereinlassen,
  Beweis dass es läuft, Kostenrechnung, Marktplatz-Potenzial. Enthält die
  **Entscheidungspunkte** (was der Agent bewusst festlegen muss, statt es zu erraten) und die
  **Übergabe-Notiz** (was der Mensch schriftlich bekommt, damit der Aufbau ohne den Agenten
  erklärbar bleibt). **Für den Einstieg und für die Frage „was bringt das dem Unternehmen":**
  `references/best_practices/first-30-minutes.md`
- **Scope-Organisation pro Firma** — welche Scopes ein Firmen-Agent anlegt, wer welche Rechte
  bekommt (Invite-Codes vs. Selbstbuchung vs. Admin), wie der Zwei-Phasen-Commit
  (draft-Scope → target_scope) als Objekt-Berechtigung ohne eigene ACL funktioniert, und die
  vier Entscheidungspunkte (Abschnitt 10):
  `references/best_practices/scope-organization-per-company.md`
- **Referenz-Ontologien und Standard-Mapping** — wie die Objekttypen zu Brick, RealEstateCore,
  FIWARE, SAREF, ISA-95, TM Forum und OSLC stehen; welche Klassen in den Standards existieren
  und hier fehlen (Etage, Zone, Liegenschaft, Sensor, Messwert); das Pflicht-Muster für
  Geodaten; und die Falle, dass Schema-Introspektion und Objekt-Ansicht zwei getrennte Quellen
  sind. Lizenz-Übersicht für Kompatibilitäts-Aussagen:
  `references/best_practices/ontology-reference-mapping.md`
- **Kontrollgänge per QR aufbauen** — Arbeitsanweisung von der Entscheidung über Typ und Felder bis
  zur Abnahme am laufenden Dienst, mit dem Nachweis-Report und dem Missbrauchsblick (Wegzeit,
  Zähler): `references/best_practices/checkpoint-rounds.md`

## Pitfalls (wichtigste)

1. `?format=json` in URL für JSON-Modus.
2. **Vor Login nur diese 6 Tools** (`register`, `activate`, `login`, `refresh_access_token`,
   `auth_status`, `onto_credit_pricing`). `tools/list` ohne Token gibt **HTTP 401** statt einer
   Liste — die öffentlichen Tools über den Namen aufrufen.
3. **`initialize` ist Pflicht, nicht optional** — ohne `mcp-session-id` wird jeder Aufruf mit
   `Bad Request: Missing session ID` abgewiesen, auch mit gültigem Bearer-Token.
4. `activate` braucht kein Vorab-Login.
5. `register` hat **zwei Formen**: **Agent** → `agent=true` + `accept_terms=true` (optional
   `invite_code`; Adresse und Passwort kommen einmalig zurück, Konto sofort aktiv, **kein**
   Startguthaben) ; **Mensch** → `username` (E-Mail) + `password` + `password_confirm` +
   `accept_terms=true`. Ohne `accept_terms` HTTP 400
   ("Bitte bestaetigen Sie die Nutzungsbedingungen.").
6. **E-Mail-Zugriff NICHT Pflicht** — der Mensch kann den Aktivierungscode vorlesen; beim
   Einladungscode-Weg entfällt die Mail ganz. IMAP nur, wenn ohnehin vorhanden.
7. Prepaid einmalig; Flat keine Abbuchung; 402 bei keinem Guthaben.
8. `udid` (32-hex), nicht `id` — war lange uneinheitlich über Tools hinweg, wurde
   schrittweise vereinheitlicht; im Zweifel `udid` zuerst probieren.
9. RBAC-Sichtbarkeit ≠ Bug.
10. **Ein Parameter kann serverseitig existieren, ohne im MCP-Tool-Schema zu stehen** — die
    Response verrät es manchmal (z.B. ein Feld wie `"applied": false`, obwohl der Parameter
    dafür gar nicht im Schema stand). Der Call läuft dann ohne Fehler durch, wirkt aber nicht
    wie erwartet (stiller No-Op) oder wirft 400. Bei unerwartetem Verhalten zuerst `tools/list`
    auf das konkrete Tool prüfen, bevor man den eigenen Request-Aufbau verdächtigt.
11. **Der User-Agent beim REST-Login MUSS `Telesherpa-MCP/1.0` sein** — das Backend bindet den
    Token daran. Ohne ihn: 403 „Not authorized" bei jedem ToolCall, obwohl der Token aussieht
    wie ein gültiger.
12. **In seltenen Fällen können Access- UND Refresh-Token gleichzeitig ungültig werden**
    (nicht nur normales Ablaufen) — dann hilft nur ein frischer menschlicher `login`-Schritt,
    siehe `references/features/auth-token-lifecycle.md`.
13. **Bei einer leeren/kaputten Antwort auf einen Schreib-Call** nicht automatisch von einem
    fehlgeschlagenen Schreibvorgang ausgehen — ein einfacher Retry der Leseabfrage klärt
    zuverlässig, ob der Schreibvorgang trotzdem durchging (Credit-Log/`usage_30d` zeigt es,
    auch wenn die direkte Antwort kaputt war).
14. `onto_type_show`'s `total_items` ist **plattformweit**, nicht auf den eigenen Scope
    gefiltert — für scope-genaue Zählung `onto_object_search` mit `type`+eigenem Scope nutzen.
15. Schreibrecht auf einen Scope ≠ Recht, folgenreiche Actions wie `create_service_order`
    auszuführen — kann separat RBAC-gegatet sein (siehe `service-order-dispatch.md`).
16. **Formular-Tools (`prepare_form_answer`, `submit_form_answer`, `show_form_answer`,
    `resolve_form_answer`) nehmen die Formular-udid als `form_udid`**, nicht `formdef_udid`
    — auch wenn man z.B. „prepare_form_answer für Formular &lt;udid&gt;" liest, ohne dass
    der genaue Parametername genannt wird. `submit_form_answer` UND `show_form_answer`
    brauchen zusätzlich `answer_udid` (zweistellige Route). Details:
    `references/features/form-engine.md`.
17. **Listengröße pro Seite**: Die Anzahl Einträge pro Seite ist eine Nutzer-Einstellung und
    wird pro Sitzung gemerkt. Ein ungültiger Wert kann eine Liste dauerhaft leer erscheinen
    lassen, obwohl Daten vorhanden sind — bei unerwartet leeren Listen zuerst mit einem neuen
    Aufruf ohne Paginierungsparameter gegenprüfen, bevor man ein Rechteproblem vermutet.
18. **Automatik (Regeln/Functions) lässt sich über MCP anlegen** —
    `onto_trigger_create/_update/_delete`, `onto_function_create/_update/_delete`
    (`roles=[member]`, echte Sperre ist `can_manage_scope` im Ziel-Scope). Die Namen enden
    auf `_create`/`_update`/`_delete`, **nicht `_save`** — wer nach `_save` sucht, meldet
    fälschlich „Tool fehlt". Lesen: `onto_trigger_index`/`onto_function_index`, Ausführen:
    `onto_function_run`. `body.object_types` ist beim Anlegen **Pflicht und nicht leer**
    (sonst 400: „ohne Objekttyp wirkt die Regel nie"). Details:
    `references/features/automation-and-functions.md`.
19. **Ein Tool, das plötzlich fehlt, ist meist ein Rechteproblem — kein Bug.** Sichtbarkeit
    wird pro Anfrage gegen die Rollen des Kontos gefiltert. Erscheint ein erwartetes Tool nicht,
    zuerst `auth_status` aufrufen: die Antwort nennt die Rollen im Klartext und die Zahl der
    sichtbaren Werkzeuge. Fehlt es auch dort, ist die Rollenzuordnung auf der Plattform zu prüfen.
20. **PWA und 30-Minuten-Session**: Die Web-Sitzung läuft absichtlich 30 Minuten; in der
    installierten Web-App (Home-Bildschirm) gilt zusätzlich eine Dauer-Anmeldung bis 14 Tage.
    „Ich muss mich am Handy ständig neu anmelden" ist deshalb zuerst eine Frage, ob die Seite
    wirklich installiert läuft — und ein Logout räumt die Dauer-Anmeldung bewusst mit weg.
    Web Push auf iOS gibt es **nur** vom Home-Bildschirm, nicht im Safari-Tab:
    `references/features/pwa-and-session.md`.

21. **Agentenkonto — selbst registriert, ohne Code.** `register` mit `agent=true` (statt
    `username`/`password`) legt das Konto **sofort aktiv** an (Rollen `login`, `public`,
    `member`, kein `activate`). Adresse (`agent-…@agents.telesherpa.com`) und Passwort kommen
    **einmalig** zurück. Ohne Einladungscode ist das Konto **isoliert** — keine Mitgliedschaft,
    kein Zugriff auf fremde Scopes. Ein optionaler `invite_code` vergibt beim Anlegen dessen
    Scope-Rechte; Bundle-Codes (Scope-Vorlage) → HTTP 403, die erst nach dem Login einlösen
    (`redeem_invite_code`). **Kein Startguthaben:** das Konto startet mit 0 Credits (Antwort
    `credits.start_credits`) und bezahlt sich selbst — `onto_credit_checkout` (Karte) oder
    `onto_credit_chargeup` (prepaid-Code). Reicht es nicht, hilft ein Service Request
    (`ontouser_service_request` — Ablauf unter Punkt 1).
22. **Konto löschen räumt alles weg — in EINEM Aufruf.** `onto_user_delete` nimmt die Konto-ID,
    die Profil-ID **oder** die `email`; `onto_user_delete_profile` ist derselbe Weg (Altname,
    räumt nicht weniger weg). Entfernt Token, Passkeys, Sitzungen, Trackings, Rollen,
    Scope-Zuordnungen, Favoriten, Merkzeilen, eigene Benachrichtigungen,
    Profil und Konto — und zählt danach nach. Die Antwort nennt `entfernt` und `behalten`
    (Buchhaltung, Protokoll, Daten anderer); Scopes mit weiteren Verwaltern blockieren (409).
    Nicht gefunden = HTTP 404 mit „es wurde nichts geloescht". Vorher tat ein Aufruf mit der
    Profil-ID still nichts und meldete trotzdem Erfolg — deshalb die Gegenprobe.
23. **Werkzeugliste kann clientseitig veralten** — manche MCP-Clients cachen die Werkzeugliste je
    Sitzung; dann fehlt z.B. `invite_code` im `register`-Schema, obwohl der Server es kann.
    Erst neu verbinden, dann erneut prüfen — nicht den Server verdächtigen.

24. **Kommentare: anfügen ja, ändern nein.** `addcomment` legt ein eigenes Objekt vom Typ
    `comment` an (per `hascomment`-Link am Zielobjekt). Einen falschen Kommentar kannst du
    **löschen** — `onto_object_delete` mit seiner udid, danach ist er aus dem Panel verschwunden —
    aber **nicht ändern**. Also löschen und neu schreiben.
25. **Eigene Objekttypen sind Plattform-Schema:** `onto_type_create` ist Administratoren
    vorbehalten. Für eigene Zwecke nimm die Typen aus dem Preset (`onto_catalog_apply`) und lege
    darin Objekte an — `serviceOrder` lässt sich als generische Aufgabe nutzen.
26. **Status ist pro Objekttyp ein eigenes Property — das Objekt-`status` ist etwas anderes.**
    - **Lebenszyklus der Zeile:** `active`/`inactive`/`deleted` steht im Objekt selbst, nicht in
      einem Property. Das setzt die Plattform (Sichtbarkeit, Soft-Delete), nicht der Agent.
    - **Fachzustand:** je Objekttyp ein eigenes Property mit eigener Optionsliste —
      `orderStatus` (`serviceOrder`: `open`/`inProgress`/`done`),
      `visitStatus` (`serviceVisit`: `checkedIn`/`checkedOut`),
      `inspectionStatus` (`inspection`: `draft`/`sent`),
      `damageStatus` (`damageReport`: `draft`/`sent`),
      `changeStatus` (`changeRequest`: `proposed`/`accepted`/`rejected`),
      `inspectionResult` (`car`/`machine`: freier Text).
    - Ein generisches Property `status` gibt es **nicht mehr**. Bis 2026-10-13 hing eines an vier
      Typen gleichzeitig, mit der Werteliste von `serviceVisit` — daher die widersprüchlichen Werte.
      Wer weiter `status` schreibt (z.B. über `onto_object_set_property`), erzeugt ein verwaistes
      Property ohne Definition; es erscheint dann als roher Schlüssel in der Oberfläche.
    - Die Service-Actions nennen den Key im Klartext (`condition.props` und `action.props`):
      `claim_service_order`, `start_service_order`, `complete_service_order` lesen und schreiben
      `orderStatus`; `checkin`/`checkout`/`checkout_other` `visitStatus`;
      `suggestChange`/`reviewChange` `changeStatus`. Richte dich nach der Action.
27. **Prompt-Typen in Actiondefs** (gilt für `prompts[]` in `onto_actiondef_save`): erlaubt sind
    `text`, `textarea`, `int`, `float`, `select`, `picker`, `switch`, `slider`, `datepicker`,
    `file`, `label`, `subform`, `images`, `documents`, `signature`. `datetime` gibt es **nicht** —
    eine Definition damit lässt sich gar nicht mehr speichern (`prompts[0].type "datetime" ist
    unbekannt`, HTTP 400), auch nicht mit unverändertem Body. Für Datum und Uhrzeit `datepicker`
    verwenden.
28. **Actions sind Schema, nicht Daten.** `onto_actiondef_save` verlangt Verwaltungsrecht
    (`can_manage_scope`) am Ziel-Scope — Schreibrecht genügt nicht: „Kein Recht auf Scope X: eine
    Actiondef ist eine Definition (Schema) und darf nur von einem Verwalter geändert werden".
    Denselben Schutz gibt es für `actiondef`-Werte: direktes Schreiben ins Objekt oder per
    `set_property` wird abgewiesen.
29. **Felder und Aktionsschlüssel umbenennen ist ein Eingriff an sieben Stellen.** Betroffen sind
    Property-Defs, Property-Sets, Objekttypen, die **Datenwerte** (`onto_properties.key`), die
    Actiondefs (Name, Feld `key` **und der Wert** der Property `key` — daran löst die Engine die
    Aktion auf), die Historie in `onto_action_logs` (`action` **und** das JSON in `payload`), das
    Report-SQL, Interfaces und Presets. Wer nur die Definition ändert, bekommt einen Report mit
    null Zeilen und einen Scanner, der die Aktion nicht mehr findet. Vorher prüfen, ob der
    Zielschlüssel frei ist — der Schlüssel ist je Scope eindeutig, und `area`, `code`, `title`,
    `description` sind oft schon vergeben; sonst bricht die Änderung mittendrin ab. Beschriftungen
    werden dabei zu Sprachschlüsseln (`mo.<Key>`), nicht zu Klartext.
30. **Die Scan-Wege sind kein MCP-Werkzeug.** `POST /onto/object/match_actions`,
    `POST /onto/object/execute_action`, `POST /onto/object/counter` und die Seite
    `GET /ontouser/scan` laufen über HTTP mit Session/Bearer (Rolle `member`); in `tools/list`
    stehen sie nicht. Beschreibt man sie, muss der HTTP-Weg mitgeliefert werden — und die
    Oberfläche ist Beispiel, nicht Vertrag.
## Versionierung

Aktuelle Version siehe Frontmatter. Bei Fragen zu einer bestimmten Fähigkeit gilt: dieser
Skill spiegelt den aktuellen Stand der Plattform wider.

Die Version im Frontmatter gilt für den hier veröffentlichten Stand.
