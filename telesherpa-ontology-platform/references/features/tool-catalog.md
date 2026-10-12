# Tool-Katalog

Diese Übersicht ist aus der Werkzeugliste des Dienstes erzeugt: **175 Tools**, davon
**108** sichtbar für ein Konto mit
`login`+`public`+`member`, **111** mit `login`+`public`+`admin`, **6** ohne Token. Ein Konto mit
`admin` **und** `member` sieht alle **175**.

**Stand 2026-10-11.** Für das eigene Konto zählt `auth_status` (`tools_visible`) — das ist die
Zahl, die dort gilt.

**Die Rollen-Spalte sagt, wer das Tool SIEHT.** Sie schützt nicht die Ausführung — die
eigentliche Absicherung ist das Schreibrecht am Scope (`can_write`, `can_manage_scope`).
Maßgeblich für das eigene Konto bleibt `tools/list` mit dem eigenen Token; der Dienst prüft
das Recht bei jedem Aufruf nach.

**Achtung: `admin` ist KEINE Obermenge von `member`.** Ein Teil der Tools ist **nur** für
`member` sichtbar, ein anderer **nur** für `admin`; nur ein Teil trägt beide Rollen. Ein
Admin-Konto sieht also `change_password`, `onto_catalog_apply` oder `onto_actiondef_save`
**nicht**, weil dort ausschließlich `member` steht. Wer nach einer Rollenänderung „mir fehlen
Tools" meldet, muss **beide** Zahlen prüfen — nicht annehmen, admin sehe alles.

## Ohne Token (öffentlich, 6 Tools)

`register`, `activate`, `login`, `refresh_access_token`, `auth_status`, `onto_credit_pricing`

`tools/list` ohne Token liefert **HTTP 401** statt einer Liste — die sechs sind über den
**Namen** aufrufbar (der `www-authenticate`-Header nennt sie). Ohne Token sind diese sechs auch
**ausführbar**; alle anderen antworten ohne Token mit 401, auch wenn sie sichtbar wären.

**Was das für den Client heißt:** Ein Client, der erst `initialize` und `tools/list` erfolgreich
abschließen muss — die üblichen SDKs, Desktop-Clients, auch Hermes — bekommt ohne Token **gar
keinen** Werkzeugkatalog. Der erste Aufruf muss die sechs Namen also kennen. Ein Werkzeug, das
nicht im Katalog steht, existiert für den Agenten nicht, auch wenn der Server es ausführen würde.

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
6 lesend für `admin`+`member`, 3 schreibend nur `admin`).

**Die Lehre:** ein Tool, das auf einen 404-Pfad zeigt UND für niemanden erreichbar ist, ist
toter Ballast — es bläht den Katalog auf und kostet in der Tool-Qualitätsbewertung (Tool-Anzahl).
Beides zusammen prüfen, nicht nur die Sichtbarkeit.

## Basis-Tools

| Tool | Rollen | Zweck |
|---|---|---|
| `auth_status` | öffentlich | Diagnose: Token gültig? welche Rollen? was tun? |
| `login` | öffentlich | client_id + client_secret → Bearer-Token |
| `refresh_access_token` | öffentlich | Token erneuern (kein Passwort) |
| `register` | öffentlich | Account anlegen (Mensch: `username`+Voucher · Agent: `invite_code`) |
| `activate` | öffentlich | Voucher einlösen (nur E-Mail-Weg; beim Einladungscode entfällt es) |
| `onto_credit_pricing` | öffentlich | Preise lesen |
| `get_image` | `member` | Bild/Datei herunterladen |
| `get_document` | `member` | Dokument herunterladen |
| `upload_file_to_object` | `member` | Datei/Bild an ein Objekt (auch `structuredimage`) |
| `upload_staging_csv` | `member` | CSV in die Import-Staging-Ablage |
| `upload_code_allowlist` | `member`, `admin` | XLSX/XLS-Bulk der E-Mail-Allowlist eines Invite-Codes |
| `change_password` | `member` | Passwort ändern |
| `change_profile` | `member` | Profil ändern |
| `renew_account` | `member` | Konto verlängern |
| `get_notifications` | `member` | Benachrichtigungen |

## Tool-Katalog der Ontologie-Endpunkte

### `onto_object_*`

| Tool | Rollen |
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

| Tool | Rollen |
|---|---|
| `onto_type_create` | `admin` |
| `onto_type_delete` | `admin` |
| `onto_type_index` | `admin` |
| `onto_type_show` | `member` |
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
| `onto_property_def_create` | `admin`, `member` |
| `onto_property_def_delete` | `admin` |
| `onto_property_def_index` | `admin` |
| `onto_property_def_delete_override` | `admin`, `member` |
| `onto_property_def_update` | `admin`, `member` |

### `onto_property_set_*`

| Tool | Rollen |
|---|---|
| `onto_property_set_create` | `admin`, `member` |
| `onto_property_set_delete` | `admin` |
| `onto_property_set_index` | `admin` |
| `onto_property_set_update` | `admin`, `member` |

### `onto_interface_*`

| Tool | Rollen |
|---|---|
| `onto_interface_create_def` | `admin` |
| `onto_interface_delete_def` | `admin` |
| `onto_interface_index` | `admin` |
| `onto_interface_map_markers` | `member` |
| `onto_interface_show` | `member` |
| `onto_interface_update_def` | `admin` |

### `onto_actiondef_*`

| Tool | Rollen |
|---|---|
| `onto_actiondef_delete` | `member` |
| `onto_actiondef_index` | `admin` |
| `onto_actiondef_list` | `member` |
| `onto_actiondef_save` | `member` |

### `onto_scope_*`

| Tool | Rollen |
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

> **`onto_user_delete`** räumt in EINEM Aufruf alles zum Konto weg: Token, Passkeys, Sitzungen,
> Trackings, Rollen, Scope-Zuordnungen, Favoriten, Merkzeilen, eigene Benachrichtigungen,
> Profil und Konto — und zählt danach nach. `id` darf die Konto-ID, die Profil-ID **oder** die
> `email` sein. Die Antwort nennt `entfernt` und `behalten`
> (Buchhaltung, Protokoll, Daten anderer). `onto_user_delete_profile` ist derselbe Weg
> (Altname). Nicht gefunden = HTTP 404, kein stiller Erfolg.

### `onto_report_*`

| Tool | Rollen |
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

| Tool | Rollen |
|---|---|
| `onto_catalog_apply` | `member` |
| `onto_catalog_clear` | `member` |
| `onto_catalog_index` | `member` |
| `onto_catalog_presets` | `member` |

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
| `onto_favorite_add` | `member` |
| `onto_favorite_delete` | `member` |
| `onto_favorite_index` | `member` |
| `onto_favorite_toggle` | `member` |

### `onto_file_*`

| Tool | Rollen |
|---|---|
| `onto_file_delete` | `member` |
| `onto_file_structured` | `member` |

### `onto_series_*` (Sensor-Zeitreihen)

| Tool | Rollen |
|---|---|
| `onto_series_range` | `member` |
| `onto_series_last` | `member` |

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
| `onto_trigger_create` | `member` |
| `onto_trigger_delete` | `member` |
| `onto_trigger_index` | `admin`, `member` |
| `onto_trigger_update` | `member` |

### `onto_function_*`

| Tool | Rollen |
|---|---|
| `onto_function_create` | `member` |
| `onto_function_delete` | `member` |
| `onto_function_index` | `admin`, `member` |
| `onto_function_run` | `admin`, `member` |
| `onto_function_update` | `member` |

### `onto_credit_*`

| Tool | Rollen |
|---|---|
| `onto_credit_balance` | `admin`, `member` |
| `onto_credit_chargeup` | `admin`, `member` |
| `onto_credit_checkout` | `admin`, `member` |
| `onto_credit_create_prepaid` | `admin` |
| `onto_credit_receipt` | `admin`, `member` |

### `onto_watch_*`

| Tool | Rollen |
|---|---|
| `onto_watch_changes` | `member` |
| `onto_watch_cursor` | `member` |

### `onto_action_*`

| Tool | Rollen |
|---|---|
| `onto_action_logs` | `admin` |

### `ontouser_scope_*`

| Tool | Rollen |
|---|---|
| `ontouser_scope_assign_public` | `member`, `admin` |
| `ontouser_scope_create` | `member`, `admin` |
| `ontouser_scope_index` | `member`, `admin` |
| `ontouser_scope_list_public` | `member`, `admin` |
| `ontouser_scope_remove_public` | `member`, `admin` |
| `ontouser_scope_set_visibility` | `member`, `admin` |

### `ontouser_interface_*`

| Tool | Rollen |
|---|---|
| `ontouser_interface_map_markers` | `member` |
| `ontouser_interface_show` | `member` |

### `ontouser_notification_*`

| Tool | Rollen |
|---|---|
| `ontouser_notification_index` | `member`, `admin` |
| `ontouser_notification_read` | `member`, `admin` |

### `ontouser_object_*`

| Tool | Rollen |
|---|---|
| `ontouser_object_show` | `member` |

### `ontouser_service_*`

| Tool | Rollen |
|---|---|
| `ontouser_service_request` | `member`, `admin` |

### Formular & Invite/Mitglieder (ohne Namenspräfix)

| Tool | Rollen |
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
3. **Zwei Namensfamilien für dieselbe Sache**: `onto_scope_*` ist die Admin-Sicht
   (`onto_scope_create` ist `admin`), `ontouser_scope_*` die Selbstverwaltung. Für eigene
   Scopes die `ontouser_*`-Familie verwenden — beide Wege rechnen ab und zählen gegen
   `profiles.max_scopes`.
4. **Reihenfolge der Tools im Katalog ist keine Priorität.** Die Liste ist nach Ressourcen
   gruppiert; die Wahl des richtigen Wegs steht in `references/features/`.
5. **Eine falsche Rollenzuordnung ist stumm.** Wird ein Werkzeug für mehr Konten sichtbar als
   beabsichtigt oder für alle unsichtbar, gibt es keine Fehlermeldung und keinen Log-Eintrag.
   Prüfmittel ist die Zahl aus `auth_status` (`tools_visible`) — nach jeder Rechteänderung
   gegen den Sollwert stellen.
6. **Parameter sind in den Tool-Schemata dokumentiert, nicht hier.** Bei Unsicherheit
   `tools/list` auf das konkrete Tool ansehen — die `description` enthält das verifizierte
   Body-Format und die Pflichtfelder.
7. **Die eigene Zahl kommt aus `auth_status`.** `tools_visible` gilt für das angemeldete
   Konto; nach jeder Rechteänderung neu stellen statt aus diesem Katalog ableiten.
8. **`admin` sieht NICHT alles.** Ein Teil der Tools trägt ausschließlich `member`, ein
   anderer ausschließlich `admin`; nur ein Teil beide. Ein Admin-Konto, das `change_password`
   oder `onto_catalog_apply` vermisst, hat keinen Fehler — die Rollenliste nennt dort nur
   `member`. Beim Prüfen immer **beide** Zahlen stellen (`auth_status` mit dem
   jeweiligen Konto).

## Tool-Beschreibungen und Annotations

Jedes Tool hat eine **sprechende Beschreibung** (Verb + Ressource + wann/wann nicht + genannte
Alternative) und **MCP-Annotations** (`readOnlyHint`, `destructiveHint`, `idempotentHint`,
`openWorldHint`). Ein Tool ohne Beschreibung und ohne Annotations ist für ein Modell kaum
auswählbar — beides gehört dazu.

Die Beschreibungen sind zentral gepflegt und werden beim Start eingelesen. Sie lassen sich
damit ändern, ohne die Tool-Definitionen anzufassen.
