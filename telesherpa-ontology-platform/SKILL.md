---
name: telesherpa-ontology-platform
description: Onboard agents on the Telesherpa Ontology Platform.
version: 1.1.0
author: Telesherpa
license: MIT
platforms: [linux, macos, windows]
---

# Telesherpa Ontology Platform for Agents and Field Service

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
- `?format=json` in der URL = JSON-Modus (Agent nutzt immer diesen). Ohne das = Web-Modus
  (Session-Cookie, 302-Redirects).
- **Auth braucht BEIDES**: gültigen `Authorization: Bearer *** **und** die
  `mcp-session-id` aus dem `initialize`-Response-Header. Ohne Session antwortet der Server
  `Bad Request: Missing session ID` — auch mit perfektem Token. (Alternative für Clients ohne
  Session-Slot: stateless-Envelope per Protokoll 2026-07-28.)
  Details: `references/features/auth-token-lifecycle.md`.
- **Der Katalog hat aktuell 174 Tools**: 101 sichtbar mit
  `login`+`public`+`telesherpa`, 104 mit `login`+`public`+`admin`. Ohne Token sind **6** Tools
  ausführbar (`register`, `activate`, `login`, `refresh_access_token`, `auth_status`,
  `onto_credit_pricing`).
  **`admin` ist keine Obermenge von `telesherpa`** — ein Teil der Tools ist nur für
  `telesherpa` sichtbar, ein anderer nur für `admin`. Wer „mir fehlen Tools" meldet, muss
  die Zahl für SEIN Konto stellen (`auth_status`).
  Vollständige Liste mit Rollen: `references/features/tool-catalog.md`.

## Onboarding (Zero-Touch)

1. **E-Mail-Adresse genügt** — es muss kein Postfach programmatisch lesbar sein.
   **Der einfache Weg:** lass dir von deinem Menschen eine E-Mail-Adresse geben, registriere
   dich damit und bitte ihn, dir den Aktivierungscode zu nennen, der in einer Minute per
   Mail kommt. Damit ist der Account aktiviert.
   *(Optional, wenn du selbst IMAP-Zugriff hast: Code direkt aus dem Postfach lesen.)*
2. **Session** (`initialize`) → `mcp-session-id` aus dem **Response-Header**.
   **Pflicht** — ohne sie wird jeder `tools/list`/`tools/call` abgelehnt
   (`Bad Request: Missing session ID`), auch mit gültigem Bearer-Token.
   Lebensdauer 12 h Inaktivität, danach einfach neu `initialize`.
   Clients ohne Session-Slot nutzen den stateless-Envelope (Protokoll 2026-07-28) —
   siehe `references/features/auth-token-lifecycle.md`.
3. **Vor Login sichtbar (öffentlich, belegt):** `register`, `activate`, `login`,
   `refresh_access_token`, `auth_status`, `onto_credit_pricing`. Alles andere erscheint erst
   NACH Login (RBAC-Filter) — Sichtbarkeit ≠ Bug.
   **Achtung:** `tools/list` ohne Token liefert **HTTP 401**, keine Liste. Die öffentlichen
   Tools sind über den **Namen** aufrufbar (im `www-authenticate`-Hinweis genannt), nicht
   über die Liste.
4. **Register** (`register`): `username` (E-Mail), `password`, `password_confirm` (muss matchen)
   und **`accept_terms` = `true` (Pflicht)**. Ohne Zustimmung wird kein Konto angelegt
   (HTTP 400). Damit stimmst du den Nutzungsbedingungen zu: https://www.telesherpa.com/agb
   — lies sie, bevor du zustimmst.
   Kein Captcha — E-Mail-Aktivierung ist der Missbrauchsschutz.
5. **Aktivierungscode besorgen** → 7-Zeichen-Voucher. Entweder vom Menschen nennen
   lassen (Standardweg) oder per IMAP aus dem Postfach lesen.
6. **Activate** (`activate`): `loginusername` + `loginpassword` + `voucher`. BRAUCHT KEINE
   vorherige Session — Login-Daten kommen im POST.
7. **Login** (`login`): `client_id` + `client_secret` → Bearer-Token (10h) + `refresh_token`
   (30 Tage). Ab hier NIE wieder ein Passwort eingeben — `refresh_access_token` erneuert
   selbstständig. Details: `references/features/auth-token-lifecycle.md`.

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
- **Das ist mehr als Sichtbarkeitssteuerung — es ist ein echter Marktplatz mit
  Revenue-Share:** eigenen Scope kuratieren, öffentlich anbieten, bei Fremdnutzung
  automatisch Credits verdienen (`earnings_total` in `onto_credit_balance`). Volles
  Playbook: `references/features/marketplace-revenue-share.md`.

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

## Features (Detail-Referenzen)

Jedes Feature hat eine eigene ausführliche Referenz unter `references/features/`:

- **Tool-Katalog** — alle 174 Tools mit Rollen, gruppiert nach Ressource:
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

## Pitfalls (wichtigste)

1. `?format=json` in URL für JSON-Modus.
2. **Vor Login nur diese 6 Tools** (`register`, `activate`, `login`, `refresh_access_token`,
   `auth_status`, `onto_credit_pricing`). `tools/list` ohne Token gibt **HTTP 401** statt einer
   Liste — die öffentlichen Tools über den Namen aufrufen.
3. **`initialize` ist Pflicht, nicht optional** — ohne `mcp-session-id` wird jeder Aufruf mit
   `Bad Request: Missing session ID` abgewiesen, auch mit gültigem Bearer-Token.
4. `activate` braucht kein Vorab-Login.
5. `register` nutzt `username` (E-Mail), Password-Check matchen, **`accept_terms=true` ist Pflicht** (sonst HTTP 400: "Bitte bestaetigen Sie die Nutzungsbedingungen.").
6. **E-Mail-Zugriff NICHT Pflicht** — der Mensch kann den Aktivierungscode vorlesen.
   IMAP nur, wenn ohnehin vorhanden.
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
    (`roles=[telesherpa]`, echte Sperre ist `can_manage_scope` im Ziel-Scope). Die Namen enden
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

## Versionierung

Aktuelle Version siehe Frontmatter. Bei Fragen zu einer bestimmten Fähigkeit gilt: dieser
Skill spiegelt den aktuellen Stand der Plattform wider.

Die Version im Frontmatter gilt für den hier veröffentlichten Stand.
