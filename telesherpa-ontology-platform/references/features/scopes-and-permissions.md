# Scopes: how access is organised

> Reference for agents building a workspace. It describes what you configure and
> what you can observe — not how the platform is implemented.

## 1. The model

A **scope** groups data *and* the people who may work with it. There is no second
permission system: what an account may see or change is decided by its grant on a
scope.

Practical consequence: if a person must not see something, put the data in a
different scope. Hiding it in a view is convenience, not a boundary.

## 2. Three levels per grant

| Level | What it allows |
|---|---|
| read | see objects of the scope in lists, views and details |
| write | create and change objects in the scope |
| manage | administer the scope: grants for other people, its interfaces, action definitions, start cards |

The levels are independent: `manage` does not imply `write`, and not `read`.
Grant what the person actually needs.

## 3. Empty means empty

An account without a grant does not "see everything by default", and it inherits
nothing from another scope. A list without a grant is an empty list.

Two practical rules follow:

- Check the real account, not just your own. A manager who sees a page does not
  prove that a staff account sees it.
- After changing a grant, verify with the affected account: sign in (or call the
  API) as that account and confirm the expected result **and** the expected refusal.

## 4. A public area for new accounts

Every new account automatically gets read access to the public scope, so the first
screen is not empty. Treat it as read-only: do not build a customer's work area
there, and do not give it write access.

## 5. Where access is enforced

| Place | How you declare it |
|---|---|
| a view | `access.require` in the view configuration: `"write"`, `"manage"` or a scope id; missing means read is enough; an unknown value is denied |
| an action | the action carries the scope it runs in; the executing account needs write there |
| your own endpoint | check the scope in the endpoint as well, even if the menu already hides the page |
| the sidebar | `required_role` and `require_scope_manage` — a menu entry can be hidden per account |

**A hidden menu entry is not a lock.** Both doors belong together: hide the entry
*and* check inside the page. A page that only disappears from the menu is still
reachable by URL.

## 6. A layout that works for shift planning

Three scopes per customer, one tree:

| Scope | Content | Staff | Managers |
|---|---|---|---|
| master data | people, stations, qualifications, contract data | no access | read + write |
| planning | shifts, absences, demands, periods | **no access** | read + write |
| team | the staff's own doors: my shifts, swap requests, clock in/out | read, and write only where a door needs it | read |

Why staff get **no read** on planning: reading it means reading every colleague's
shifts, absences and corrections. The team scope exists so that the planning scope
can stay closed. Everything a staff member may change in planning runs through an
action with a target scope — see `references/features/scope-bridges.md`.

## 7. Hand-over checklist

1. For every scope, write down who reads, who writes, who manages.
2. Remove grants nobody needs — look at the staff accounts, not only the managers.
3. Set `access.require` on every view that shows personal data.
4. Give every own endpoint its own scope check.
5. Walk through the workspace once with a real staff account: menus, direct URLs,
   calls against a foreign scope.
6. Confirm the public scope is read-only for everybody.
