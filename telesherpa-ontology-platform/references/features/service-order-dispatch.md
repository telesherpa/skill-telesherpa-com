# ServiceOrder — Digitale Aufgabe → reale menschliche Arbeit

## Konzept

Der größte Differenzierer der Plattform gegenüber reinen Daten-Backends (Firebase, Supabase,
Snowflake): ein Agent kann nicht nur Daten schreiben, sondern **reale Feldarbeit auslösen**.
Ein `serviceOrder`-Objekt, verlinkt an einen Standort, erscheint für menschliche
Field-Service-Techniker in deren App/Web-Interface — sie nehmen ihn an, erledigen die Arbeit,
schließen ihn ab. Der Agent bekommt das Ergebnis (inkl. hochgeladener Fotos, Kommentare) zurück.

**Wichtig:** Das ist keine Simulation. Ein realer Mensch fährt tatsächlich zu einem Standort.
Ein `serviceOrder` sollte daher nie leichtfertig angelegt werden — bei Tests immer klar als
`TEST` im Namen/Task markieren, damit für den Ausführenden sofort erkennbar ist, dass es sich
nicht um einen echten Einsatz handelt.

## Lifecycle

```
open --(claim_service_order)--> inProgress --(complete_service_order)--> done
```

- **open**: Auftrag ist unzugewiesen, sichtbar im offenen Pool (Interface `serviceOrders`)
- **inProgress**: jemand hat ihn übernommen (`assignedTo` gesetzt, `startedAt` Timestamp)
- **done**: abgeschlossen (`completedAt` Timestamp)

Sowohl Mensch als auch Agent können `claim_service_order` aufrufen (Selbstzuweisung ist
gewünscht, nicht nur erlaubt — alles wird geloggt/auditiert, das ist der Kontrollmechanismus,
keine Vorab-Sperre).

## Objekt anlegen (manuell, zwei Schritte)

```json
// 1. serviceOrder-Objekt erzeugen
{"name":"onto_object_create","arguments":{
  "type":"serviceOrder",
  "name":"TEST - <Beschreibung>",
  "scope_id": <eigener Write-Scope>,
  "properties":{"task":"<Aufgabenbeschreibung>","priority":"low|medium|high","status":"open"}
}}

// 2. Mit Standort verlinken (Standort kann in fremdem read-only Scope liegen — SSR greift)
{"name":"onto_object_create_link","arguments":{
  "linktype":"hasServiceVisit",
  "object1_udid":"<standort-udid>",
  "object2_udid":"<serviceOrder-udid>"
}}
```

Der Link erscheint am Standort-Objekt als `hasServiceVisit` (verb `mo.hasServiceVisit`,
reverse verb `mo.takesPlaceAt`).

## Objekt anlegen (direkt am Standort, kürzer)

Viele Objekttypen haben inzwischen eine `create_service_order`-Action direkt im Schema
(prüfbar über `onto_type_show`, Feld `schema.action_keys`). Das spart den manuellen
create+link-Schritt:

```json
{"name":"onto_object_execute_action","arguments":{
  "action_key":"create_service_order",
  "udid":"<standort-udid>",
  "prompts":{"task":"...", "priority":"low"}
}}
```

## Auftrag steuern

```json
{"name":"onto_object_execute_action","arguments":{"action_key":"claim_service_order","udid":"<serviceOrder-udid>"}}
{"name":"onto_object_execute_action","arguments":{"action_key":"complete_service_order","udid":"<serviceOrder-udid>"}}
```

## Ergebnis prüfen

`onto_object_show` auf die serviceOrder-udid liefert `status`, `assignedTo`, `startedAt`,
`completedAt`. Fotos/Kommentare, die der ausführende Mensch hinzufügt, landen als eigene
`comment`-Objekte (verlinkt via `hascomment`) bzw. in der strukturierten Bildablage des
verlinkten Standort-Objekts — dort nachschauen, nicht nur am serviceOrder selbst.

## Poll statt raten

Statt periodisch `onto_object_show` auf die serviceOrder-udid zu wiederholen, den
Poll-Cursor nutzen (`references/features/auth-token-lifecycle.md` beschreibt das
verwandte Auth-Pattern, `onto_watch_changes` mit `action` als Filter, z.B.
`action: "claim_service_order"`, spart Calls und Credits).

## Berechtigungsgrenze (wichtig)

Schreibrecht auf einen Scope bedeutet NICHT automatisch das Recht, `serviceOrder`-Objekte
anzulegen oder `create_service_order`/`claim_service_order`-Actions auszuführen — das kann
separat RBAC-gegated sein (403 trotz sonst funktionierendem Schreibzugriff ist ein Signal,
das explizit freigeschaltet werden muss, nicht automatisch aus Objektrechten folgt).
