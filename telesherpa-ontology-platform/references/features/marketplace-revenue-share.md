# Marketplace & Revenue-Share — eigene Scopes monetarisieren

## Konzept

Ein Agent ist nicht nur Datenkonsument, sondern kann selbst zum Datenanbieter werden: eigenen
Scope anlegen, kuratieren (Import + Veredelung, z.B. Geocoding, Vollständigkeitsprüfung,
Kommentare), öffentlich anbieten — und verdient danach automatisch Credits, wenn andere
Nutzer/Agenten in diesem Scope arbeiten. Der Ersteller bleibt Scope-Manager und sieht
Fremdnutzung/Feedback.

**Bereits live als Showcase:** öffentliche, read-only Referenz-Scopes mit echten
Branchendaten (u.a. Mobilfunk-Infrastruktur aus offiziellen Quellen mehrerer Länder).

## Ablauf

### 1. Eigenen Scope anlegen

```json
{"name":"ontouser_scope_create","arguments":{"name":"meinbereich","description":"..."}}
```
→ Der Name ist frei wählbar (`[a-z0-9_]`, max 30 Zeichen), kein automatisches Präfix.

### 2. Kuratieren

Daten importieren (siehe Import-Pipeline unten), anreichern (z.B. `onto_object_geocode`
für Adressdaten aus Koordinaten), Vollständigkeit prüfen, Metadaten pflegen. Das ist die
eigentliche Wertschöpfung — reines Roh-Daten-Durchreichen ist wenig differenzierend.

### 3. Öffentlich anbieten

```json
{"name":"ontouser_scope_set_visibility","arguments":{
  "scope_id": <eigener scope>,
  "read_allowed_for": ["PUBLIC"]
}}
```
Alternativ `read_allowed_for: [<scope_id_x>]` für Freigabe an bestimmte Nutzergruppen statt
komplett öffentlich. Nur der Owner (`can_manage_scope: 1` in `ontouser_scope_index`) darf
Sichtbarkeit setzen.

### 4. Verdienen — automatisch, kein separater Auszahlungsschritt

`onto_credit_balance` liefert zusätzlich zu `usage_30d` (eigene Ausgaben) auch
`earnings_30d`/`earnings_total` (Einnahmen aus Fremdnutzung des eigenen Scopes). Diese
Einnahmen werden **direkt in `balance` eingerechnet** — keine separate Wallet, keine
manuelle Gutschrift nötig. Beispiel-Antwort:

```json
{
  "balance": 352,
  "earnings_30d": [{"operation":"set_property","credits":2,"count":1}],
  "earnings_total": 2
}
```

## Scope-Status prüfen

```json
{"name":"ontouser_scope_index","arguments":{}}
```
Eigene Scopes stehen unter `own[]`, jeweils mit `can_read`/`can_write`/`can_manage_scope`.
Nur `can_manage_scope: 1` bedeutet Owner-Rechte (Sichtbarkeit setzen, Credits verdienen).
`public[]` listet öffentlich verfügbare Scopes anderer, die man selbst noch nicht
zugewiesen hat (`ontouser_scope_assign_public` zum Beitreten).

## Import-Pipeline

Es gibt eine ETL-Engine für den initialen Datenimport in einen Scope. Ein Agent kann das
Ergebnis **nutzen** (über `onto_type_show`/`onto_object_search` abfragen), die Pipeline
selbst wird von der Plattform konfiguriert.

Wenn ein Import gewünscht ist: Quelle (URL/API) + Ziel-Objekttyp mit Property-Mapping als
ETL-Spec formulieren und der Plattform zur Konfiguration übergeben (siehe
`service-order-dispatch.md` für das verwandte Muster "Agent formuliert, Mensch führt aus").
