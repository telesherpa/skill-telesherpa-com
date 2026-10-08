# Tool-Katalog

Diese Übersicht ist aus der Werkzeugliste des Dienstes erzeugt: **174 Tools**, davon
**101** sichtbar für ein Konto mit
`login`+`public`+`telesherpa`, **104** mit `login`+`public`+`admin`, **6** ohne Token.

**Gemessen, nicht geschätzt:** die Zahlen stammen aus der Rollen-Zuordnung des Dienstes
(`TOOL_ROLES`, 174 Einträge) und decken sich mit dem, was `/health` als `tools` meldet. Eine
frühere Fassung dieses Katalogs nannte 172/100/110 — sie war aus zwei Quellen falsch
zusammengeführt. Wer die Zahl stellt, nimmt `auth_status` (`tools_visible`).

**Die Rollen-Spalte sagt, wer das Tool SIEHT.** Sie schützt nicht die Ausführung — die
eigentliche Absicherung ist das Schreibrecht am Scope (`can_write`, `can_manage_scope`).
Die Rollenzuordnung selbst ist nicht Teil der veröffentlichten Schnittstelle.

**Achtung: `admin` ist KEINE Obermenge von `telesherpa`.** Ein Teil der Tools ist **nur** für
`telesherpa` sichtbar, ein anderer **nur** für `admin`; nur ein Teil trägt beide Rollen. Ein
Admin-Konto sieht also `change_password`, `onto_catalog_apply` oder `onto_actiondef_save`
**nicht**, weil dort ausschließlich `telesherpa` steht. Wer nach einer Rollenänderung „mir fehlen
Tools" meldet, muss **beide** Zahlen prüfen — nicht annehmen, admin sehe alles.

## Ohne Token (öffentlich, 6 Tools)

`register`, `activate`, `login`, `refresh_access_token`, `auth_status`, `onto_credit_pricing`

`tools/list` ohne Token liefert **HTTP 401** statt einer Liste — die sechs sind über den
**Namen** aufrufbar (der `www-authenticate`-Header nennt sie). Ohne Token sind diese sechs auch
**ausführbar**; alle anderen antworten ohne Token mit 401, auch wenn sie sichtbar wären.

### Eine Rollenliste ist stumm — in beide Richtungen

Zwei Fälle, die **keine** Fehlermeldung erzeugen und deshalb leicht übersehen werden:

- Eine **leere** Rollenliste bedeutet **nicht** „niemand", sondern „alle": ein Werkzeug, das
  beschränkt sein sollte, wird für jedes Konto mit gültigem Token sichtbar.
- Eine Rollenliste mit **ausschließlich ungültigen** Namen macht ein Tool für **niemanden**
  sichtbar — ebenfalls lautlos.

In beiden Fällen sieht die Tool-**Liste** für ein berechtigtes Konto unauffällig aus, und der
Code liest sich korrekt. Ausführbar bleibt ein Tool trotzdem nur mit den Schreibrechten am
Scope — die eigentliche Sperre liegt dort, nicht in der Rollenliste.

**Die Lehre für jede Rollenänderung:** `auth_status` aufrufen und `tools_visible` gegen die
erwartete **Anzahl** stellen. Eine Zahl deckt beide stillen Fälle auf.

### Altweg-Tools mit toten Zielen

Drei Report-Tools der ersten Welt zeigten auf **gelöschte Pfade** (alle 404) und trugen
**ausschließlich** einen Rollennamen, den es nicht gibt — also für niemanden sichtbar. Sie sind
entfernt; die Funktion ist vollständig durch die `onto_report_*`-Tools abgedeckt (9 Tools:
6 lesend für `admin`+`telesherpa`, 3 schreibend nur `admin`).

**Die Lehre:** ein Tool, das auf einen 404-Pfad zeigt UND für niemanden erreichbar ist, ist
toter Ballast — es bläht den Katalog auf und kostet in der Tool-Qualitätsbewertung (Tool-Anzahl).
Beides zusammen prüfen, nicht nur die Sichtbarkeit.

## Basis-Tools

| Tool | Rollen | Zweck |
|---|---|---|
| `auth_status` | öffentlich | Diagnose: Token gültig? welche Rollen? was tun? |
| `login` | öffentlich | client_id + client_secret → Bearer-Token |
| `refresh_access_token` | öffentlich | Token erneuern (kein Passwort) |
| `register` | öffentlich | Account anlegen |
| `activate` | öffentlich | Voucher einlösen |
| `onto_credit_pricing` | öffentlich | Preise lesen |
| `get_image` | `telesherpa` | Bild/Datei herunterladen |
| `get_document` | `telesherpa` | Dokument herunterladen |
| `upload_file_to_object` | `telesherpa` | Datei/Bild an ein Objekt (auch `structuredimage`) |
| `upload_staging_csv` | `telesherpa` | CSV in die Import-Staging-Ablage |
| `upload_code_allowlist` | `telesherpa`, `admin` | XLSX/XLS-Bulk der E-Mail-Allowlist eines Invite-Codes |
| `change_password` | `telesherpa` | Passwort ändern |
| `change_profile` | `telesherpa` | Profil ändern |
| `renew_account` | `telesherpa` | Konto verlängern |
| `get_notifications` | `telesherpa` | Benachrichtigungen |

## Tool-Katalog der Ontologie-Endpunkte

### `onto_object_*`

| Tool | Rollen |
|---|---|
| `onto_object_addcomment` | `telesherpa` |
| `onto_object_bulk_action` | `telesherpa` |
| `onto_object_create` | `telesherpa` |
| `onto_object_create_link` | `telesherpa` |
| `onto_object_delete` | `telesherpa` |
| `onto_object_delete_link` | `telesherpa` |
| `onto_object_delete_property` | `telesherpa` |
| `onto_object_execute_action` | `telesherpa` |
| `onto_object_geocode` | `telesherpa` |
| `onto_object_graph` | `telesherpa` |
| `onto_object_index` | `telesherpa` |
| `onto_object_search` | `telesherpa` |
| `onto_object_search_scopes` | `admin` |
| `onto_object_set_property` | `telesherpa` |
| `onto_object_show` | `telesherpa` |
| `onto_object_update` | `telesherpa` |
| `onto_object_update_link` | `telesherpa` |

### `onto_type_*`

| Tool | Rollen |
|---|---|
| `onto_type_create` | `admin` |
| `onto_type_delete` | `admin` |
| `onto_type_index` | `admin` |
| `onto_type_show` | `telesherpa` |
| `onto_type_update` | `admin` |

### `onto_link_*`

| Tool | Rollen |
|---|---|
| `onto_link_create_def` | `admin` |
| `onto_link_delete_def` | `admin` |
| `onto_link_index` | `admin` |
| `onto_link_update_def` | `admin` |

### `onto_property_def_*`

| Tool | Rollen |
|---|---|
| `onto_property_def_create` | `admin`, `telesherpa` |
| `onto_property_def_delete` | `admin` |
| `onto_property_def_index` | `admin` |
| `onto_property_def_delete_override` | `admin`, `telesherpa` |
| `onto_property_def_update` | `admin`, `telesherpa` |

### `onto_property_set_*`

| Tool | Rollen |
|---|---|
| `onto_property_set_create` | `admin`, `telesherpa` |
| `onto_property_set_delete` | `admin` |
| `onto_property_set_index` | `admin` |
| `onto_property_set_update` | `admin`, `telesherpa` |

### `onto_interface_*`

| Tool | Rollen |
|---|---|
| `onto_interface_create_def` | `admin` |
| `onto_interface_delete_def` | `admin` |
| `onto_interface_index` | `admin` |
| `onto_interface_map_markers` | `telesherpa` |
| `onto_interface_show` | `telesherpa` |
| `onto_interface_update_def` | `admin` |

### `onto_actiondef_*`

| Tool | Rollen |
|---|---|
| `onto_actiondef_delete` | `telesherpa` |
| `onto_actiondef_index` | `admin` |
| `onto_actiondef_list` | `telesherpa` |
| `onto_actiondef_save` | `telesherpa` |

### `onto_scope_*`

| Tool | Rollen |
|---|---|
| `onto_scope_assign_profile` | `admin` |
| `onto_scope_create` | `admin` |
| `onto_scope_delete` | `admin` |
| `onto_scope_index` | `admin` |
| `onto_scope_remove_profile` | `admin` |
| `onto_scope_save_structuredimage_categories` | `telesherpa` |
| `onto_scope_search` | `admin` |
| `onto_scope_search_profiles` | `admin` |
| `onto_scope_toggle_scope_flag` | `admin` |
| `onto_scope_structuredimage_categories` | `telesherpa` |
| `onto_scope_update` | `admin` |

### `onto_settings_*`

| Tool | Rollen |
|---|---|
| `onto_settings_create` | `admin` |
| `onto_settings_delete` | `admin` |
| `onto_settings_index` | `admin` |
| `onto_settings_list` | `admin` |
| `onto_settings_update` | `admin` |

### `onto_user_*`

| Tool | Rollen |
|---|---|
| `onto_user_delete` | `admin` |
| `onto_user_delete_profile` | `admin` |
| `onto_user_get` | `admin` |
| `onto_user_index` | `admin` |
| `onto_user_show` | `admin` |
| `onto_user_update` | `admin` |
| `onto_user_update_profile` | `admin` |

### `onto_report_*`

| Tool | Rollen |
|---|---|
| `onto_report_aggregate` | `admin`, `telesherpa` |
| `onto_report_copy` | `admin` |
| `onto_report_csv` | `admin`, `telesherpa` |
| `onto_report_delete` | `admin` |
| `onto_report_index` | `admin`, `telesherpa` |
| `onto_report_json` | `admin`, `telesherpa` |
| `onto_report_map_data` | `admin`, `telesherpa` |
| `onto_report_save` | `admin` |
| `onto_report_show` | `admin`, `telesherpa` |

### `onto_catalog_*`

| Tool | Rollen |
|---|---|
| `onto_catalog_apply` | `telesherpa` |
| `onto_catalog_clear` | `telesherpa` |
| `onto_catalog_index` | `telesherpa` |
| `onto_catalog_presets` | `telesherpa` |

### `onto_i18n_*`

| Tool | Rollen |
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

| Tool | Rollen |
|---|---|
| `onto_favorite_add` | `telesherpa` |
| `onto_favorite_delete` | `telesherpa` |
| `onto_favorite_index` | `telesherpa` |
| `onto_favorite_toggle` | `telesherpa` |

### `onto_file_*`

| Tool | Rollen |
|---|---|
| `onto_file_delete` | `telesherpa` |
| `onto_file_structured` | `telesherpa` |

### `onto_series_*` (Sensor-Zeitreihen)

| Tool | Rollen |
|---|---|
| `onto_series_range` | `telesherpa` |
| `onto_series_last` | `telesherpa` |

Eine Reihe hat **keine eigene udid** — ihre Identität ist das Paar
`(object_udid, series_key)`. Der `series_key` ist derselbe String wie der Key des zugehörigen
Property; das Property ist der **Zeiger** auf die Reihe (Def-Feld `computed = {"series": true}`),
sein Wert trägt es nicht. Welche Reihen ein Objekt hat, nennt `onto_object_show` im Feld
`series_keys`.

Sichtbarkeit richtet sich nach dem **Scope des Property**, nicht des Objekts (fail-closed) —
ein fremder Scope liefert 403, obwohl das Tool sichtbar ist.

**Alle Zeitangaben sind UTC** (ISO 8601 mit `Z`, z. B. `2026-10-06T23:09:00Z`); das gilt für
`points[].ts`, `series_ts` **und** für die Parameter `from`/`to`. Details und Use-Cases:
`references/features/sensor-zeitreihen.md`.

### `onto_import_*`

| Tool | Rollen |
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

| Tool | Rollen |
|---|---|
| `onto_trigger_create` | `telesherpa` |
| `onto_trigger_delete` | `telesherpa` |
| `onto_trigger_index` | `admin`, `telesherpa` |
| `onto_trigger_update` | `telesherpa` |

### `onto_function_*`

| Tool | Rollen |
|---|---|
| `onto_function_create` | `telesherpa` |
| `onto_function_delete` | `telesherpa` |
| `onto_function_index` | `admin`, `telesherpa` |
| `onto_function_run` | `admin`, `telesherpa` |
| `onto_function_update` | `telesherpa` |

### `onto_credit_*`

| Tool | Rollen |
|---|---|
| `onto_credit_balance` | `admin`, `telesherpa` |
| `onto_credit_chargeup` | `admin`, `telesherpa` |
| `onto_credit_checkout` | `admin`, `telesherpa` |
| `onto_credit_create_prepaid` | `admin` |

### `onto_watch_*`

| Tool | Rollen |
|---|---|
| `onto_watch_changes` | `telesherpa` |
| `onto_watch_cursor` | `telesherpa` |

### `onto_action_*`

| Tool | Rollen |
|---|---|
| `onto_action_logs` | `admin` |

### `ontouser_scope_*`

| Tool | Rollen |
|---|---|
| `ontouser_scope_assign_public` | `telesherpa`, `admin` |
| `ontouser_scope_create` | `telesherpa`, `admin` |
| `ontouser_scope_index` | `telesherpa`, `admin` |
| `ontouser_scope_list_public` | `telesherpa`, `admin` |
| `ontouser_scope_remove_public` | `telesherpa`, `admin` |
| `ontouser_scope_set_visibility` | `telesherpa`, `admin` |

### `ontouser_interface_*`

| Tool | Rollen |
|---|---|
| `ontouser_interface_map_markers` | `telesherpa` |
| `ontouser_interface_show` | `telesherpa` |

### `ontouser_notification_*`

| Tool | Rollen |
|---|---|
| `ontouser_notification_index` | `telesherpa`, `admin` |
| `ontouser_notification_read` | `telesherpa`, `admin` |

### `ontouser_object_*`

| Tool | Rollen |
|---|---|
| `ontouser_object_show` | `telesherpa` |

### `ontouser_service_*`

| Tool | Rollen |
|---|---|
| `ontouser_service_request` | `telesherpa`, `admin` |

### Formular & Invite/Mitglieder (ohne Namenspräfix)

| Tool | Rollen |
|---|---|
| `add_code_allowlist_entry` | `telesherpa`, `admin` |
| `create_invite_code` | `telesherpa`, `admin` |
| `delete_code_allowlist_entry` | `telesherpa`, `admin` |
| `list_code_allowlist` | `telesherpa`, `admin` |
| `list_formdefs` | `telesherpa` |
| `list_invite_codes` | `telesherpa`, `admin` |
| `list_members` | `telesherpa`, `admin` |
| `onto_formdef_create` | `telesherpa` |
| `onto_formdef_list_templates` | `telesherpa` |
| `onto_formdef_save` | `telesherpa` |
| `prepare_form_answer` | `telesherpa` |
| `redeem_invite_code` | `telesherpa`, `admin` |
| `remove_member` | `telesherpa`, `admin` |
| `resolve_form_answer` | `telesherpa` |
| `revoke_invite_code` | `telesherpa`, `admin` |
| `show_form_answer` | `telesherpa` |
| `show_formdef` | `telesherpa` |
| `submit_form_answer` | `telesherpa` |
| `toggle_member_flag` | `telesherpa`, `admin` |

## Zeitangaben — immer UTC

**Jeder Zeitstempel, den dieser Dienst zurückgibt, ist UTC in ISO 8601 mit `Z`** — z. B.
`2026-10-06T23:09:00Z`. Der Wert trägt seine Zone selbst; du brauchst keine Umrechnung und
darfst keine annehmen.

- **Nicht selbst umrechnen.** Wer UTC in eine Ortszeit umrechnet, um zwei Werte zu
  vergleichen, baut den Fehler ein, den die Umrechnung vermeiden sollte. Vergleiche UTC mit UTC.
- **Was du sendest, ist ebenfalls UTC.** `from`/`to` bei `onto_series_range` werden als UTC
  gelesen — als `2026-10-06T23:09:00Z` oder `2026-10-06 23:09:00`. Ein angehängter Offset
  (`+02:00`) gilt und wird umgerechnet.
- **Die Web-Ansicht zeigt dagegen Betrachterzeit.** Zeigt die Oberfläche `01:09` und die API
  `23:09`, sind beide richtig — das sind 2 Stunden Zonenunterschied, kein Fehler.
- **Eine Dauer** immer aus zwei UTC-Werten **derselben** Quelle rechnen.

## Pitfalls

1. **Ein fehlendes Tool ist meist ein Rechteproblem, kein Bug.** Sichtbarkeit wird pro Anfrage
   gegen die Rollen gefiltert. Erst `auth_status` aufrufen — die Antwort nennt die Rollen im
   Klartext.
2. **Tool-Namen enden auf `_create`/`_update`/`_delete`, NICHT `_save`** (`onto_trigger_*`,
   `onto_function_*`). Wer nach `_save` sucht, meldet fälschlich „Tool fehlt".
3. **Zwei Namensfamilien für dieselbe Sache**: `onto_scope_*` ist die Admin-Sicht,
   `ontouser_scope_*` die Selbstverwaltung (Agent-Scopes). Für eigene Scopes die
   `ontouser_*`-Familie verwenden — `onto_scope_create` ist `admin`.
4. **Reihenfolge der Tools im Katalog ist keine Priorität.** Die Liste ist nach Ressourcen
   gruppiert; die Wahl des richtigen Wegs steht in `references/features/`.
5. **Eine falsche Rollenzuordnung ist stumm.** Wird ein Werkzeug für mehr Konten sichtbar als
   beabsichtigt oder für alle unsichtbar, gibt es keine Fehlermeldung und keinen Log-Eintrag.
   Prüfmittel ist die Zahl aus `auth_status` (`tools_visible`) — nach jeder Rechteänderung
   gegen den Sollwert stellen.
6. **Parameter sind in den Tool-Schemata dokumentiert, nicht hier.** Bei Unsicherheit
   `tools/list` auf das konkrete Tool ansehen — die `description` enthält das verifizierte
   Body-Format und die Pflichtfelder.
7. **`admin` sieht NICHT alles.** Ein Teil der Tools trägt ausschließlich `telesherpa`, ein
   anderer ausschließlich `admin`; nur ein Teil beide. Ein Admin-Konto, das `change_password`
   oder `onto_catalog_apply` vermisst, hat keinen Fehler — die Rollenliste nennt dort nur
   `telesherpa`. Beim Prüfen immer **beide** Zahlen stellen (`auth_status` mit dem
   jeweiligen Konto).

## Tool-Beschreibungen und Annotations

Jedes Tool hat eine **sprechende Beschreibung** (Verb + Ressource + wann/wann nicht + genannte
Alternative) und **MCP-Annotations** (`readOnlyHint`, `destructiveHint`, `idempotentHint`,
`openWorldHint`). Ein Tool ohne Beschreibung und ohne Annotations ist für ein Modell kaum
auswählbar — beides gehört dazu.

Die Beschreibungen sind zentral gepflegt und werden beim Start eingelesen. Sie lassen sich
damit ändern, ohne die Tool-Definitionen anzufassen.
