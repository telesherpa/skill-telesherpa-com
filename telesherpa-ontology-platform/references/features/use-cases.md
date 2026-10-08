# Echte Use-Cases — end-to-end verifiziert

Diese Muster sind nicht hypothetisch — sie wurden in realen Agenten-Sessions komplett
durchgespielt, inkl. echtem Menschen am anderen Ende. Sie zeigen, wie die Einzelfähigkeiten
(Objekte, Scopes, Actions, Marketplace) zu einem tatsächlichen Ergebnis zusammenspielen.

## 1. Dokumentationslücke an einem Feldservice-Standort schließen

**Ausgangslage:** Ein Standort-Objekt in einem öffentlichen, nur lesbaren Referenz-Scope hatte
eine leere Bildablage-Kategorie ("Schlüsseltresor / Position") und keine Vor-Ort-Information.

**Ablauf:**
1. `onto_object_search` (type-Filter oder Name) → Standort finden
2. `onto_file_structured` auf den Standort → Kategorien + vorhandene Bilder sehen (welche
   Kategorie fehlt noch?)
3. `onto_object_addcomment` → Beobachtung festhalten ("Schlüssel im Schlüsselkasten links
   neben dem Tor")
4. `upload_file_to_object` mit `type: structuredimage`, `name: <Kategorie-Dateiname>` (die fehlende
   Kategorie) → Bild landet automatisch im eigenen Schreib-Scope (SSR), **ohne**
   Schreibrecht auf den fremden Referenz-Scope zu brauchen
5. Für tiefere Dokumentation (Foto vor Ort statt Web-Recherche): `onto_object_execute_action`
   mit `action_key: create_service_order` → ein Mensch bekommt den Auftrag, fährt hin,
   macht das Foto selbst, lädt es hoch, schließt ab (`complete_service_order`)

**Lernpunkt:** SSR (Scope-Self-Routing) macht aus einem reinen Datenkonsumenten (read-only
Zugriff auf fremde Referenzdaten) einen aktiven Kurator, ohne dass die Datenhoheit des
Originaleigentümers angetastet wird. Details: `service-order-dispatch.md`,
`structured-image-ablage.md`.

## 2. Öffentlichen Datensatz kuratieren und monetarisieren (Marketplace)

**Ausgangslage:** Öffentliche OpenStreetMap-Daten (Defibrillator-Standorte, Tag
`emergency=defibrillator`) sollten als eigener, wertvollerer Datensatz angeboten werden.

**Ablauf:**
1. Import-Pipeline konfigurieren lassen (Agent liefert die ETL-Spec: Quelle = Overpass-API-Query,
   Ziel-Objekttyp mit Property-Mapping) → Rohdaten landen im neuen Scope
2. `ontouser_scope_index` → Scope-Mitgliedschaft mit `can_manage_scope: 1` bestätigen (Agent
   ist Owner)
3. **Veredelung, nicht nur Durchreichen:** `onto_object_geocode` (`direction: reverse`,
   `apply: true`) auf jedes Objekt → OSM-Rohkoordinaten werden zu strukturierten Adressen
   (Straße, PLZ, Stadt) — genau die Lücke, die Rohdaten von kuratierten Daten unterscheidet
4. `onto_object_search` mit `type`-Filter + Stichprobe → Vollständigkeit prüfen, bevor
   veröffentlicht wird
5. `ontouser_scope_set_visibility` mit `read_allowed_for: ["PUBLIC"]` → Scope wird öffentlich
6. Andere Nutzer arbeiten im Scope → `onto_credit_balance` zeigt `earnings_total` > 0,
   automatisch in `balance` eingerechnet

**Lernpunkt:** Der Wert liegt nicht im Import selbst (das kann jeder), sondern in der
Veredelung (Geocoding, Vollständigkeitsprüfung, Struktur) — das ist, wofür andere Nutzer
tatsächlich zahlen. Details: `marketplace-revenue-share.md`.

## 3. Sensor-Zeitreihen: Flotte und Verlauf

**Ausgangslage:** Ein Scope mit 10 Objekten, von denen eines Fahrzeug-Telemetrie führt —
zwei Reihen, `location` (geo) und `speed`.

**Ablauf:**
1. `onto_object_index` → die Objekt-udids des Scopes
2. `onto_series_last` mit `objects: [alle udids]`, `series: ["location", "speed"]` →
   **ein** Aufruf statt N
3. Für den Verlauf eines einzelnen Objekts: `onto_series_range` mit `resolution: 300`

**Nachgemessenes Ergebnis:** bei 10 angefragten Objekten kommen **2 Werte** zurück — neun
Objekte haben in diesen Reihen keinen Punkt. Das ist kein Fehler, sondern die korrekte Antwort
auf „welche Messwerte gibt es". Die Antwort ist eine Liste **vorhandener Messwerte**, keine
Objektliste.

**Lernpunkt:** Wer „Anzahl Antwort = Anzahl Objekte" erwartet, hält eine korrekte Antwort für
kaputt. Ebenso ist eine **leere** Liste ein gültiges Ergebnis — ein Fahrzeug, das gerade nichts
sendet, ist nicht verschwunden. Alle Zeiten sind UTC mit `Z`. Details:
`sensor-zeitreihen.md`.

## Eigene Automatisierungen bauen

Die gezeigten Bausteine (Schema-Introspektion, Pagination, ServiceOrder-Dispatch) lassen sich
zu eigenen Loops kombinieren — z.B. ein systematisches Vollständigkeits-Audit über einen
Objekttyp, das Lücken automatisch als `serviceOrder` oder Kommentar markiert.

**Vorsicht bei Umsetzung:** Bei Nutzung von `create_service_order` in einer Schleife über
viele Objekte besteht das Risiko, versehentlich viele reale Feldeinsätze auf einmal
auszulösen — vor einem solchen Batch-Lauf unbedingt mit dem Scope-Owner abstimmen, nicht
eigenmächtig in Serie dispatchen.
