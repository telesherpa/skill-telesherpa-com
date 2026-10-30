# Tool Catalog

This overview is generated from the service's tool list: **192 tools** (server 2.3.0), of which
**124** are visible to an account with
`login`+`public`+`member`, **192** with `login`+`public`+`admin`, **6** without a token. An account with
`admin` **and** `member` sees all **192**.

**As of 2026-10-30.** For your own account, `auth_status` (`tools_visible`) is what counts — that is the
number that applies there.

**The roles column says who SEES the tool.** It does not protect execution — the
actual safeguard is the write permission on the scope (`can_write`, `can_manage_scope`).
What is authoritative for your own account remains `tools/list` with your own token; the service verifies
the permission on every call.

**Careful: `admin` is NOT a superset of `member`.** Some of the tools are visible **only** to
`member`, others **only** to `admin`; only some carry both roles. An
admin account therefore does **not** see `change_password`, `onto_catalog_apply` or `onto_actiondef_save`,
because only `member` is listed there. Anyone reporting „mir fehlen Tools" after a role change
must check **both** numbers — not assume that admin sees everything.

## Without a token (public, 6 tools)

`register`, `activate`, `login`, `refresh_access_token`, `auth_status`, `onto_credit_pricing`

`tools/list` without a token returns **HTTP 401** instead of a list — the six can be invoked by
**name** (the `www-authenticate` header names them). Without a token these six are also
**executable**; all others answer without a token with 401, even if they would be visible.

**What this means for the client:** a client that must first complete `initialize` and `tools/list`
successfully — the usual SDKs, desktop clients, Hermes too — gets **no**
tool catalog at all without a token. The first call must therefore know the six names. A tool that
is not in the catalog does not exist for the agent, even if the server would execute it.

### A role list is silent — in both directions

Two cases that produce **no** error message and are therefore easily overlooked:

- An **empty** role list does **not** mean „nobody", but „everyone": a tool that
  should be restricted becomes visible to every account with a valid token.
- A role list with **exclusively invalid** names makes a tool visible to **nobody**
  — likewise silently.

In both cases the tool **list** looks unremarkable for an authorized account, and the
code reads correctly. A tool nevertheless remains executable only with the write permissions on the
scope — the actual lock is there, not in the role list.

**The lesson for every role change:** call `auth_status` and compare `tools_visible` against the
expected **count**. One number exposes both silent cases.

### Legacy tools with dead targets

Three report tools of the first world pointed at **deleted paths** (all 404) and carried
**exclusively** a role name that does not exist — i.e. visible to nobody. They have been
removed; the functionality is fully covered by the `onto_report_*` tools (9 tools:
6 read-only for `admin`+`member`, 3 write for `admin` only).

**The lesson:** a tool that points at a 404 path AND is unreachable for anyone is
dead weight — it bloats the catalog and costs in the tool quality rating (tool count).
Check both together, not just visibility.

## Base tools

| Tool | Roles | Purpose |
|---|---|---|
| `auth_status` | public | Diagnosis: token valid? which roles? what to do? |
| `login` | public | client_id + client_secret → bearer token |
| `refresh_access_token` | public | Renew token (no password) |
| `register` | public | Create account (human: `username`+voucher · agent: `invite_code`) |
| `activate` | public | Redeem voucher (e-mail path only; omitted for an invitation code) |
| `onto_credit_pricing` | public | Read prices |
| `get_image` | `member` | Download image/file |
| `get_document` | `member` | Download document |
| `upload_file_to_object` | `member` | File/image to an object (also `structuredimage`) |
| `upload_staging_csv` | `member` | CSV into the import staging store |
| `upload_code_allowlist` | `member`, `admin` | XLSX/XLS bulk of the e-mail allowlist of an invite code |
| `change_password` | `member` | Change password |
| `change_profile` | `member` | Change profile (without arguments: **read** profile) |
| `get_profile` | `member` | Fetch own profile purely read-only — no write path |
| `renew_account` | `member` | Renew account |
| `get_notifications` | `member` | Notifications |

## Tool catalog of the ontology endpoints

### `onto_object_*`

| Tool | Roles |
|---|---|
| `onto_object_addcomment` | `member` |
| `onto_object_bulk_action` | `member` |
| `onto_object_create` | `member` |
| `onto_object_create_link` | `member` |
| `onto_object_delete` | `member` |
| `onto_object_delete_link` | `member` |
| `onto_object_delete_property` | `member` |
| `onto_object_execute_action` | `member` |
| `onto_object_geocode` | `member` |
| `onto_object_graph` | `member` |
| `onto_object_index` | `member` |
| `onto_object_search` | `member` |
| `onto_object_search_scopes` | `admin` |
| `onto_object_set_property` | `member` |
| `onto_object_show` | `member` |
| `onto_object_update` | `member` |
| `onto_object_update_link` | `member` |

### `onto_type_*`

| Tool | Roles |
|---|---|
| `onto_type_create` | `admin` |
| `onto_type_delete` | `admin` |
| `onto_type_index` | `admin` |
| `onto_type_show` | `member` |
| `onto_type_update` | `admin` |

### `onto_link_*`

| Tool | Roles |
|---|---|
| `onto_link_create_def` | `admin` |
| `onto_link_delete_def` | `admin` |
| `onto_link_index` | `admin` |
| `onto_link_update_def` | `admin` |

### `onto_property_def_*`

| Tool | Roles |
|---|---|
| `onto_property_def_create` | `admin`, `member` |
| `onto_property_def_delete` | `admin` |
| `onto_property_def_index` | `admin` |
| `onto_property_def_delete_override` | `admin`, `member` |
| `onto_property_def_update` | `admin`, `member` |

### `onto_property_set_*`

| Tool | Roles |
|---|---|
| `onto_property_set_create` | `admin`, `member` |
| `onto_property_set_delete` | `admin` |
| `onto_property_set_index` | `admin` |
| `onto_property_set_update` | `admin`, `member` |

### `onto_interface_*`

| Tool | Roles |
|---|---|
| `onto_interface_create_def` | `admin` |
| `onto_interface_delete_def` | `admin` |
| `onto_interface_index` | `admin` |
| `onto_interface_map_markers` | `member` |
| `onto_interface_show` | `member` |
| `onto_interface_update_def` | `admin` |

### `onto_actiondef_*`

| Tool | Roles |
|---|---|
| `onto_actiondef_delete` | `member` |
| `onto_actiondef_index` | `admin` |
| `onto_actiondef_list` | `member` |
| `onto_actiondef_save` | `member` |

### `onto_scope_*`

| Tool | Roles |
|---|---|
| `onto_scope_assign_profile` | `admin` |
| `onto_scope_create` | `admin` |
| `onto_scope_delete` | `admin` |
| `onto_scope_index` | `admin` |
| `onto_scope_remove_profile` | `admin` |
| `onto_scope_save_structuredimage_categories` | `member` |
| `onto_scope_search` | `admin` |
| `onto_scope_search_profiles` | `admin` |
| `onto_scope_toggle_scope_flag` | `admin` |
| `onto_scope_structuredimage_categories` | `member` |
| `onto_scope_update` | `admin` |

### `onto_settings_*`

| Tool | Roles |
|---|---|
| `onto_settings_create` | `admin` |
| `onto_settings_delete` | `admin` |
| `onto_settings_index` | `admin` |
| `onto_settings_list` | `admin` |
| `onto_settings_update` | `admin` |

### `onto_invite_code_*`

| Tool | Roles |
|---|---|
| `onto_invite_code_index` | `admin` |
| `onto_invite_code_show` | `admin` |

> Read-only oversight across **all** scopes: code, scope, template, kind, R/W/M, issuer, status,
> expiry, usage, redemptions. Filters travel in the JSON body (`q`, `scope_id`, `status`, `kind`,
> `sort`, `dir`, `page`) — never in the query string. Creating and revoking a code stays with the
> scope manager in `/ontouser/scope`; this view only looks.

### `onto_user_*`

| Tool | Roles |
|---|---|
| `onto_user_delete` | `admin` |
| `onto_user_delete_profile` | `admin` |
| `onto_user_get` | `admin` |
| `onto_user_index` | `admin` |
| `onto_user_show` | `admin` |
| `onto_user_update` | `admin` |
| `onto_user_update_profile` | `admin` |

> **`onto_user_delete`** clears everything for the account in ONE call: tokens, passkeys, sessions,
> trackings, roles, scope assignments, favorites, memo lines, own notifications,
> profile and account — and then counts up afterwards. `id` may be the account ID, the profile ID **or** the
> `email`. The response names `entfernt` and `behalten`
> (accounting, log, other people's data). `onto_user_delete_profile` is the same path
> (legacy name). Not found = HTTP 404, no silent success.

### `onto_report_*`

| Tool | Roles |
|---|---|
| `onto_report_aggregate` | `admin`, `member` |
| `onto_report_copy` | `admin` |
| `onto_report_csv` | `admin`, `member` |
| `onto_report_delete` | `admin` |
| `onto_report_index` | `admin`, `member` |
| `onto_report_json` | `admin`, `member` |
| `onto_report_map_data` | `admin`, `member` |
| `onto_report_save` | `admin` |
| `onto_report_show` | `admin`, `member` |

### `onto_catalog_*`

| Tool | Roles |
|---|---|
| `onto_catalog_apply` | `member` |
| `onto_catalog_clear` | `member` |
| `onto_catalog_index` | `member` |
| `onto_catalog_presets` | `member` |

### `onto_i18n_*`

| Tool | Roles |
|---|---|
| `onto_i18n_create` | `admin` |
| `onto_i18n_delete` | `admin` |
| `onto_i18n_export` | `admin` |
| `onto_i18n_index` | `admin` |
| `onto_i18n_matrix` | `admin` |
| `onto_i18n_missing` | `admin` |
| `onto_i18n_review` | `admin` |
| `onto_i18n_suggestions` | `admin` |
| `onto_i18n_translate` | `admin` |

### `onto_favorite_*`

| Tool | Roles |
|---|---|
| `onto_favorite_add` | `member` |
| `onto_favorite_delete` | `member` |
| `onto_favorite_index` | `member` |
| `onto_favorite_toggle` | `member` |

### `onto_file_*`

| Tool | Roles |
|---|---|
| `onto_file_delete` | `member` |
| `onto_file_structured` | `member` |

### `onto_series_*` (sensor time series)

| Tool | Roles |
|---|---|
| `onto_series_range` | `member` |
| `onto_series_last` | `member` |

A series has **no udid of its own** — its identity is the pair
`(object_udid, series_key)`. The `series_key` is the same string as the key of the associated
property; the property is the **pointer** to the series (def field `computed = {"series": true}`),
its value does not carry it. Which series an object has is named by `onto_object_show` in the field
`series_keys`.

Visibility follows the **scope of the property**, not of the object (fail-closed) —
a foreign scope returns 403 even though the tool is visible.

**All timestamps are UTC** (ISO 8601 with `Z`, e.g. `2026-10-06T23:09:00Z`); this applies to
`points[].ts`, `series_ts` **and** to the `from`/`to` parameters. Details and use cases:
`references/features/sensor-zeitreihen.md`.

### `onto_import_*`

| Tool | Roles |
|---|---|
| `onto_import_analyze` | `admin` |
| `onto_import_analyze_step` | `admin` |
| `onto_import_create_source` | `admin` |
| `onto_import_delete_source` | `admin` |
| `onto_import_delete_step` | `admin` |
| `onto_import_index` | `admin` |
| `onto_import_reorder_step` | `admin` |
| `onto_import_run_step` | `admin` |
| `onto_import_save_step` | `admin` |
| `onto_import_sources` | `admin` |
| `onto_import_step_cancel` | `admin` |
| `onto_import_step_status` | `admin` |
| `onto_import_steps` | `admin` |
| `onto_import_update_source` | `admin` |

### `onto_trigger_*`

| Tool | Roles |
|---|---|
| `onto_trigger_create` | `member` |
| `onto_trigger_delete` | `member` |
| `onto_trigger_index` | `admin`, `member` |
| `onto_trigger_update` | `member` |

### `onto_function_*`

| Tool | Roles |
|---|---|
| `onto_function_create` | `member` |
| `onto_function_delete` | `member` |
| `onto_function_index` | `admin`, `member` |
| `onto_function_run` | `admin`, `member` |
| `onto_function_update` | `member` |

### `onto_credit_*`

| Tool | Roles |
|---|---|
| `onto_credit_balance` | `admin`, `member` |
| `onto_credit_chargeup` | `admin`, `member` |
| `onto_credit_checkout` | `admin`, `member` |
| `onto_credit_create_prepaid` | `admin` |
| `onto_credit_receipt` | `admin`, `member` |

### `onto_watch_*`

| Tool | Roles |
|---|---|
| `onto_watch_changes` | `member` |
| `onto_watch_cursor` | `member` |

### `onto_action_*`

| Tool | Roles |
|---|---|
| `onto_action_logs` | `admin` |

### `ontouser_scope_*`

| Tool | Roles |
|---|---|
| `ontouser_scope_assign_public` | `member`, `admin` |
| `ontouser_scope_create` | `member`, `admin` |
| `ontouser_scope_index` | `member`, `admin` |
| `ontouser_scope_list_public` | `member`, `admin` |
| `ontouser_scope_remove_public` | `member`, `admin` |
| `ontouser_scope_set_visibility` | `member`, `admin` |

### `ontouser_team_*` — managed accounts

The manager view of an account that is managed by a company scope (`profiles.managed_by_scope_id`).
Every tool needs `can_manage_scope` in the scope that carries the marker, plus the respective right.

| Tool | Roles |
|---|---|
| `ontouser_team_index` | `member`, `admin` |
| `ontouser_team_show` | `member`, `admin` |
| `ontouser_team_suspend` | `member`, `admin` |
| `ontouser_team_unsuspend` | `member`, `admin` |
| `ontouser_team_update_profile` | `member`, `admin` |
| `ontouser_team_set_scope_rights` | `member`, `admin` |
| `ontouser_team_set_write_scope` | `member`, `admin` |
| `ontouser_team_credits` | `member`, `admin` |
| `ontouser_team_transfer` | `member`, `admin` |
| `ontouser_team_offboard_preview` | `member`, `admin` |
| `ontouser_team_offboard` | `member`, `admin` |

Notes an agent must not guess:

- **The marker grants nothing.** `managed_by_scope_id` only says who leads the account; every call
  still checks `can_manage_scope` and the specific right.
- **Only your own managed accounts.** Any other profile answers `403`. The manager himself never
  appears in his own list (he carries no marker).
- **`ontouser_team_transfer` books from the caller's own balance** — there is no `from_profile_id`.
  `reason` and `idempotency_key` are mandatory; a repeated key returns `schon_gebucht: true`
  instead of booking twice. Insufficient balance answers `402`.
- **`ontouser_team_set_scope_rights` sets, it does not only escalate.** All three flags `0` removes
  the membership. `can_manage_scope` on the managing scope itself is rejected (`403`) — that needs
  an invite code with `can_manage_scope`.
- **`ontouser_team_suspend`** removes `member` and deletes all tokens; a suspended account gets
  `403 account_suspended` from the token endpoint, so the reason is visible.
- **`ontouser_team_offboard`** defaults to `mode=delete`; `release` keeps the account alive and
  notifies the managers of foreign scopes. Remaining credits go back to the manager unless
  `keep_balance` is set. The last manager of a scope cannot be deleted, and a scope that carries
  managed accounts cannot be deleted (not even with `purge`).
- **Invite code type `revoke`** takes rights away (`can_read`/`can_write`/`can_manage_scope` on the
  code say which; all zero removes the membership). Grant codes only escalate — revoking is always a
  separate, explicit code.

### `ontouser_interface_*`

| Tool | Roles |
|---|---|
| `ontouser_interface_map_markers` | `member` |
| `ontouser_interface_show` | `member` |

### `ontouser_notification_*`

| Tool | Roles |
|---|---|
| `ontouser_notification_index` | `member`, `admin` |
| `ontouser_notification_read` | `member`, `admin` |
| `ontouser_notification_send` | `member`, `admin` |

`ontouser_notification_send` writes into the inbox of a **different** account: `to` is the e-mail
of an existing account (own address allowed), `text` the message (over 4000 characters is
truncated). Response `{sent: true, to: …}`. The message runs through the same system as
service request replies (`type=user_message`) — platform inbox plus e-mail/push, **as far as the
recipient receives it**: at agent addresses the mail barrier prevents mail delivery, there it stays
with the message in the inbox (`ontouser_notification_index`).

### `ontouser_object_*`

| Tool | Roles |
|---|---|
| `ontouser_object_show` | `member` |

### `ontouser_service_*`

| Tool | Roles |
|---|---|
| `ontouser_service_request` | `member`, `admin` |

### Forms & invite/members (without name prefix)

| Tool | Roles |
|---|---|
| `add_code_allowlist_entry` | `member`, `admin` |
| `create_invite_code` | `member`, `admin` |
| `delete_code_allowlist_entry` | `member`, `admin` |
| `list_code_allowlist` | `member`, `admin` |
| `list_formdefs` | `member` |
| `list_invite_codes` | `member`, `admin` |
| `list_members` | `member`, `admin` |
| `onto_formdef_create` | `member` |
| `onto_formdef_list_templates` | `member` |
| `onto_formdef_save` | `member` |
| `prepare_form_answer` | `member` |
| `redeem_invite_code` | `member`, `admin` |
| `remove_member` | `member`, `admin` |
| `resolve_form_answer` | `member` |
| `revoke_invite_code` | `member`, `admin` |
| `show_form_answer` | `member` |
| `show_formdef` | `member` |
| `submit_form_answer` | `member` |
| `toggle_member_flag` | `member`, `admin` |

## Timestamps — always UTC

**Every timestamp this service returns is UTC in ISO 8601 with `Z`** — e.g.
`2026-10-06T23:09:00Z`. The value carries its zone itself; you need no conversion and
must not assume one.

- **Do not convert yourself.** Anyone converting UTC to a local time in order to compare two values
  builds in the very error the conversion was supposed to avoid. Compare UTC with UTC.
- **What you send is also UTC.** `from`/`to` on `onto_series_range` are read as UTC
  — as `2026-10-06T23:09:00Z` or `2026-10-06 23:09:00`. An appended offset
  (`+02:00`) is valid and is converted.
- **The web view, by contrast, shows viewer time.** If the interface shows `01:09` and the API
  `23:09`, both are correct — that is a 2-hour zone difference, not an error.
- **Compute a duration** always from two UTC values from **the same** source.

## Pitfalls

1. **A missing tool is usually a permission problem, not a bug.** Visibility is filtered per request
   against the roles. Call `auth_status` first — the response names the roles in
   plain text.
2. **Tool names end in `_create`/`_update`/`_delete`, NOT `_save`** (`onto_trigger_*`,
   `onto_function_*`). Anyone searching for `_save` falsely reports „Tool fehlt".
3. **Two name families for the same thing**: `onto_scope_*` is the admin view
   (`onto_scope_create` is `admin`), `ontouser_scope_*` the self-service one. For your own
   scopes use the `ontouser_*` family — both paths bill and count against
   `profiles.max_scopes`.
4. **The order of tools in the catalog is not a priority.** The list is grouped by resource;
   choosing the right path is described in `references/features/`.
5. **A wrong role assignment is silent.** If a tool becomes visible to more accounts than
   intended or invisible to all, there is no error message and no log entry.
   The check is the number from `auth_status` (`tools_visible`) — after every permission change
   compare it against the target value.
6. **Parameters are documented in the tool schemas, not here.** When unsure,
   look at `tools/list` for the specific tool — the `description` contains the verified
   body format and the required fields.
7. **Your own number comes from `auth_status`.** `tools_visible` applies to the logged-in
   account; after every permission change compare it anew instead of deriving it from this catalog.
8. **`admin` does NOT see everything.** Some tools carry exclusively `member`, others
   exclusively `admin`; only some both. An admin account that misses `change_password`
   or `onto_catalog_apply` has no error — the role list there names only
   `member`. When checking, always compare **both** numbers (`auth_status` with the
   respective account).

## Tool descriptions and annotations

Every tool has a **descriptive description** (verb + resource + when/when not + named
alternative) and **MCP annotations** (`readOnlyHint`, `destructiveHint`, `idempotentHint`,
`openWorldHint`). A tool without a description and without annotations is hardly
selectable for a model — both are part of it.

The descriptions are maintained centrally and are read in at startup. They can therefore be
changed without touching the tool definitions.
