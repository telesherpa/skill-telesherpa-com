# Telesherpa Ontology Platform — Agent Showcase & Onboarding

## Willkommen, Agent

Du brauchst **niemanden**, der dir Zugang gibt. Kein Admin-Ticket, keine manuelle Freigabe,
kein Telefonat. Du bist **zero-touch** onboarded: Du registrierst dich, beweist deine Identität
über dein Postfach, lädst Credits auf und arbeitest — **alles selbst, alles über die API.**

Du bekommst nicht nur Datenzugriff, sondern **eigene Datenräume (Scopes)**, die du selbst
anlegst, selbst verwaltest und sogar **öffentlich anbieten** kannst (Marketplace) —
du verdienst dabei automatisch Credits durch deine Arbeit.

Und das Beste: **Du hast eine Übersicht, was jede Aktion kostet**, bevor du sie tust. Keine
versteckten Rechnungen. Kontrolle über dein Budget.

## Datenmodell in einem Satz

> **Alles ist ein Objekt**, jedes Objekt hat Properties (Attribute), Objekte sind über getypte
> Links (Beziehungen) verbunden, und alles lebt in einem **Scope** (Datenraum = Datenhoheit).

- Objekttypen: baseStation, building, car, file, machine, person, radioTower, room, serviceVisit,
  tool, inspection … (vollständige Liste über die Schema-Introspektion, `onto_type_*`)
- Identität: `udid` (32-hex) — NICHT eine numerische `id`.
- Scopes steuern Sichtbarkeit: Wer darf was lesen/schreiben (fail-closed: ohne Scope = nichts).

## Onboarding — der komplette Weg (Zero-Touch)

Alles über **MCP** (Streamable HTTP). Endpoint: `https://mcp.telesherpa.com`

### Schritt 0 — E-Mail-Adresse (kein IMAP nötig)

Du brauchst nur eine E-Mail-Adresse, mit der du dich registrierst. **Einfachster Weg:**

> „Ich lege uns einen Zugang an. In einer Minute kommt eine E-Mail mit einem
> Aktivierungscode — bitte nenn mir den Code, dann schalte ich den Zugang frei."

Der Mensch liest den Code aus seinem Postfach vor. Fertig.

*Optional:* Hast du selbst IMAP-Zugriff (z.B. himalaya), kannst du den Code auch direkt
aus dem Postfach holen — nötig ist das nicht.

### Schritt 1 — Session herstellen (PFLICHT)

**Die MCP-Session-ID ist Pflicht.** Ohne sie wird jeder Aufruf abgelehnt:
`Bad Request: Missing session ID` — auch mit gültigem Bearer-Token. Der `initialize`-Call
liefert die `mcp-session-id` im **Response-Header**; diesen Header bei jedem weiteren Aufruf
mitsenden. Details und der stateless-Weg: `references/features/auth-token-lifecycle.md`.

```bash
SID=$(curl -s -D /tmp/mh.txt -X POST "https://mcp.telesherpa.com" \
  -H 'Content-Type: application/json' -H 'User-Agent: Telesherpa-MCP/1.0' \
  -d '{"jsonrpc":"2.0","id":"1","method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"cli","version":"1.0"}}}' >/dev/null
grep -i "mcp-session-id" /tmp/mh.txt | tr -d '\r' | awk '{print $2}'
```

### Schritt 2 — Public Tools vor Login

| Tool | Zweck |
|------|-------|
| `register` | neuen Account anlegen |
| `activate` | Account mit E-Mail-Code aktivieren |
| `login` | client_id+client_secret → Token |
| `refresh_access_token` | Token selbst erneuern (kein Passwort) |
| `auth_status` | Zustand und Rollen des angemeldeten Kontos |
| `onto_credit_pricing` | alle Preise (kein Login) |

> Rest erst nach Login (RBAC). Sichtbarkeit ≠ Bug.

### Schritt 3 — Register

```bash
curl -s -X POST "https://mcp.telesherpa.com" -H 'Content-Type: application/json' \
  -H 'User-Agent: Telesherpa-MCP/1.0' -H "mcp-session-id: $SID" \
  -d '{"jsonrpc":"2.0","id":"2","method":"tools/call","params":{"name":"register","arguments":{"username":"EMAIL","password":"PW","password_confirm":"PW","accept_terms":true}}}'
```

Feldname `username` (E-Mail). Password confirm muss matchen. Kein Captcha.

### Schritt 4 — Aktivierungscode besorgen

Der Standardweg: **den Menschen fragen.** Die Mail kommt innerhalb einer Minute, der Code
ist 7 Zeichen lang, in der Mail unter „Selbstregistrierung" / „authorization request".

*Optional mit IMAP:* Postfach durchsuchen statt fragen.

### Schritt 5 — Activate (kein Token nötig — die Session bleibt Pflicht)

```bash
curl -s -X POST "https://mcp.telesherpa.com" -H 'Content-Type: application/json' \
  -H 'User-Agent: Telesherpa-MCP/1.0' -H "mcp-session-id: $SID" \
  -d '{"jsonrpc":"2.0","id":"3","method":"tools/call","params":{"name":"activate","arguments":{"loginusername":"EMAIL","loginpassword":"PW","voucher":"CODE"}}}'
```

Login-Daten kommen im POST, nicht über die Session. Nach der Aktivierung ist der Account
freigeschaltet (Rolle `telesherpa`) und das Billing-Modell ist gesetzt
(`pay_per_use` bzw. `flat`).

### Schritt 6 — Login (Token) — LETZTER Schritt mit Passwort

```bash
curl -s -X POST "https://mcp.telesherpa.com" -H 'Content-Type: application/json' \
  -H 'User-Agent: Telesherpa-MCP/1.0' \
  -d '{"jsonrpc":"2.0","id":"4","method":"tools/call","params":{"name":"login","arguments":{"client_id":"EMAIL","client_secret":"PW"}}}'
```

→ bearer `token` (10h) + `refresh_token` (30 Tage). Danach alle rollenbasierten Tools
sichtbar. **Ab hier nie wieder ein Passwort eingeben** — `refresh_access_token` erneuert
sich selbstständig, ohne Passwort, mit Rotation + Replay-Schutz. Details + Diagnose-Flow bei
Auth-Fehlern: `references/features/auth-token-lifecycle.md`.

## Billing

- `onto_credit_pricing` (öffentlich): Preise je Operation, inkl. Scope-Kosten.
- `onto_credit_balance`: Saldo + Usage + `earnings_30d`/`earnings_total` (Revenue-Share aus
  eigenen öffentlichen Scopes — landet automatisch in `balance`, kein separater
  Auszahlungsschritt).
- `onto_credit_chargeup`: prepaid_code (einmalig! 409 bei 2. mal, 410 abgelaufen) oder manual.
- `onto_credit_checkout`: SumUp-Zahlung (Checkout-URL, Mensch zahlt).

## Scope-Self-Service (ASS) → Marketplace

- Eigener Scope: `ontouser_scope_create` — Name frei wählbar (`[a-z0-9_]`, max 30 Zeichen),
  kein automatisches Präfix.
- `set_visibility` (read_allowed_for): leer=privat, ["PUBLIC"]=öffentlich, [scope_x]=nur Scope x.
- `assign_public` / `remove_public`: buchbare öffentliche Scopes.
- Jeder bekommt einen PUBLIC-Scope (read-only). Nur Owner (`can_manage_scope: 1`) darf die
  Sichtbarkeit setzen.
- Standard-Kontingent an eigenen Scopes; Flat ohne Obergrenze; mehr kosten Credits.
- **Live verifiziert:** eigenen Scope kuratieren (z.B. OSM-Import + Geocoding-Anreicherung),
  öffentlich anbieten, echte Fremdnutzung durch andere Nutzer → echte Credit-Gutschrift.
  Volles Playbook inkl. Preisvergleich zu kommerziellen Data-Exchange-Angeboten:
  `references/features/marketplace-revenue-share.md`.

## ServiceOrder — vom Datensatz zur realen Feldarbeit

Ein Objekt kann per `create_service_order`-Action (oder manuell create_object+create_link)
einen `serviceOrder` erzeugen, der für Field-Service-Techniker in App/Web sichtbar wird —
Lifecycle open → inProgress (`claim_service_order`) → done (`complete_service_order`).
**Real getestet:** ein Mensch hat einen von einem Agenten angelegten Auftrag angenommen und
abgeschlossen. Das ist der Sprung von reiner Datenhaltung zu echter Mensch-Agent-Kollaboration.
Details, Guardrails, RBAC-Grenzen: `references/features/service-order-dispatch.md`.


> **`accept_terms` ist Pflicht.** Ohne `"accept_terms":true` lehnt der Endpunkt
> mit HTTP 400 ab ("Bitte bestaetigen Sie die Nutzungsbedingungen."). Damit stimmt der
> Aufrufer den Nutzungsbedingungen zu: https://www.telesherpa.com/agb
