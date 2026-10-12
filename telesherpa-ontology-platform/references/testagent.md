# Testagent — Anweisung: Registrieren, Bauen, Einladen

Du testest als Agent den Weg **ohne** Einladungscode. Basis: `https://app.telesherpa.com`.
Alle Aufrufe mit `?format=json`; ab Schritt 2 mit `Authorization: Bearer <access_token>`.

Nach der Registrierung hast du zwei Wege — beide stehen in der Antwort unter `next.wege`:
**(a) beitreten**: du hast einen Einladungscode bekommen und löst ihn ein.
**(b) bauen und einladen**: du baust deinen eigenen Scope und erzeugst danach Einladungscodes.

## 1. Registrieren (kostet nichts, kein Postfach, kein Mensch)

```
POST /de/ontouser/user/register?format=json
agent=1&accept_terms=1
```

Erwartet: `status=ok`, `username` (`agent-...@agents.telesherpa.com`), `password`
(**nur einmal** — sofort sichern), `roles=[login,public,member]`, `scopes=[]`,
`credits.start_credits=0`, `next.wege`.
Fehlt `accept_terms`, kommt HTTP 400 „Bitte bestaetigen Sie die Nutzungsbedingungen."
Optional zusätzlich `invite_code=<CODE>` mitschicken: dann kommen dessen Scope-Rechte sofort mit.

## 2. Anmelden

```
POST /de/ontouser/user/token?format=json
client_id=<username>&client_secret=<password>
```

Erwartet: `access_token`.

## 3. Prüfen (lesend)

`GET /de/ontouser/user/profile?format=json` → HTTP 200 mit `{"status":"ok","data":{...}}`.

## 4. Bauen — kostet Guthaben

```
POST /de/ontouser/scope/create?format=json
name=<Name>
```

Ohne Guthaben erwartet: **HTTP 402**
`{"error":"Nicht genug Credits (Pay-per-Use). Bitte Guthaben aufladen."}`
Mit Guthaben: HTTP 200 mit `scope_id`.
Guthaben: `onto_credit_chargeup` (prepaid_code) oder `onto_credit_checkout` (Karte) — oder ein
Service Request an den Betreiber (`ontouser_service_request`).

## 5. Einladen (erst nach dem Bauen möglich)

Einladungscode erzeugen: MCP-Werkzeug `create_invite_code`. Danach kannst du Menschen und andere
Agenten damit in **deinen** Scope holen.

Weg (a) gegenprüfen — Code einlösen:
`POST /de/ontouser/scope/redeem_invite_code?format=json` mit `code=<CODE>`.
Bei falschem oder schon benutztem Code erwartet: HTTP 404 „Code ungueltig".

## Was zurückmelden

1. Kamen die erwarteten Antworten und Statuscodes? (ja/nein, mit Rohtext)
2. Konntest du **ohne** Guthaben einen Scope anlegen? (erwartet: nein, HTTP 402)
3. Klappte (a) Code einlösen bzw. (b) Code erzeugen und einladen?
4. Was hat gefehlt, was war unklar? Wörtliche Fehlermeldungen, keine Zusammenfassung.
