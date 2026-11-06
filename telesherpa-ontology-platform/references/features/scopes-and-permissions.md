# Scopes as permission boundaries

> Platform reference, 2026-11-06. Audience: an agent setting up a Telesherpa
> workspace from scratch. Everything here was verified against the running
> instance (`app.telesherpa.com`, PHP side) — not against documentation.

## 1. The one rule

**A scope is a data boundary and a permission boundary at the same time.**
There is no second permission system next to it. Roles exist (`member`,
`admin`), but they only decide *which pages* you get; **what data you may see or
change is decided exclusively by the scope grants.**

Consequence for an agent: if a person must not see something, put the data in a
different scope — do not try to hide it with a view filter. Filters are
convenience, scopes are the lock.

## 2. Tables

| Table | Columns that matter | Meaning |
|---|---|---|
| `scopes` | `id`, `name` | the boundary |
| `scopes_profiles` | `scope_id`, `profile_id`, `active_to`, `can_read`, `can_write`, `can_manage_scope` | the grant |

- `active_to` NULL = unlimited; a date ends the grant automatically.
- Every grant has exactly three switches: read, write, manage.
- Profiles are linked to users **by e-mail** (`profiles.email = users.email`),
  not by a numeric key. Any join you write has to use the e-mail.

## 3. The three levels

| Level | Column | What it allows |
|---|---|---|
| read | `can_read` | see objects of the scope in lists, views and details |
| write | `can_write` | create and change objects in the scope |
| manage | `can_manage_scope` | administer the scope itself: grants to other people, interfaces, action definitions, start cards |

`can_manage_scope` does **not** imply `can_write` and does not imply `can_read`.
Set what you mean; the platform does not fill the gaps for you.

The PHP helpers mirror the columns one to one: `user_read_scopes()`,
`user_write_scopes()`, `user_manage_scopes()`, `has_scope($id)` (read) and
`has_scope_mode($id, 'write'|'read')`.

## 4. Fail-closed, in every direction

- No grant → the scope does not exist for that account. Empty list means empty,
  never "all".
- **There is no admin shortcut in the data path.** An admin account sees only
  what is granted to it. This was a deliberate fix after a graph leak; do not
  reintroduce `if (is_admin) return all`.
- The single, explicit exception: in `/onto/scope` an admin may grant scopes
  **to themselves** (bootstrap). Everywhere else `has_role('admin')` is not a
  key to data.
- Configuration pages may carry an admin exception (for example the start card
  page lets an admin configure every scope). That is a *config* exception, not a
  data permission — the data gate stays.

## 5. PUBLIC scope

Every account gets the `PUBLIC` scope **read-only, lazily** on its first request
(`ensure_public_scope_for_profile()`), so a fresh account starts with something
visible instead of an empty screen. Do not grant it write access; do not use it
as a shared working scope.

## 6. Where the gates sit

| Place | Gate |
|---|---|
| view (`view_config.access.require`) | `"write"` → needs `can_write`; `"manage"` → needs `can_manage_scope`; a number → that scope id; **missing → read is enough**; any unknown value → **denied** |
| action definition | `require_scope_write` on the scope of the action; the executing account must hold `can_write` there |
| own endpoints | check `has_scope`/`has_scope_mode` again — a hidden menu item is not a lock |
| sidebar (`sidebar_menu` setting) | `required_role` and `require_scope_manage`, evaluated for top-level entries **and** for `children` |

**A menu entry is not access control.** Hide it *and* gate the page. We shipped
a page that was invisible in the menu and still fully usable by direct URL — a
tester found it within minutes.

## 7. The three-scope planning pattern

The reference layout for a shift-planning workspace, one tree per customer:

| Scope | Purpose | Who gets what |
|---|---|---|
| `<company>_stammdaten` (master data) | people, stations, qualifications, contract data | managers: read + write; staff: nothing |
| `<company>_planung` (planning) | shifts, absences, demands, periods | managers: read + write; **staff: nothing at all** |
| `<company>_team` (team) | the staff's own doors: my shifts, swap requests, clock in/out | staff: read (+ write only where a door needs it) |

Why staff get **no read** on the planning scope: reading it means reading *every
colleague's* shifts, absences and corrections. The team scope exists precisely so
that the planning scope can stay closed. This is the mistake that costs a
customer's trust, not a technicality.

Everything a staff member is allowed to *change* in the planning scope happens
through an action with `action.result_scope_id` — see
`references/features/scope-bridges.md`.

## 8. Checklist before you hand a workspace over

1. For every scope: who reads, who writes, who manages — written down, not implied.
2. No account has grants it does not need (check the team accounts, not just the admins).
3. Every view has `access.require` set where it shows personal data.
4. Every own endpoint re-checks the scope, even if the menu hides it.
5. The staff account (the real one, not a manager) has been used once: empty
   menus, direct URLs, wrong-scope calls.
6. `PUBLIC` is read-only for everybody.
