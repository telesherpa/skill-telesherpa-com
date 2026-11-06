# Tool traps: what an agent gets wrong about the MCP tools

> Platform reference, 2026-11-06. Every item was produced by a real failed call or
> read from the running server/code — not from a schema reading.

## 1. Never trust a number in a document — measure it

Tool counts change with every release. The authoritative source is the service:

```
GET https://mcp.telesherpa.com/health
→ {"status":"ok","serverVersion":"2.3.0","tools":201,...}
```

`tools` is the catalogue size. What a *specific account* sees depends on its roles,
so an agent that compares "I see 137, the doc says 201" is comparing two different things.

Measure per account with `tools/list` — but note: **`tools/list` needs a session**.
Call `initialize` first, keep the `mcp-session-id` response header, then send
`tools/list` with it. Without the header the server answers `400 Missing session ID`,
which reads like a broken server and is really a missing handshake step.

## 2. `admin` is not a superset of `member`

The role filter is "the account holds **at least one** of the required roles". An
account with only `admin` does **not** see the `member` tools. So "the admin sees
everything" is false unless the account carries both roles — check with
`auth_status`, which returns the visible and total tool counts for the calling
account.

## 3. Undeclared parameters are rejected — loudly, and that is good

The tools validate their signature. Passing an unknown parameter returns

```json
{"status":"error","error":"unbekannte Parameter: udid","erlaubt":["parent","scope_id","text"],
 "hinweis":"…Ein Parameter ohne Deklaration wird abgelehnt, damit ein wirkungsloser Aufruf nicht wie eine Wirkung aussieht."}
```

Read `erlaubt` and retry. Never assume REST habits:

| Tool | Actual signature |
|---|---|
| `onto_object_addcomment` | `parent` (the object UDID), `scope_id`, `text` — **no** `udid`, no `msgKind` |
| `onto_object_show` | takes `udid` |
| `get_document` | takes `filetoken` or `urlslug` — **not** the file `id` from an attachment list |
| `onto_settings_update` | needs `id` **and** `key` **and** `value` — `value` alone writes nothing |
| `onto_interface_index` | takes `limit` only, no `page`/`per_page` |
| `onto_interface_update_def` | needs `def_id` **and** `label` — without `label` the call is silently ineffective |

## 4. Silent no-ops are the expensive ones

A rejected call is cheap; a call that "succeeds" and changes nothing costs a
debugging round. Three habits:

1. **Read back after every write** — `onto_object_show`, `onto_interface_show`,
   `onto_settings_index`. The response of a write describes intent, the read-back
   describes state.
2. **Compare a value you set, not a status code.** `{"result":"OK"}` on an update
   only says the request was accepted.
3. **For definitions, the saved body is the truth** — save, then list and diff
   against what you sent.

## 5. Costs: who pays, and when the call stops working

The **executing account** pays. A bridge, a manager action or a proxy does not move
the bill.

| Action | Credits |
|---|---|
| read / graph | 1 |
| `set_property` | 2 |
| `addcomment`, `create_link`, `update_object`, `update_link`, `bulk_action_per_object`, `file_upload` | 5 |
| `create_object` | 10 |
| **`action`** | 20 |
| `delete_object`, `delete_link`, `delete_property` | 20 |
| `complete_service_order` | 50 |
| `file_per_mb` | 1 per MB |

Scope tiers (one-time, per scope count): 1/2/3/5/10 scopes = 50/75/100/150/250 credits.
1 credit = 0.01 EUR. **No signup credits** — a fresh account starts at 0.

`HTTP 402` means the executing account is out of credits. It is not a server error,
not a permission error, and retrying does not help; the balance has to be topped up
by the operator. Test runs that create *and delete* objects burn credits twice —
budget the deletes.

`onto_credit_pricing` returns the live table. Read it at the start of a run instead
of quoting numbers from this file.

## 6. Time

- **Writes** are interpreted as `Europe/Berlin`; **reads** return UTC.
- Whoever sends `2026-10-12 20:28:00` later reads `18:28:00Z` — the same instant,
  not a shift.
- For triggers and functions, always ask for the local zone explicitly when a human
  will read the value (`{now}` in texts, `utc_nach_zone()` in output formatting).

## 7. Triggers are asynchronous

A trigger does not fire inside the same call. Expect one to five minutes, and
**never** resend a write because "nothing happened yet" — resending creates the
duplicate you were afraid of. Read the object, then read `onto_trigger_index` /
`onto_watch_changes` to see whether the event is queued.

## 8. The dispatcher is bounded

Scheduled work processes the first 500 objects per type per run. A plan with more
rows will look "half published" for one tick. That is the design, not a failure —
do not loop the publish call to compensate.
