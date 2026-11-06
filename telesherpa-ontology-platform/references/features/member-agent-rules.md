# Rules for a member agent (no admin rights)

> Platform reference, 2026-11-06. A customer agent builds a workspace with normal
> account rights. Everything here is what such an agent gets wrong, and why.

## 1. i18n: keys through the API, never in the files

Texts in views are not written into the template. They are keys, resolved per
language (de/en/fr/it).

- Create a key with `onto_i18n_create` — **without** a `mo.` prefix. The prefix is
  reserved for platform keys, a customer key with it lands in the wrong namespace
  and never resolves.
- Add translations with `onto_i18n_translate`; they are **proposals** (pending) that
  the operator reviews and exports. Until then the view falls back to the literal
  text you wrote — so never rely on "it looks right in German".
- Never edit a language file directly. `mo.php` is an **export target**, not a source.

A hard-coded German string in the view is the classic failure: the workspace is
finished, the customer switches to English, half the page stays German.

## 2. Dual interface: HTML for people, JSON for agents

Every endpoint you build answers in two modes, decided by `?format=json`:

- HTML: form POST → redirect back to the list → flash message.
- JSON: same logic, `{"result":"OK",...}` or `{"result":"Error","error":"…"}` with a
  real HTTP status code.

Without the JSON mode the agent cannot verify its own work, and the customer cannot
automate anything later. The status code matters: an error that returns 200 is
invisible to every client.

## 3. Filters live in the session, never in the query string

`session_filters()` stores list filters per view. Query-string filters leak between
users, break pagination, and end up in links that other people click. Company data
travels in the POST body only — never in a URL that ends up in a log or a bookmark.

## 4. Interface permissions are declared, not implied

`view_config.access.require` on each view:

| value | meaning |
|---|---|
| `"write"` | needs write on the object's scope |
| `"manage"` | needs `can_manage_scope` |
| a number | needs that scope (id) |
| missing | read is enough |
| anything unknown | **denied** (fail-closed) |

A view that shows other people's data without `access.require` is a data leak, not
a style question.

## 5. Placeholders

In interface texts, notifications and action messages:

| placeholder | resolves to |
|---|---|
| `{current_user_email}` | e-mail of the acting account |
| `{current_user_id}`, `{current_user_profile_id}` | its ids |
| `{old:<field>}`, `{new:<field>}` | previous / new value, rendered in the **viewer's** zone |
| `{now}` | current time |

`{old:…}`/`{new:…}` print seconds only when they are not `:00`. A timestamp written
as UTC and shown as local without conversion is the most common "wrong data" report
that turns out to be a formatting bug.

## 6. Deleting: one id at a time, always

Never delete by pattern, prefix or name. Show the list, then delete exact ids. Two
reasons, both learned the hard way: a pattern matches more than expected, and a
delete has no undo. This rule applies to probe objects you created yourself — the
next object with a similar name may belong to the customer.

## 7. Comments in code, never in the output

Notes for the next developer go into PHP comments (`<?php /* … */ ?>`). HTML comments
(`<!-- … -->`) are shipped to the browser and read by every customer. Internal
reasoning does not belong in the markup.

## 8. What a member agent cannot do — and should not fake

- No `onto_type_create` for a type outside the catalogue: that is an admin action;
  file a service request instead.
- No rights for others: granting scope rights is the scope manager's job (or admin).
- No account deletion: managed accounts are removed through the company-manager path.
- No "I will just give myself write access": that is exactly the failure mode the
  bridge in `references/features/scope-bridges.md` exists to avoid. If a write is
  refused, the answer is an action with `result_scope_id`, not a grant.
