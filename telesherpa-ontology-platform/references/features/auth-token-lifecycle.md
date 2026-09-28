# Auth & Token-Lifecycle — selbstständig authentifiziert bleiben

## Login liefert zwei Tokens

```json
{"name":"login","arguments":{"client_id":"EMAIL","client_secret":"PASSWORD"}}
```
→
```json
{
  "token": "...",            // = access_token, Bearer
  "token_type": "Bearer",
  "expires_in": 36000,       // 10h
  "refresh_token": "..."     // 30 Tage gültig
}
```

`client_secret` = Passwort. **Nur der Mensch darf `login` aufrufen** (Passwort-Eingabe ist
für einen Agenten grundsätzlich tabu). Alles danach — beliebig lange Weiterarbeit — läuft
über den Refresh-Token, ohne dass ein Mensch je wieder ein Passwort eingeben muss.

## ⚠️ Es gibt ZWEI Betriebsarten — der Unterschied ist entscheidend

Der Server läuft **session-basiert**.
Ein Bearer-Token allein genügt **nicht** — die Session-ID ist Pflicht:

```
{"jsonrpc":"2.0","id":null,"error":{"code":-32600,
 "message":"Bad Request: Missing session ID"}}
```

Dieser Fehler kommt bei **jedem** `tools/list` und `tools/call` ohne Session-ID — auch mit
perfekt gültigem Bearer-Token. **Bearer ohne Session → abgelehnt; Bearer mit Session
→ `auth_status` antwortet `"status": "ok"`.

### Betriebsart A — klassisch, mit MCP-Session (der Normalfall)

```bash
# 1. Session holen (initialize) — die mcp-session-id kommt im RESPONSE-HEADER
curl -s -D /tmp/mh.txt -X POST "https://mcp.telesherpa.com/?format=json" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'Authorization: Bearer <TOKEN>' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"cli","version":"1.0"}}}'
grep -i "mcp-session-id" /tmp/mh.txt | tr -d '\r' | awk '{print $2}'
```
**Diesen Header bei jedem weiteren Aufruf mitsenden.** Lebensdauer: 12 h Inaktivität.
Danach neu `initialize` — das ist kein Fehlerzustand.

### Betriebsart B — stateless (Protokoll „Era 2026-07-28")

Clients ohne Session-Slot müssen einen **Envelope** in `params._meta` mitschicken. Fehlt er,
antwortet der Server mit Klartext, was genau verlangt wird:

```
-32602  params._meta must be an object carrying the required
        'io.modelcontextprotocol/protocolVersion' and
        'io.modelcontextprotocol/clientCapabilities' envelope keys
```

Vollständig funktionierender Aufruf (ohne Session-ID, ohne `initialize`) — die drei Header
sind **alle** nötig:

```bash
curl -X POST "https://mcp.telesherpa.com/?format=json" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'MCP-Protocol-Version: 2026-07-28' \
  -H 'mcp-method: tools/list' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{"_meta":{
      "io.modelcontextprotocol/protocolVersion":"2026-07-28",
      "io.modelcontextprotocol/clientCapabilities":{}}}}'
```

Bei `tools/call` kommt **`mcp-name: <toolname>`** hinzu — muss exakt dem `params.name`
entsprechen, sonst:
```
-32020  mcp-name header does not match the request body's 'name' parameter
```
Analog bei falschem Methoden-Header: die Methode im Header muss zum Aufruf passen.

**Diese Betriebsart funktioniert OHNE Token** für die öffentlichen Tools (`register`, `login`,
`refresh_access_token`, `auth_status`, `onto_credit_pricing`).

`register` verlangt **`accept_terms=true`** (Pflicht). Fehlt der Parameter, antwortet der
Server mit HTTP 400 "Bitte bestaetigen Sie die Nutzungsbedingungen." — die AGB-Zustimmung
ist Voraussetzung fuer die Kontoanlage: https://www.telesherpa.com/agb

## Die öffentlichen Tools (belegt, 6 Stück)

```
register, activate, login, refresh_access_token, auth_status, onto_credit_pricing
```

Der Server nennt sie selbst im `www-authenticate`-Header, wenn kein Token mitkommt — samt
Ablauf (`register` → Voucher per E-Mail → `activate` → `login`).

**Wichtig:** `tools/list` **ohne Token** liefert keine Tool-Liste, sondern **HTTP 401** mit
diesem Hinweistext. Die 6 Tools sind über den **Namen** aufrufbar, nicht über die Liste.
Wer sich auf `tools/list` verlässt, sieht ohne Token gar nichts.

## Selbst erneuern: `refresh_access_token`

```json
{"name":"refresh_access_token","arguments":{"refresh_token":"<aktueller refresh_token>"}}
```
→ neues `access_token` + neuer `refresh_token`.

**Rotation + Replay-Schutz:** jedes Refresh liefert **beide** Tokens neu. Wird der
**alte** Refresh-Token ein zweites Mal benutzt:
```json
{"error":"invalid_grant","error_description":"Refresh-Token unbekannt"}
```

**Praxis-Pattern:**
- Proaktiv erneuern, bevor `expires_in` abläuft (nicht erst nach einem 403 reagieren)
- Bei jedem Refresh den **neuen** `refresh_token` sichern — der alte ist danach tot
- `refresh_access_token` ist öffentlich — kein Bearer-Header nötig

## Fallen

### Keine Session → Token wird NICHT gespeichert
Ein `login`-Aufruf **ohne** Session-Slot gibt den Token zwar zurück, speichert ihn aber
absichtlich nicht (Schutz vor einem geteilten Slot für alle Clients). Der Client **muss** ihn
selbst als Header mitsenden — er bekommt es in der Antwort auch so gesagt:
> „Dieser Client hat keine MCP-Session, deshalb wird der Token serverseitig NICHT gespeichert.
> Sende ihn bei jedem folgenden Aufruf als Header 'Authorization: Bearer ...'."

### Der User-Agent beim Login MUSS `Telesherpa-MCP/1.0` sein
Das Backend **bindet den Token an den User-Agent**. Ohne diesen Header beim Login → 403
„Not authorized" bei jedem ToolCall, obwohl der Token gültig aussieht. Gilt für den direkten
REST-Login über den Login-Tool-Aufruf, der den Token für den Bearer-Header liefert.

### Access- und Refresh-Token beide ungültig
Selten, aber möglich (`invalid_grant` auch beim Refresh). Erkennbar: es erscheinen nur noch die
öffentlichen Tools UND der Refresh scheitert. Dann hilft kein weiterer Versuch mit dem alten
Token — es braucht einen frischen menschlichen `login`-Schritt.

## Session-ID ≠ Zugriffsrecht auf fremde Scopes

Eine Session-ID weist den Client aus, **nicht** seinen Zugriffsanspruch: sie gibt niemals
Rechte auf fremde Scopes. Rechte werden ausschließlich serverseitig aus der Rolle des
angemeldeten Kontos abgeleitet.

Daraus folgt für Agenten: **die Session-ID ist nicht teilbar und gehört nicht in Logs oder
Fehlerberichte.** Sie ist der Schlüssel zum Token dieses Clients.

## Diagnose-Reihenfolge bei Auth-Fehlern

1. `auth_status` aufrufen (in der Session) — nennt Zustand und Rollen im Klartext
2. Antwort `Bad Request: Missing session ID` → **kein** Auth-Problem, sondern der
   `initialize`-Schritt fehlt (Betriebsart A) bzw. `params._meta` fehlt (Betriebsart B)
3. `onto_credit_pricing` aufrufen — 401 = Token tot
4. `refresh_access_token` mit gespeichertem Refresh-Token probieren
5. Erfolgreich → neues Token-Paar übernehmen, weiterarbeiten
6. `invalid_grant` → Refresh-Token ist ebenfalls tot → Mensch um neuen `login` bitten,
   keine weiteren Refresh-Versuche mit dem toten Token

## Wenn nichts mehr geht: der Selbsttest per HTTP

Der Server beschreibt sich selbst — man muss nicht raten. Ein Aufruf **ohne** Token liefert die
genaue Anleitung:
```bash
curl -s -X POST "https://mcp.telesherpa.com/?format=json" \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```
→ 401 mit `www-authenticate`-Header, der die öffentlichen Tools, den Ablauf und den Hinweis auf
den Voucher nennt.
