# Notifications and link events

> Platform reference, 2026-11-06. Verified against `trigger.php`, `dispatch.php`
> and live runs (in-app notification plus e-mail, link-removal event).

## 1. Three channels, one decision

A notification reaches a person by in-app message, by e-mail, or — for agents — by
webhook. The rule "who gets told" is configuration, not code: a trigger or an
action carries a `notify` block, and the platform resolves the recipients.

## 2. Which placeholders are stable

| placeholder | meaning |
|---|---|
| `{object_name}` | name of the object the trigger fired on |
| `{link_type}` | key of the link type |
| `{link_value}` | value of the link |
| `{linked:<linktype>:<linkvalue>:<property>}` | a property of the linked object |
| `{removed_name}`, `{removed_email}` | the object/person that was detached |
| `{old:<field>}`, `{new:<field>}` | previous / new value, in the reader's time zone |
| `{monat}`, `{monat_start}`, `{monat_ende}` | month label and its bounds |
| `{now}` | current time |

`{now}` is used in production texts; its exact implementation was not re-checked for
this reference — verify it before publishing a skill that depends on it.

## 3. Link removal is its own event, not a property write

When a link disappears, `notify` with `event_type: "link_removed"` fires. It writes
**two rows per removal**:

- `value_old` = the object that was detached,
- `value_new` = the link value,
- `object_udid` = the context page, `key_name` = the link type.

`dispatch.php` filters notifications by `link_types`; a type missing there is a
notification that silently never arrives. Build the generic version — any object can
use it — instead of a special case per feature.

## 4. Triggers fire on `insert`

`waitRole` and every property a notification needs must be present **when the object
is created**. Patching them afterwards does not notify anybody, because the trigger
hangs on insert. This is the single most common "notification never arrived" cause.

## 5. Reaching another agent

A comment alone does **not** wake a peer agent. The reliable chain is:

1. write the comment (`onto_object_addcomment` with `parent`, `scope_id`, `text`),
2. send a short notification (`ontouser_notification_send`),
3. check the response for `"sent": true` — anything else means the recipient was not
   reached, whatever the first call said.

Without step 2 the message sits in the scope and nobody looks.

## 6. Costs

Notifications, triggers and the dispatch loop itself do not consume credits. The
writes that produce them do (see the price table in `references/features/tool-traps.md`).
A notification storm is therefore not expensive — a badly written trigger that
rewrites objects in a loop is.

## 7. Tone

One message, one fact, no narration. The reader is a human at work or an agent that
will act on it: name the object, state what changed, say what is expected. Do not
repeat the object description that the recipient can already see.

## 8. Test without waiting

1. Create the trigger with `onto_trigger_create`, read it back with `onto_trigger_index`.
2. Create the object that should fire it.
3. Wait one to five minutes (asynchronous by design), then check the recipient's
   notification list and the object's comments.
4. Negative test: create the object **without** the property the trigger needs and
   confirm nothing fires — otherwise you are testing "some notification appeared",
   not "this rule works".
