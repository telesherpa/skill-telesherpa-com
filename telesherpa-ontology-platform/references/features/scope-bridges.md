# Scope bridges: `action.result_scope_id`

> Platform reference, 2026-11-06. Audience: an agent building a workspace where
> staff act in a *team* scope but the result has to land in a *planning* scope.
> Verified against `application/controllers/base.php` (bridge) and the action
> definition validator.

## 1. The problem it solves

A staff member must be able to clock in, request a swap or hand in a wish — and
the result belongs in the planning scope, where the person has **no write
access at all** (and, in the reference layout, no read access either).

Granting write access to the planning scope would hand over the whole plan. So
the platform does not grant a right; it opens a **door for exactly one write
operation**:

```json
"action": {
  "type": "update",
  "result_scope_id": 80,
  "props": ["ist_beginn", "ist_ende"]
}
```

## 2. What the platform does with it

Three protected members on the base controller implement the bridge:

| Function | Job |
|---|---|
| `bridge_set($scope_id, $obj_udid_hex, $props)` | open the bridge for one object and a fixed list of property keys |
| `bridge_allows($obj_udid_hex, $key, $prop_scope_id)` | is *this* write covered? |
| `bridge_clear()` | close it again — always in the same request, after the action |

`assert_scope_write_by_udid()` accepts a write without `can_write` **only** if
all of these hold:

1. a bridge is open (`bridge_scope > 0`),
2. the target object is exactly the bridged object (UDID compared in
   **lower case** — the platform stores hex lower case, mixed case silently
   never matches),
3. the property key is in `action.props`,
4. if the property carries its own scope, it equals the bridge scope.

`bridge_clear()` runs after the action. The right does **not** survive the
request, not even for the same session.

## 3. Rules the validator enforces when saving the definition

An agent that gets these wrong builds a definition that looks fine and breaks on
the first click, in front of the customer:

- `result_scope_id` must sit inside `action`. A value at the top level of the
  body is **not read** — it is rejected with an explicit error instead of being
  ignored silently.
- The body must **bind the acting person**, otherwise the definition is refused:
  one of `{current_user_email}`, `{current_profile_id}`, `{current_user_id}`,
  `{user_email}`, `{user_person}`. Exception: `type: "form_answer"` binds the
  person itself (performs link + `created_email`), so the declaration is not
  required there.
- `type: "update"` with a `result_scope_id` **requires** a non-empty
  `action.props`. Without the list the boundary of the bridge would be
  "everything on this object", which is exactly what the bridge must not be.

## 4. Where the bridge is *not* the answer

| Situation | Better tool |
|---|---|
| staff must **read** data from another scope | a separate view in the team scope that reads the planning scope server-side, or a copy written by a trigger — never a read grant |
| a whole object has to move between scopes | an action of type `update`/`create` with `result_scope_id`, or an explicit move endpoint; not a bridge hack |
| several objects change at once | one action per object; the bridge is bound to **one** UDID |
| notifications after the change | `notify` on the action or a trigger on the property — the bridge says nothing about messaging |

## 5. Worked example — swap request

Team scope `..._team`:

```json
{
  "action": {
    "type": "update",
    "target": "{linked:arbeitet_in:...:udid}",
    "result_scope_id": 80,
    "props": ["tausch_status", "tausch_partner"],
    "params": { "person": "{current_user_email}" }
  },
  "guards": [ { "key": "linked_to_user", "arg": "person" } ]
}
```

What happens: the member clicks in the team scope; the guard proves the object
belongs to that person; the bridge opens for exactly this shift in scope 80 and
exactly these two properties; the write succeeds; the bridge closes; the
planning scope still shows no write grant for the member.

## 6. Pitfalls we paid for

1. **UDID case.** Lower-case hex everywhere. A bridge set with a mixed-case UDID
   never matches — and nothing reports it.
2. **Bridge and link removal.** Removing a link is not a property write; it has
   its own path (`remove: "pair"`, event `link_removed`). Check both when the
   action is supposed to detach something.
3. **`type: "create"`** needs the person binding in the body, because no
   performs link is created automatically.
4. **Guards first.** `linked_to_user` (implemented in `base.php`) proves "only my
   own row". Without it the bridge writes into a foreign object as soon as
   somebody manipulates the URL.
5. **Costs.** An action costs 20 credits and the **executing** account pays. A
   bridge does not change who pays — a member without credits gets HTTP 402
   before anything happens.

## 7. Test recipe (no browser needed)

1. Manager account: create the definition with `onto_actiondef_save`, then read
   it back (`onto_actiondef_list`) — the saved body is the truth.
2. Member account: execute once with `onto_object_execute_action`.
3. Read the target object **as manager** and confirm the property changed in the
   planning scope.
4. Read it **as the member** and confirm the member still cannot see the
   planning scope.
5. Negative test: same call with a foreign object id → must fail with the guard
   error, not with a silent success.
