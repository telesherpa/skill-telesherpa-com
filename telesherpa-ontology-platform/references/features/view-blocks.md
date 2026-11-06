# Views without SQL — the building blocks

A view is described, not written: you say which objects, which columns, which
condition — the server builds the query. Two rules follow from that:

- Raw SQL is **not available** to customer accounts. A view that carries hand-written
  SQL is refused with `403`; build it from the blocks below.
- Every built block carries the scope filter, so a view can only ever show what its
  reader may see.

Minimal shape:

```json
{
  "elements": ["filter", "table"],
  "table": {
    "from": "objects",
    "type": "shift",
    "columns": [
      {"key": "tag",  "prop": "shiftStart", "format": "wochentag_datum", "label": "Day"},
      {"key": "zeit", "prop": "shiftStart", "bis": "shiftEnd", "format": "zeitspanne", "label": "Time"}
    ],
    "sort": {"key": "tag", "dir": "asc"}
  }
}
```

`type` is the object type name, `prop` a field key, `format` the display format
(`text`, `datum`, `zeit`, `wochentag_datum`, `wochentag_datum_zeit`, `zeitspanne`,
`kalenderwoche`, `jahr`, `zahl`, `waehrung`, `monat`, `warnung`).

## Whose rows? — the person binding

A view that lives in one scope but shows data from another (a team door) must bind the
rows to the signed-in person. One block does that:

```json
"person": {"link": "worksShift", "richtung": "rueckwaerts", "prop": "email", "type": "person"}
```

The server builds the comparison against the e-mail of the signed-in person — you do not
write `{current_user_email}` yourself. Without a person binding, a view with
`data_scope_id` is refused.

## Time window, filters, sorting

```json
"zeitraum": {"key": "tag", "von": "{period_start}", "bis": "{period_end}"},
"param": "woche",
"span": 7,
"filter": ["station", "status"],
"sort": {"key": "tag", "dir": "asc"}
```

`param`/`span` create the period switch in the header (`?woche=YYYY-MM-DD`); the
`zeitraum` uses the resulting placeholders. `filter` lists column keys — the value comes
from the filter bar, never from the definition.

## Status — active, deleted, both

By default a list shows living objects only. A recycle-bin view asks for the others:

```json
"table": {"from": "objects", "type": "shift", "status": "deleted", "columns": [...]}
```

`status`: `active` (default), `deleted`, `any`.

## Not yet taken

Rows that carry **no** link of a given type — open shifts nobody is assigned to:

```json
"ohne_link": {"link": "worksShift", "richtung": "rueckwaerts"}
```

Add `"linkvalue"` when only one meaning of the link must be missing.

## Fits me — matching my own links

The row hangs on the same target as the signed-in person, just through another meaning
of the same link:

```json
"passt": {"link": "concerns", "mein_value": "einsetzbar fuer",
          "objekt_value": "Station", "type": "space"}
```

Read as: *person —concerns/"einsetzbar fuer"→ station* and *shift —concerns/"Station"→
station*, and both point at the same object. That is how "open shifts I am qualified
for" is expressed without a second query.

## Rule columns — one check per row

Each rule becomes a column that shows its text when the condition holds, and stays empty
otherwise:

```json
"regeln": [
  {"key": "long",  "feld": "plannedHours", "op": ">", "wert": 10, "text": "over 10 h"},
  {"key": "break", "feld": "breakMinutes", "op": "<", "wert": 30, "text": "break too short"}
]
```

`op` is one of `=`, `!=`, `>`, `<`, `>=`, `<=`; `wert` must be a number.

## Rule columns across rows — per person

These look at the other shifts of the same person:

```json
"personenregeln": [
  {"key": "u18",  "art": "u18", "datum": "shiftStart", "zeit_ab": "22:00", "text": "minor after 22:00"},
  {"key": "day",  "art": "tagessumme", "wert": 10, "datum": "shiftStart", "text": "over 10 h that day"},
  {"key": "rest", "art": "ruhezeit",   "wert": 11, "datum": "shiftStart", "text": "less than 11 h rest"}
]
```

- `u18` — the person on the shift is younger than 18 (optionally only after `zeit_ab`).
- `tagessumme` — the person's planned hours on that day exceed `wert`.
- `ruhezeit` — the gap to the person's previous shift is below `wert` hours.

The person is resolved through the shift's `worksShift` link.

## Counters

```json
"counters": {
  "from": "objects",
  "zaehler": [
    {"type": "shift", "label": "all"},
    {"type": "shift", "label": "open", "filter": [{"feld": "shiftStatus", "op": "=", "wert": "offen"}]}
  ]
}
```

Each counter needs `type`, `label` and — if it filters — a `wert`; a filter without a
value is refused. `"from": "objects"` is required, otherwise the counter block is ignored.

## Pitfalls

- A column `key` is used once per view — including rule columns. A duplicate is refused.
- `prop` must be a field key of the object type; unknown names are refused, not ignored.
- Formats change the display only; sorting and filtering use the raw value.
- The stored `sql` inside a saved view is **generated**. Only a view with
  `"from": "sql"` carries hand-written SQL — and that one needs the operator.
