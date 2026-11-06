# Start cards: the team's front page, configured once

> Platform reference, 2026-11-06. Verified against the running instance
> (`/onto/startcard`, `onto_startcards`, the four MCP tools). Status: **live**.

## 1. What a start card is

A start card is a saved link to an interface, published for a scope. Every member
who can read that scope finds it on `/ontouser` in the panel **"Startkarten fürs
Team"** — a separate panel, deliberately **not** mixed with the personal
favourites (those have drag handle, pencil and bin; start cards do not).

Two different things, two different tables:

| | personal favourites | start cards |
|---|---|---|
| table | `onto_favorites` | `onto_startcards` |
| who configures | each user, for themselves | whoever manages the scope |
| visible to | that one account | everyone who reads the scope |
| MCP | `onto_favorite_*` | `onto_startcard_*` |

## 2. Visibility — two conditions, both required

A card is shown to a user only if **both** hold:

1. the user can **read the card's scope** (`has_scope`), and
2. the target interface is in **that user's interface list** (which includes the
   interface's own `access.require` gate).

Condition 2 is not cosmetic: without it the card would be a dead link for some of
the team, and a dead link on the front page is worse than no card.

There are **no roles on a card**. A card is for members, full stop.

## 3. Who may configure

`/onto/startcard` checks `can_manage_scope` — an account that manages **no** scope
gets **HTTP 403** with "Sie verwalten keinen Scope", no table, no `+` button. An
admin account may configure all scopes (configuration exception, not a data right).

The menu entry lives under Settings and carries `require_scope_manage`, so it is
hidden for everyone else. Menu visibility is not access control — the page gates
itself, and so does every endpoint (`save`, `delete`, `move` return 403 without the
right).

## 4. The page

House pattern, identical to `/onto/scope`: `+` in the header, one modal for create
and edit, one for delete, pencil and bin in the row, real POST with a redirect and
a flash message. `?format=json` on the same endpoints returns JSON for agents.

## 5. MCP tools

| Tool | Purpose |
|---|---|
| `onto_startcard_index` | list the cards the caller may see |
| `onto_startcard_save` | create or change (`id` optional) |
| `onto_startcard_move` | `id` + `dir` = `up` / `down` |
| `onto_startcard_delete` | `id` |

They take **no** `roles` parameter. Ordering is by `sort_order`, maintained by the
move call.

## 6. When a customer asks for "my area"

Do **not** build a special page. The `blocks` element is live: an interface can
carry `blocks` and inside it several building blocks (`table`, `cards`), each with
its own filters. That is the supported way for a workspace to get its own front
page — the start cards then point at it like at any other interface.
