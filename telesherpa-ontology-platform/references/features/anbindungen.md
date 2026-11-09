# Anbindungen (connections, write-back, outbox)

Umbrella term for the four write-back building blocks: **Verbindung** (vault credential),
**Weitergabe** (`http_call` effect), **Ausgang** (outbox queue), **Aenderungsfeed** (`watch/changes`).

## Typical setup sequence

1. `onto_connection_check` - is the vault usable at all?
2. `onto_connection_save` with a `target_key` such as `sap_prod` and the secret.
3. `onto_outbox_preview` with the planned `http_call` (without `send`) to see method, URL and body.
4. Build the rule with `onto_trigger_create` or the action definition, effect type `http_call`.
5. Watch `onto_outbox_index`; resend a single row with `onto_outbox_retry`.

## Change and removal

- **Change**: `onto_connection_save` with the same `target_key`. The row is replaced, the id
  stays, there is no history.
- **Remove**: `onto_connection_remove` with the `id` from `onto_connection_index`. Only your own
  connections are deletable; foreign ones answer 404 without disclosing that they exist.
- Before removing, check `onto_outbox_index` - queued rows for that target will fail afterwards.

## Rules of the road

- The secret never comes back out. `secret_set` only says whether one is stored.
- A connection belongs to exactly one identity (the profile that created it).
- Delivery is idempotent per outbox row; a retry does not send twice.
