# Building checkpoint rounds with QR — work instruction with proof

**Goal:** A guard or security service acknowledges its patrol rounds on site by QR code, and afterwards
there is a record that also withstands an audit — one row per check with time, point,
account and a plausibility rating.

**For whom:** Operator/administrator. Object types are platform schema (`onto_type_create` is
the operator's job); customer agents work with the types that are there and build actions,
views and reports on them.

The technical behaviour is documented in `references/features/scan-and-checkpoints.md` (server contract,
execution rule, field formats). What follows is the order in which you build something like this.

## 0. Decisions before building (do not guess)

| Question | Why it is clarified beforehand |
|---|---|
| Which patrol rounds (`tour`) and sections (`section`) exist? | They produce the list of points, not the other way round |
| How often must each point be checked (`interval_min`)? | Determines the “too early / overdue” rating in the record |
| Who may acknowledge? | Role and scope; `PUBLIC` in the scope means reading, **not** scanning along |
| What happens when a check is missed? | Today only visible in the report. An alarm is a separate order (trigger/function) |
| Must it be proven **who** checked, or also **where**? | Today the device position is **not** part of the record. Anyone who needs it must commission it explicitly |
| How much feedback is reasonable? | Sound, colour and haptics per level; too much feedback gets tuned out |

## 1. Types and fields (a prerequisite, not your step)

```
onto_type_create        name="checkpoint", label="mo.Checkpoint",
                        description="Kontrollpunkt fuer den Rundgang: QR-Tag am Ort"
onto_property_def_create key="code",         value_type="string",   label="mo.Code"
onto_property_def_create key="title",        value_type="string",   label="mo.Title"
onto_property_def_create key="section",      value_type="string",   label="mo.Section"
onto_property_def_create key="tour",         value_type="string",   label="mo.Patrol"
onto_property_def_create key="interval_min", value_type="int",      label="mo.Interval_Min"
onto_property_def_create key="checked_at",   value_type="datetime", label="mo.Checked_At"
onto_property_def_create key="checked_by",   value_type="string",   label="mo.Checked_By"
onto_property_set_create key="checkpoint_props", label="mo.Checkpoint_Props",
                        def_ids=[…], type_ids=[<Typ-ID von checkpoint>]
```

Rules for this:

- **Field names in English, label as i18n key** (`mo.…`). If the key does not exist,
  UI and introspection show the raw key. First read `onto_i18n_index`/`onto_i18n_matrix`,
  then `onto_i18n_create`; the assumption/the export is done by the operator.
- **Do not rebuild the address:** the set `address` (`street`, `postalCode`, `city`, `latitude`,
  `longitude` with geocode and reverse layout) exists and is bound to the type.
- `checked_at`/`checked_by` are **state fields**: they sit on the object so that a list “when
  last checked” works without evaluating the log. The history still remains in the log.
- Set required fields (`required`) sparingly — a required field blocks every execution.

## 2. Create points and print codes

```
onto_object_create      type="checkpoint", name="KP-01 Haupteingang",
                        props={code:"KP-01", title:"Haupteingang", section:"Erdgeschoss",
                               tour:"Rundgang Nord", interval_min:60}
```

- The QR code contains the **UDID** of the object (32 hex characters), nothing else. No plain text, no
  URL with parameters: the UDID is the key, the meaning is in the properties.
- Print codes **durably**: a torn-off or pasted-over code is the most common failure.
- A point that is never checked does not belong in the round — otherwise the record permanently
  rates it “overdue”.

## 3. Build the action

```
onto_actiondef_save   key="qr_control", label="Kontrolle quittieren",
                      object_types=["checkpoint"], scopes=[<Kundenscope>], roles=["member"],
                      body={"action":{"type":"update",
                                      "props":{"checked_at":"{now}","checked_by":"{user_email}"}},
                            "scan":{"ausfuehrung":"auto"},
                            "counter":{"label":"Kontrollen heute","sql":"… {scope_where}"},
                            "feedback":{…}}
```

- **No link** in this action: a checkpoint has no counterpart, so a scan is enough
  and the action may run immediately (`scan.ausfuehrung: "auto"`).
- An action that **resolves** a link (e.g. package from the shelf) **never** runs automatically —
  the server enforces that, regardless of `ausfuehrung`.
- Always formulate `condition` on the server side (e.g. “only when the order is open”), with
  `message` in plain text for the user.
- Feedback in three stages: scan accepted, result, error. For each stage sound, colour, haptics and
  duration; `dedupe_ms` against double scans.

## 4. Acceptance (on the running service, not at the desk)

1. **Introspection:** `onto_type_show` on the type — fields and action keys must appear,
   labels in the target language.
2. **Real case:** run one real scan per round (`QR code → action`).
   Then check on the object: `checked_at` set, `checked_by` = account.
3. **Log:** `onto_action_logs` must carry one row per execution with action key, object,
   account and `scope_id`.
4. **Execution rule:** a resolving action must show buttons, a harmless one runs immediately.
5. **Counter-check negative case:** the same code twice within the dedupe window must not book
   twice; an unknown code must not offer any action.
6. **Record:** open the report and read it against your own scans.

## 5. Proof (report “Kontrollen”)

Per row: time, checkpoint, scope, account, result, interval to the previous check at the same
point, target from `interval_min`, rating, previous point, travel time, round.

- Rating in the SQL, **not in the scan**: `erste Kontrolle` (no predecessor), `zu frueh` (< target),
  `ueberfaellig` (> 2 × target), otherwise `ok`. The factor is an agreement with the client —
  it belongs in writing, not set silently.
- **Travel time** = minutes since the last check by the same account at a **different** point.
  Conspicuously small values (under two minutes) are the best indication of ticking off without changing location —
  which is why the column is in the record and not in a special evaluation.
- Filters: scope, checker (email), period. Filter values go **only via POST** in the
  order of the `search_cols`; the JSON/GET route ignores them.
- **Write the scope out in the report SQL.** `{scope_where}` is **not** replaced in the report (that
  only applies to actiondef counters and interface SQL): with the placeholder the query aborts, without a
  scope condition it evaluates across all scopes.
- `candownload = 1` releases the CSV export (for audits and files).
- `candownload` also means: what the report shows can be taken away as a file. Scope and filter
  are therefore part of the promise to the customer, not just a display question.

## 6. Keep an eye on misuse

The record is only as good as its side calculation. That is why you supply from the start:

- travel-time column (ticking off without changing location),
- counter in the actiondef (“Kontrollen heute”) as an immediate check for the shift supervisor,
- `scope_id` in the log so that the rows remain attributable to the customer,
- if required a second code per point (object + location) when the process needs a link.

What the record **cannot** do, and that belongs in the handover: it proves time, account, object and
sequence — not presence at the location. Presence requires the device position (separate
order, see `references/features/scan-and-checkpoints.md`).
