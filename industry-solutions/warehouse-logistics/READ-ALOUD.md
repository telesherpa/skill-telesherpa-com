# Warehouse logistics — read-aloud text, version 1.0

This document is the warehouse-logistics skill spoken as continuous prose, so it can be read aloud. Tables and configuration snippets from the skill are turned into sentences. Tool names and JSON keys are spoken as they are written. The skill itself lives at industry-solutions/warehouse-logistics, next to the platform skill and the shift-planning skill.

## Summary

Version 1.0 of the warehouse-logistics skill tells an agent how to set up QR labels, packages and stock bookings on Telesherpa. There are three scopes: master data, warehouse and team. A package carries its book quantity, its unit, and, when the business uses them, a lot and a best-before date. A printed label has its own Crockford code, not the object id, and is only a pointer. A single scan binds a free label to a package. Leadership can hang a bound label onto another package. Labels are printed on Avery 3475 sheets, twenty-four to an A4 page. Managers book stock with a single scan of the package: put away, pick, relocate and count. Each booking writes a stock-movement row and updates the package quantity in the same run. The place named in the prompt is recorded on the movement. Staff bind labels through the team door and the scan page; this skill describes stock bookings on the manager path. People, sites and the three-scope pattern are reused from shift-planning when that workspace already exists. Load the platform skill first. This file is English because agents read it; answer the customer in their language; keep tool names and keys verbatim.

## How to use the skill

Load telesherpa-ontology-platform first, then this skill. You are expected to be able to register, authenticate and call MCP tools. Option values are plain strings with no translation layer. The actions label_bind and label_rebind are proven against the platform values frei, gebunden and gesperrt; do not replace those values. Other option values you create yourself are English. Labels shown to the customer's staff are set in the language agreed in the intake. Tool responses and error texts may use the platform language. Read the returned allowed keys and the meaning of an error; never translate keys or option values back into a call.

Reuse, do not copy. People, sites — organisation and building — and the three-scope pattern already live in shift-planning. If that workspace exists for the same business, reuse its master data and, if the intake says so, its team door. Link to those files; do not duplicate their recipes here.

Restaurant values in the skill are examples, using the fictional business DemoRestaurant.

## Before you claim anything — the checks

A workspace is not set up because objects exist. Before you write any progress or completion report, run the following checks and quote the values you read back. A statement without numbers from a read-back is a claim, not a status.

First, the three scopes exist with the agreed suffixes, and you are manager of all three. Query ontouser_scope_index.

Second, for each scope, quote the number of objects, members and managers from onto_scope_index, using the fields obj_count, user_count and manager_count.

Third, onto_type_show must show that the type label carries code, charge, labelStatus with the values frei, gebunden and gesperrt, printedAt and printedBy; that package carries menge as an integer, einheit, lot and bestBefore; and that stockMovement carries movementType, menge, einheit, grund, bookingTime and bookedBy.

Fourth, views exist and the start card is registered. Query onto_settings_list per scope and onto_startcard_index.

Fifth, the actions label_bind, label_rebind, stock_putaway, stock_pick, stock_relocate and stock_count are present. Each stock action points at a function whose scopes include the package's scope. Every manager action carries require_scope_write. Query onto_actiondef_list and onto_function_index.

Sixth, objects per type match the intake — business, site, people, storage, packages, labels. Query onto_object_index.

Seventh, one bound label really points at a package, and the quantity sits on that package. Use onto_object_graph on the label for the identifies edge, and onto_object_show on the package for menge, einheit, lot and bestBefore, and on the label to confirm there is no menge there.

Eighth, one real end-to-end step as staff: a free label is bound through a single scan, using GET /ontouser/scan or the HTTP contract in the platform scan chapter, then a read-back of labelStatus and the identifies edge.

Ninth, a print sheet for one charge exists; a reprint of that charge creates no new labels. Open and read the PDF, and compare onto_object_index counts for that charge before and after.

Tenth, one manager booking each of stock_putaway, stock_pick, stock_relocate and stock_count, plus one label_rebind, was read back — quantity, movement, the moves and movesTo links, and identifies. Use onto_object_execute_action and then onto_object_show.

Eleventh, the credit balance you report is the one you measured with onto_credit_balance.

A status message names its stage as "stage X of 5", covering phases 0 to 5. "Everything is set up" is not a status: it names no phase, no number and no check. If you cannot fill the table, you are not done, and the correct answer is the missing line, not a summary.

## Status of version 1.0

This version was verified against the platform schema on the thirteenth of November 2026, Europe/Berlin. Types, properties, the links identifies, moves and movesTo, and the catalogue preset lager were read with member tools on that date. Stock bookings and label_rebind were executed and read back on that date. Treat points marked "confirm" as open until they are quoted from your workspace.

Version 1.0 covers labels, print sheets, single-scan bind and rebind, the three-scope layout, and single-scan putaway, pick, relocate and count, each writing a stockMovement and updating menge.

Links point to the main branch of the platform skill, read on the same date. The platform skill may not document the label type, the short URL slash q slash and then the code, label_bind, label_rebind, identifies, Avery 3475 print, stockMovement or the stock actions. For those topics this skill and a live onto_type_show, onto_link_index and onto_actiondef_list win. If a linked file has moved, look it up by name and report the broken link; do not guess its content.

The skill is for restaurants and other businesses that store goods in packages and scan labels.

The tool count is not stated on purpose. Measure it with GET https://mcp.telesherpa.com/health, field tools, and with auth_status for what your account sees.

## Goal

Managers maintain sites, storage locations, packages and printed labels. A package carries its book quantity menge, unit einheit, lot and best-before date bestBefore. A label is only a pointer: it has its own code, it can be printed, locked, issued more than once for the same package, and rebound onto another package by leadership. A single scan binds a free label to a package. Staff reach the bind action through the team door and the scan page. After sign-in, the short URL on the sticker opens the linked package; before sign-in the object name is not shown.

Managers book stock through single-scan actions on the package: put away, pick, relocate and count. Each booking creates a stockMovement and updates menge in the same run. The place named in the prompt is recorded on the movement as movesTo.

The operating picture is in operating-model.md, described later in this reading.

Charts and reports: offer them, do not build them yourself. A member cannot build a report; order it through the service channel. The menu, the wording and the request template are in the platform skill, reports.md. What a member can build is a block view and, where the view allows it, a CSV of labels, packages and movements for the manager, in phase 4.

## Definition of Done

The workspace is configured when a person who has never seen the build can print a sheet of labels, stick one on a package, bind it with a single scan, and see the quantity on that package, and a manager can put away, pick, relocate and count that package — without asking anyone. Everything below is a precondition for saying that, not a description of it.

Phase 0 is done when the three scopes exist with the agreed suffixes, the rights matrix is set, types are activated, functions and actions are installed in matching scopes, and the smoke test is quoted: ontouser_scope_index lists three scopes; a staff test returns 403 on the warehouse scope; onto_type_show on label, package and stockMovement quotes the fields above; onto_function_index shows stock_move_apply and stock_count_apply with the package scope in scopes.

Phase 1 is done when the business and site are reused or created, people, storage locations and packages with menge, einheit, lot and bestBefore exist, there is one label charge and one print sheet. Quote object counts per type, one package with quantity, lot and best-before, and one label with code, charge and labelStatus frei.

Phase 2 is done when bind, several labels on one package, lock, and a reprint of the same charge are quoted: identifies on a bound label, four labels on one package, a locked label no longer resolves, and the label count for the charge is unchanged after reprint.

Phase 3 is done when staff accounts, the start card, a bind performed as staff through a single scan, and short-URL behaviour are quoted: the staff login binds a free label; the short URL hides the name before sign-in and opens the package after.

Phase 4 is done when action-log evidence and a manager list or CSV of labels, packages and movements are quoted: one label_bind row in onto_action_logs, and the list or CSV opened and read.

Phase 5 is done when putaway, pick, relocate, count and label rebind have each been read back: menge after putaway and pick; unchanged menge after relocate; absolute menge after count; identifies on the new package after rebind; each movement has moves and movesTo.

The hard rule: the workspace counts as configured only when the acceptance scenario has run end to end once and its result is quoted, step by step. Until then the status is "stage X of 5". "Created" is not "configured". An object list proves that rows exist. A working workspace proves that the next person can print, stick, bind and book.

## Build order — phases 0 to 5

Every phase file uses the same frame: goal, definitions, config snippets, checks via MCP, checks in the browser per role, acceptance, and typical errors.

Phase 0, in phase0.md, covers three scopes, the rights matrix, object types, properties, link types, functions, actions and the smoke test. Phase 1, in phase1.md, reuses or creates sites and people, storage, packages, a label charge, Avery 3475 print and manager lists. Phase 2, in phase2.md, covers label_bind, several labels on one package, lock with status gesperrt, and reprint of the same charge. Phase 3, in phase3.md, covers staff accounts, the scan page, the start card, and the short URL before and after sign-in. Phase 4, in phase4.md, covers the action log and a manager list or CSV of labels, packages and movements. Phase 5, in phase5.md, covers stock_putaway, stock_pick, stock_relocate, stock_count, label_rebind, lot and best-before.

Before you start, read prerequisites, then the intake — ask the business, do not guess — then begin phase 0. The end-to-end check is the acceptance scenario. Paid test runs follow measurement.md. The operating model is in operating-model.md.

Working conventions for every phase: test objects carry the prefix TEST-, then the phase, in name and external_id. Save the current definition — read it, keep the JSON — before you change it. Read back after every write. A write response describes intent; the read-back describes state. Leave e-mail and push off on notification rules unless the intake asks for a notice you can actually send and test.

## Core rules, to be read before phase 0

One. Staff have no right on the warehouse scope, neither read nor write, and read, no write, on master. They reach a label through the scan — the code on the sticker — not through a warehouse membership. Writing the bind goes through a scope bridge, action.result_scope_id, limited to the keys in action.props and the identifies link. Do not grant warehouse read "so the scan works". If a staff bind fails because the object is unreachable, stop and report; do not widen rights as a workaround. Stock bookings in this skill are described on the manager path: an account with write on the warehouse and on the package's scope. Do not tell the customer that staff can book putaway, pick, relocate or count through the team door unless you have measured that path in this workspace.

Two. The E2 split, basic versus protected. Master objects that staff can read hold only basic data: name, login e-mail, site, storage location, package name, menge, einheit, lot and bestBefore. Supplier terms and purchase prices do not belong on staff-readable master objects. See the platform file on scopes and permissions.

Three. A label is a pointer, not the stock. menge as an integer, einheit, lot and bestBefore hang on the package. They do not hang on the label. Several labels may identify one package — proven: four labels on one crate. Each label points to at most one package; identifies is unique on the label side.

Four. The code is not the object UDID. The platform assigns a Crockford Base32 code, twelve characters plus a check digit, without the letters I, L, O or U. You never invent the code. The sticker prints it as the public-app short URL /q/ followed by the code. After sign-in the scan opens the linked object; before sign-in the object name is not visible. Checkpoint codes in the platform skill still use a raw UDID — that is a different feature; do not put a UDID in a warehouse label.

Five. label_bind is a single-scan action, scan.ein_code. The condition is that labelStatus is frei. The action prompts for the package, object_types package, and the client fills the UDID. It sets the link identifies and labelStatus gebunden. A second bind is rejected with require absent — proven result: HTTP 400, platform message that the precondition is not met. To hang an already bound label on another package, use label_rebind, in phase 5.

Six. Status values on labelStatus are frei, gebunden and gesperrt. Gesperrt means the sticker is intentionally dead, lost or replaced, and must no longer resolve. Do not rename these values.

Seven. Print. A charge of labels is printed on Avery Zweckform 3475, seventy by thirty-six millimetres, three by eight equals twenty-four labels per A4 sheet, QR twenty-six millimetres, error-correction level M, the code readable in groups of four, the charge on the PDF. A reprint of the same charge must not create new labels. Discover the live print action or view with onto_actiondef_list and the type's interfaces; do not invent a tool name. If no print path is visible after the types are activated, stop and file ontouser_service_request.

Eight. Rights are fail-closed. No grant means no access; an empty list means empty. Set read, write and manage explicitly.

Nine. Delete one id at a time. Never by name, prefix or pattern. Deleting costs the delete price from the live table. See prerequisites.

Ten. UDIDs always lower case. Mixed case silently never matches.

Eleven. Every manager view sets view_config.access.require. Every manager action sets require_scope_write to the warehouse scope id. Staff-only-own-row actions carry a guard in the live format, a list of objects with type linked_to_user, a linktype, and a message. See the platform file on scope bridges. Install label_rebind with require_scope_write so only leadership can run it.

Twelve. Unlink with remove set to the string pair, never remove true. label_rebind already replaces identifies on the label side, replace source, and the execute response reports the removed edge as links_entfernt.

Thirteen. Time: writes are interpreted as Europe/Berlin, the platform zone; reads return UTC.

Fourteen. Scan routes are not an MCP tool. POST /onto/object/match_actions, POST /onto/object/execute_action, POST /onto/object/counter and GET /ontouser/scan run over HTTP with session or bearer, role member. Details are in the platform chapter on scan and checkpoints. Use that chapter for the server contract, ein_code, prompts, require, feedback and dedupe_ms. The stock actions in this skill are single-scan, scan.ein_code: one code identifies the package, then the action prompts for quantity and, where required, the storage location.

Fifteen. Prices only from the live list. Call onto_credit_pricing at the start of the run. Do not copy a price table from memory or from this skill. The platform skill and the tool-traps file describe how to read it.

Sixteen. Change menge only by booking a movement. Putaway adds the booked quantity, pick subtracts it, relocate leaves it unchanged, count sets it to the counted quantity. Do not use onto_object_set_property on menge to "correct" stock.

Seventeen. The stock action and its function must be allowed in the package's scope. If the function's scopes list does not include that scope, the execute call returns HTTP 403 with the platform message that the function is not allowed in this scope, and no movement is written. Fix the function, or install the canonical keys so action and function share the same scopes. Do not widen account rights as a workaround.

## Link types of this solution

Use these, invent none.

identifies runs from label to package. It is the package the sticker points at, unique on the label side. It is set by label_bind and moved by label_rebind.

moves runs from stockMovement to package. It is the package the booking applies to.

movesTo runs from stockMovement to storage. It is the storage location named in the booking prompt.

bestehtAus runs from organisation to building, and further if used. It is the site tree, the same as in shift-planning.

hasfile runs from any object to a file. It carries the print PDF and attachments.

The type also defines movesFrom, from movement to storage. The shipped functions record the prompted place on movesTo. Quote what you read back; do not invent a second place link in the report if it is not there.

A link type that is not in onto_link_index with matching types1 and types2 for the pair you need does not exist for you. identifies is defined label to package only. And: a link you have not read back with onto_object_graph or onto_object_show does not exist in the report either.

The skill points to the platform references for scopes and permissions, scope bridges, tool traps, prices, time, reports and charts, member-agent rules, notifications, start cards, forms, block views, triggers and functions, managed staff accounts, QR scan over HTTP, and QR checkpoint rounds, which are a different feature that puts a UDID on the point. People, sites and the three-scope recipe come from shift-planning, especially phase 0 and phase 1. Do not duplicate those files here.

## Stop and escalate

Stop, do not improvise, and report when any of the following happens. Inventing a workaround is forbidden — for example granting staff warehouse read, inventing label codes, putting quantity on the label, or editing menge by hand instead of booking a movement.

On HTTP 402, the executing account has no credits. Check onto_credit_balance of that account. Stop and ask for credits. Do not retry.

On HTTP 403 or a refused scope write, check rights. If the design says the account must not have the right, the fix is a bridge or a gate, not a grant. If the message says the function is not allowed in this scope, align the function's scopes with the package's scope and retry once.

If a required object type is missing from ontouser_scope_types or onto_type_show, stop. File ontouser_service_request, using the template in prerequisites. Do not substitute another type silently.

If no print path is visible for Avery 3475 after types are activated, stop and file a service request. Do not generate QR images from UDIDs yourself.

If a tool reports unknown parameters, read the allowed keys and fix the call once. If the documented parameter is not accepted, stop and report; the skill is out of date.

If the smoke test in phase 0 deviates from this skill, stop before building. Report the deviation with tool, call and response.

On an MCP timeout or a connection drop on a write, read back. Only repeat if the read-back proves nothing was written.

On three consecutive failed attempts at the same step, or spend above the agreed budget, stop and report, as measurement.md describes.

If the business is not on Europe/Berlin time, stop before phase 0 and file ontouser_service_request.

If anything would message real people outside a test, ask the human first.

If an intake answer is missing — sites, units, who may bind, who may book, how labels are handed out — ask; do not assume DemoRestaurant defaults for a real customer.

If you are about to report completion, stop. Run the checks above and quote the values.

The report to the human, and for platform faults via ontouser_service_request with category bug, states what you tried, tool and arguments without secrets, what you expected, what came back, status and error text, and what you did not do because of it.

Every status report uses the same shape: stage X of 5, then a table of values read back — the three scopes, objects per type including stockMovement, views and the start card, the six actions with manager actions gated, the two functions with scopes that include the package scope, a bound label with identifies and menge on the package, the last bookings, and the count of passed acceptance steps out of thirty-eight, from 0.1 to 5.8, and of open negative tests out of fourteen. Then a list of what is not done, each with the stage it belongs to.

Two lines are forbidden in this report: "everything is set up" and "done". The count of passed acceptance steps is the only honest answer to "are you finished". The scenario has thirty-eight steps, 0.1 to 5.8, and fourteen negative tests, N1 to N14. Both counts are in the acceptance file; do not restate them from memory.

## Prerequisites

Check every point before phase 0. Do not trust the file blindly — measure.

A fresh account starts with zero credits, and creating scopes already costs the volume price from the live table. Three new scopes on a fresh account are billed as the first, second and third own scope. If onto_credit_balance is zero, the first write answers HTTP 402. Get the money sorted before you build anything: a prepaid code from the operator, a card payment by the human, or a service request if you are an agent without a payment method. Ask for the amount you actually need. Measurement.md explains how to budget. Phase 0 alone needs the live price of the scopes you still have to create; reuse of an existing master or team door reduces that. Do not start phase 0 with less than phase 0's cost in the account, and do not improvise a workaround.

You need a Telesherpa account with MCP access; role member is enough. You create missing scopes with ontouser_scope_create and become their owner. If shift-planning already exists for this business, reuse its master scope and, if the intake says so, its team scope, instead of creating duplicates. A manager, human or agent, named in the intake, holds read, write and manage on all three scopes. You need at least one staff test account, an activated managed account with rights as in phase 0. Human registration needs an activation e-mail at a dedicated controlled mailbox; documentation addresses at example.com cannot be activated. Browser access is required for acceptance: the print PDF, the short URL before sign-in, the scan page and what a staff member sees are not proven by MCP lists alone. Plan a human or a browser-capable agent logging in as manager and as staff, and one logged-out fetch of the short URL. The platform interprets writes as Europe/Berlin and returns UTC on reads. If the business is in another zone, stop before phase 0 and file a service request.

First step: measure, do not assume. Run the read-only calls first and keep the answers in your run log. Health at the MCP host must be ok. Note the tools value; do not compare it with any number in a document. auth_status must show a valid token, roles containing member, and the number of tools visible to your account. List your tools and confirm that every tool named for the phases exists; a missing tool means stop. Phase 0 needs the scope, type, property, link, catalogue, team-rights and credit tools, and also onto_function_index, onto_function_create and onto_function_update. Phase 1 needs object create, show, index, set property, execute action, actiondef and interface tools. Phase 3 needs invite codes, transfers and start cards. Phase 4 needs the action-log tool, under the current name in your catalogue. Phase 5 uses execute action again. You also need the service-request, comment, notification and delete tools, and, for acceptance clean-up, the offboard preview and offboard tools.

onto_credit_balance gives balance and billing_model. Flat accounts are not charged; a measured test run needs a pay-per-use account. onto_credit_pricing must be saved in full with date and time, Europe/Berlin. That table wins over every figure you remember. Do not copy an old table into this run.

There is no starting credit. A fresh account has zero credits. Never trigger a payment without explicit instruction from the human. The executing account pays. A staff member running label_bind pays the action price from their balance, even through a scope bridge. Staff without credits get HTTP 402. Give each staff account a small balance with ontouser_team_transfer; reason and idempotency_key are required. Lists are free; opening a single object costs read. Writes and deletes follow the live table.

Do not quote prices from the skill. Typical keys you will see, with the live table as the authority, include read, create_object, create_link, action, file_upload, delete_object, delete_link, and the volume bands for own scopes. One credit is 0.01 euro, unless the live table says otherwise. Deleting is expensive. Budget clean-up, or keep test objects with the TEST prefix. Creating property definitions, actions and views may have no own price key. Log what is booked. Whether a print or charge action is billed as action, file_upload or another key is visible only on your statement — confirm it on the first sheet.

All types this skill uses are platform types; you activate them per scope, you do not create them. The catalogue preset lager bundles package, storage, label, person and file. It does not include organisation or building. Read onto_catalog_index first. Prefer activating types with ontouser_scope_types_set; the call replaces the list. Apply the preset only on an empty scope after you have read what it contains.

Organisation is the business; reuse it from shift-planning if it exists. Building is the site or branch; reuse it. Person is people who bind and who manage; email binds the login; reuse. Storage is the physical slot, in master; bookings record it on movesTo. Package is the physical unit and carries menge, einheit, lot and bestBefore; change menge only by booking a movement. Label is the printed sticker with its own code and charge, a pointer only, with actions label_bind and label_rebind. stockMovement is one booking row, with movementType putaway, pick, relocate or count, and links moves and movesTo. File is the print PDF, via hasfile.

Check each type with onto_type_show. Also confirm stock_move_apply and stock_count_apply in onto_function_index, or plan to install them in phase 0.

If a type is missing, stop and file a service request. A member agent cannot create types. The template uses ontouser_service_request, category feature, a subject that names the missing type, and a text that says you are setting up warehouse logistics, industry skill warehouse-logistics, version 1.0, in the three company scopes, that onto_type_show for the type returned the status or error, what you need it for, and that you have not created any workaround. The same template, category bug, is used for any platform fault from the stop list, including a missing Avery 3475 print path.

The checklist before phase 0: health ok, auth_status member, the tool list contains every tool named in phase 0; the billing model is known and the balance covers the phase-0 budget; the price table is saved with a timestamp; the types label, package, storage, stockMovement, person, organisation, building and file are readable with onto_type_show; the links identifies, moves and movesTo are present in onto_link_index; the intake is answered and confirmed by the business; and the decision is recorded, whether to reuse existing shift-planning master and maybe team, or to create new scopes.

## Intake

Send the questions to the business, owner or manager, and get written answers. Do not start phase 0 with gaps in sections A, B, D and E. Section C may be completed before phase 1, print and bind, if A, B, D and E are already closed. DemoRestaurant answers are examples; never use them as defaults for a real customer. Answers are kept in a short intake record. Staff personal data never goes into chat logs or public places. If shift-planning was already answered for this business, reuse those answers for the legal name, sites, people, label language, slug, budget contact and time zone. Ask only what is new here.

Section A, business and structure. One: name of the business, legal and short, and whether it already has shift-planning scopes. Two: sites, one or several, and whether stock is kept per site; use one building per site and name labels and packages per site. Three: physical storage areas, kitchen, cellar, cold room, off-site, and whether slots are fixed or chaotic. Four: units in use, and whether lot and best-before are required; those are einheit, lot and bestBefore on the package. Five: country, region and time zone; if it is not Europe/Berlin, stop and file a service request. Six: who maintains the list of packages and storage locations.

Section B, people and roles. Seven: who is the manager with full rights, human, agent, or both. Eight: is there a warehouse lead who binds and prints but must not change the business or people in master. Nine: people who scan, first name, last name, login e-mail, site; reuse shift-planning persons where they are the same people. Ten: does every scanner have their own e-mail and a device. Eleven: staff can read master packages, name, menge, einheit, lot, bestBefore, and storage locations; is that acceptable? Purchase prices do not belong on those objects.

Section C, rules and process. Twelve: who may bind a free label to a package, only warehouse staff, or also kitchen. Thirteen: how many labels per package are typical; the platform allows several, and four on one crate is proven. Fourteen: who may lock a label, gesperrt, when a sticker is lost or replaced. Fifteen: who prints a charge, and who reprints the same charge. Sixteen: who may book putaway, pick, relocate and count; this skill describes the manager path. Seventeen: who may rebind a bound label onto another package; label_rebind carries require_scope_write.

Section D, naming and language. Eighteen: label language for staff-facing texts. Nineteen: naming pattern for packages, storage and label charges. Twenty: the scope slug, lower case letters, digits and underscore; the scope name may have thirty characters and _warehouse takes ten, so the slug has at most twenty. If shift-planning already used a slug, keep it and only add _warehouse.

Section E, budget and contact. Twenty-one: credit budget for setup and test, and who tops up. Twenty-two: who answers questions during the build, and how fast. Twenty-three: who logs in as staff for the acceptance test, and who holds a printed test sheet to scan.

Mapping: questions 1, 2 and 20 feed phase 0 scope names and phase 1 business and site objects. Questions 3, 4 and 19 feed phase 1 storage and packages. Questions 7, 8, 10 and 12 to 17 feed phase 0 rights and the actions in phases 2, 3 and 5. Question 9 feeds phase 1 people and phase 3 accounts. Question 5 selects the country module. Questions 18 and 19 feed all labels and names. Questions 21 to 23 feed measurement and acceptance.

Create scope-specific property definitions for any new option list you introduce, with English values. Do not replace the platform values frei, gebunden and gesperrt on labelStatus; label_bind and label_rebind depend on them. The name the business uses becomes the option label in the language of question 18.

## Phase 0 — scopes, rights, types, functions and actions

The goal is three scopes with the right types activated, the rights of every account written down and set, the stock functions and actions installed in matching scopes, and a smoke test that proves the definitions are in place. No business data yet; that is phase 1.

The master scope holds business, sites, people with basic fields, storage locations, and packages with menge, einheit, lot and bestBefore. Managers write; staff read. The warehouse scope holds labels, print charges and files, stockMovement rows, functions, action definitions and manager views. Managers and a warehouse lead work here. The team scope is the staff door: staff views, the start card for the scan page, and staff-facing bind actions shared with warehouse.

Scope names use letters, digits and underscore, at most thirty characters. Check the longest one, the company slug plus _warehouse. The scope is the lock. Never rely on a view filter to hide data. If shift-planning already created master and team for this business, add only warehouse, keep the existing people and sites, and record that decision in the run log. A second master with a copy of the persons is a defect. Write all three names and returned ids into your run log, not into the public skill.

Links cross scopes. identifies runs from a label in warehouse to a package in master. The manager creates the first test bind, with write on both scopes. Staff create further binds through the scan and the scope bridge. A movement links from a stockMovement to a package on master and to a storage location on master; after the first test booking, quote the scope the movement landed in.

The rights matrix: the manager or owner has read, write and manage on all three scopes. An optional warehouse lead has read on master, and read plus write on warehouse and team, but not manage. Warehouse staff have read, no write, on master, no membership at all on warehouse, and read plus write on the team door. Kitchen, only if the intake allows them to bind, has the same pattern as staff, limited to the bind action. Staff open a label through the scan, not through a scope right. The write door for bind is the scope bridge. A direct onto_object_show of a warehouse label as staff must answer 404 or no access, confirmed in phase 3. Staff read on master is needed so the package picker on label_bind can list packages and so storage names resolve. Do not grant warehouse read "so views work". Stock bookings and label_rebind stay on the manager path in this skill.

Activate on master: organisation, building, person, storage, package, file. On warehouse: label, stockMovement, file, comment if you keep notes, plus whatever the print path requires. On team: no extra master types. ontouser_scope_types_set replaces the list. Include every type the scope must keep. The preset lager may be applied only on an empty scope after onto_catalog_index has been read; it will not add organisation or building. Types are global; then apply the preset; then install functions and actions for this warehouse — one planned rollout, not a chain of one-off edits.

onto_type_show must quote, on label, code, charge, labelStatus with frei, gebunden and gesperrt, printedAt, printedBy, and the action keys label_bind and label_rebind. On package, menge, einheit, lot, bestBefore, and the action keys stock_putaway, stock_pick, stock_relocate and stock_count. On stockMovement, movementType including putaway, pick, relocate and count, menge, einheit, grund, bookingTime and bookedBy. On storage, description, and the address set if present.

Do not override labelStatus with English values. You may set the words staff see in the intake language; the values stay as they are. The stock actions write the English movementType values putaway, pick, relocate and count.

Confirm with onto_link_index, do not invent: identifies, label to package, unique on the label side; moves, stockMovement to package; movesTo, stockMovement to storage; bestehtAus for the site tree; hasfile for the print PDF.

Install the functions in the same scopes as the packages they will book. Include the master scope id in each function's scopes if packages live on master, and install the actions so they can run there. A mismatch returns HTTP 403, with the platform message that the function is not allowed in this scope.

stock_move_apply is used by putaway, pick and relocate. Inputs: objekt, the package, required; zugang, an integer, required; art, a string, required; vorzeichen, an integer, default 1; platz, a storage object, optional. The body computes new quantity as current menge plus zugang times vorzeichen. The effects create a stockMovement named with the art, with movementType, menge, bookingTime now, bookedBy the signed-in user's e-mail, links moves to the package and movesTo to the place, then set menge on the package to the result.

stock_count_apply is used by count. Inputs: objekt, required; ist, the counted integer, required; grund, optional; platz, optional. The body is the counted quantity. The effects create a stockMovement named as an inventory row with the time, movementType count, menge equal to ist, grund, bookingTime and bookedBy, the same two links, then set menge on the package to ist.

The canonical action keys use scan.ein_code, automatic execution, and require_scope_write on the warehouse scope for manager actions. stock_putaway prompts for a required place and a required quantity, and calls the move function with art putaway and sign plus one. stock_pick prompts for a required quantity and an optional place, art pick, sign minus one. stock_relocate prompts for a required new place and an optional quantity; pass the current book quantity; art relocate, sign zero. stock_count prompts for a required counted quantity, an optional place and an optional reason, and calls the count function. label_rebind prompts for paket, the new package, with condition labelStatus gebunden, and updates identifies with replace source. label_bind is as in phase 2, and also lives on the team door with a scope bridge.

The recipe: measure credits and prices; list scopes and reuse master and team if they exist; create only missing scopes and read each back; read the catalogue, set types including stockMovement on the warehouse; show every type and quote fields and action keys; quote the links; install the two functions and the six actions and read both back; set rights and read them back with ontouser_team_show; do not create business objects yet.

The MCP checks: three scopes, suffixes as agreed, you can manage all three; types present and fields quoted; identifies, moves and movesTo present with the right directions; both functions list the package scope; the six action keys are present and label_rebind carries require_scope_write; a staff test account gets 403 or equivalent no-access on the warehouse scope, and read on master succeeds. Browser checks in phase 0 are only that the manager can see the three scopes. You may enter phase 1 only when the MCP checks are quoted. Status: stage 0 of 5 complete.

Typical errors: creating a second master although shift-planning already has one; applying lager and dropping organisation or building from an existing master; overriding labelStatus to free, bound or locked and then watching label_bind fail; granting staff warehouse read after the 403 looks inconvenient; installing a stock action that points at a function whose scopes omit the package's scope, so the first booking returns 403.

## Phase 1 — sites, people, storage, packages, label charge, print

The goal is that the manager can see sites, people, storage locations and packages with a quantity, and holds one printed charge of free labels. Nothing is bound yet. Reuse people and sites from shift-planning phase 1 when they already exist.

Master objects: organisation is the business; building is the site, linked with bestehtAus, address set if used; person has the login e-mail in lower case and basic fields only; storage has a description and an optional address; package has menge as a non-negative integer, einheit from intake 4, and lot and bestBefore when the intake uses them. Do not write the package field place as a second location next to movements.

A label is created by the platform as part of a charge, not by you inventing a code. After creation, every label in the charge must read back with a Crockford Base32 code, twelve characters plus a check digit, without I, L, O or U; a charge identifier; labelStatus frei; and printedAt and printedBy set once the sheet has been printed, which you confirm. The print sheet is Avery Zweckform 3475, as already described. Discover the live action or view with onto_actiondef_list and the interfaces on label, or on a charge object if one exists. Quote the key you used. If nothing is offered, stop and file a service request; do not draw QR codes from UDIDs.

Manager views, in the warehouse scope, with access.require set: labels, filterable by labelStatus and charge; packages, with name, menge, einheit, lot, bestBefore, and site if encoded in the name; storage locations, which may live in a manager list in master or warehouse, but staff must not be given warehouse read to see storage, because storage is in master. Follow the platform file on view blocks. Rebuild from blocks; hard-reload before you judge the page.

The recipe: reuse or create the organisation and one building, read back, link bestehtAus; reuse or create the persons from the intake, read back the emails; create at least two storage objects and read them back; create at least two packages with different menge and einheit, and with lot and bestBefore when the intake uses them, and confirm each has quantity and unit and no identifies edge yet; create one charge of labels, a full twenty-four-up sheet is the proven format, a smaller test charge is acceptable if the platform allows it, quote the count, and check the code format on every new label or on a sample of five plus the total; print the charge, open the PDF, and check layout, QR size and the charge line; build the manager views and read them back. Do not bind a label in this phase; that is phase 2. Do not write place on a package as a second location next to the movement links.

MCP checks: object counts per type quoted; one package with menge and einheit, and lot and bestBefore when set; one label with code, charge and labelStatus frei, and menge absent on the label; codes match the Crockford rule, spot-check at least five; views exist and carry access.require. In the browser the manager sees the test packages and the free labels, and the PDF opens with the 3475 layout, or the quoted test count, the charge, and codes in groups of four. Staff login is not required yet. Phase gate: quoted object counts, one package with quantity, one free labelled charge, and a PDF that was opened. Status: stage 1 of 5 complete.

Typical errors: inventing a code and writing it onto a hand-made label object; storing quantity on the label because the sticker is what people scan; filling place to remember the slot, when the booking records the place on movesTo; printing with an external QR tool that encodes the UDID.

## Phase 2 — bind, several labels on one package, lock, reprint

The goal is that a free label can be bound to a package; one package can carry several labels; a second bind of the same label is rejected; a locked label no longer resolves; and reprinting the same charge creates no new labels.

The proven behaviour of label_bind, which you quote from your live action definition and do not invent a second key for: the object type is label; the condition is labelStatus equals frei, with a plain-text message on failure; the scan is ein_code, a single scan; the prompt is a required package, object_types package, object selection, the client fills the UDID; the effect is to create the link identifies from the label to the package and set labelStatus gebunden; a second bind, require absent on that link, returns HTTP 400 and the platform text that the precondition is not met.

If you copy the action into the customer warehouse and team scopes, keep this behaviour. Add require_scope_write on the manager-facing copy if managers also have a button. The staff-facing copy, in phase 3, runs from the team door with result_scope_id equal to the warehouse scope, and a guard if the intake limits who may bind.

Uniqueness is on the label side: one label identifies at most one package. The package side is not unique. Four labels on one crate is a proven case. Bind four free labels of the test charge to package A.

A label that is lost or replaced is set to gesperrt. After that it must no longer resolve through the short URL or the scan page: no object name, no bind. Discover the live action or manager path for the status change with onto_actiondef_list. Only the warehouse lead or manager may run it, with require_scope_write. Do not delete the label object to lock it; the code must remain reserved so a found sticker stays dead.

Printing the same charge again produces a new PDF of the existing labels and must not create new label objects. Count labels for that charge before and after; the numbers must match.

To hang a bound label onto a different package, use label_rebind in phase 5. Do not delete the identifies link by hand and bind again.

The recipe: as manager, run label_bind on one free label and select package A; read the label as gebunden and the graph as identifies to package A; read package A, menge unchanged by the bind, because bind does not book quantity. Bind three further free labels to the same package A; the graph shows four incoming identifies edges. Run label_bind again on a bound label; expect HTTP 400 and no change on read-back. Lock one label, quote which, confirm gesperrt, and confirm that resolve or scan no longer opens an object. Reprint the same charge and count labels before and after.

MCP and browser checks match those steps. Status: stage 2 of 5 complete.

Typical errors: treating a 400 on the second bind as a platform fault and then deleting the link; changing quantity during bind to record a goods receipt — bind does not change menge, book a stock_putaway in phase 5; locking by deleting the object, so a later scan of the found sticker could be bound to something else if the code were reissued.

## Phase 3 — staff accounts, scan, start card, short URL

The goal is that a staff member can bind a free label to a package with a single scan, the start card leads them to the scan page, and the short URL hides the object name until sign-in. Staff accounts, invitations and transfers follow the platform company-manager file and the staff-account recipe in shift-planning phase 3. The master person.email must equal the login.

Rights are as in phase 0. label_bind for staff lives in the team scope, with result_scope_id equal to the warehouse scope, scan.ein_code, the package prompt, and a guard if only warehouse staff, not kitchen, may bind. Give the staff account a small credit balance with ontouser_team_transfer before the first scan. A 402 is not a bind failure of the skill if you top up at once; log it.

The bundled scan page is GET /ontouser/scan, role member. The contract is HTTP, not MCP. See the platform scan chapter for ein_code, prompts, blockiert, dedupe_ms and feedback. The staff action in this phase is label_bind after a single scan of a free label.

Register a start card in the team scope with onto_startcard_save and read it back with onto_startcard_index. It should lead staff to the scan page and, if you build one, to a team view of labels they bound today, only if that view can be fed without granting warehouse read, which you confirm. A card has no roles: it is visible when the scope is readable and the target view is in the interface list. Do not point the card at a manager view that lives in the warehouse scope.

The sticker carries the public-app short URL /q/ followed by the code. Before sign-in, the object name is not shown. After sign-in, a bound, not-locked label resolves to the linked object, the package it identifies or the label that points at it — quote which one you observed. A gesperrt label must not resolve to the package.

The recipe: create and activate the staff account for M1, set rights, transfer credits, read back the team show; save the start card and quote it; as M1, open the scan page, scan a free label, confirm label_bind is offered, pick package B, execute, and read back as manager, gebunden, identifies to package B, one row later in the action log; as M1, request onto_object_show on that warehouse label, expect 404 or no access, confirm, and the bind must still have succeeded; fetch the short URL logged out for a bound label, no object name; fetch it as M1, the package is reachable; fetch it for a gesperrt label, it does not open the package.

MCP and browser checks include staff rights, the start card, the bind by M1, no direct access to the warehouse label, the start card as the first thing M1 sees, a single scan offering bind, short-URL behaviour visible and not only assumed from MCP, and manager-only buttons, print and lock, invisible to M1, with direct execution denied, which is negative test N5. Status: stage 3 of 5 complete.

Typical errors: granting warehouse read after the staff show fails; putting a warehouse manager view on the team start card; testing only the MCP execute path and skipping the camera or scan page; using a checkpoint-style QR that contains a raw UDID instead of the printed short URL.

## Phase 4 — action log, manager list or CSV

The goal is that the manager can prove who bound which label, and can export or open a list of labels, packages and movements.

The source is onto_action_logs, confirm the tool name in your catalogue. Every label_bind execution must leave a row with the action key, the label, the account and the scope. Quote at least the manager bind from phase 2 and the staff bind from phase 3. A rejected second bind must not add a successful bind row. Reports that interpret this log are ordered through the service channel. Offer them; do not try to save a report yourself, which returns 403.

A member can build a block view. If the view allows a download, enable it and open the file. Minimum columns: for labels, code, labelStatus, charge, the identified package name, printed at; for packages, name, menge, einheit, lot, bestBefore, and the count of identifying labels if the view can show it; for movements, time, movementType, menge, package, place movesTo, bookedBy, and grund when present. The package list shows the book quantity menge that the bookings maintain. Movement rows are the history of those bookings. After this phase, run phase 5 and come back to quote the booking rows in the same list or CSV.

The recipe: read the action log for label_bind and quote the two successful rows and the absence of a success row for the rejected second bind; open the manager label, package and movement views, and if a CSV is available, download it, open it, and quote one row per test package and the bound test labels, and after phase 5 quote the test movements as well; read onto_credit_balance for the manager and for M1.

Checks: action-log rows quoted, views still carry access.require, balance quoted now, and the list or CSV opens and matches the read-backs. The workspace is still not "configured" until the acceptance scenario has been run end to end.

A typical error is building a privileged report yourself instead of filing a service request, or cleaning up test labels before the log has been quoted.

## Phase 5 — stock bookings, rebind, lot and best-before

The goal is that a manager can put away, pick, relocate and count a package, hang a bound label onto another package, and read lot and best-before on the package. Every booking writes a stockMovement and updates menge in the same run. Functions and actions from phase 0 must already be installed and quoted. Call onto_credit_pricing and onto_credit_balance before the first booking and after the last. Log the keys and numbers you actually read; do not copy a figure from this skill. This phase describes the manager path. Do not add the stock actions to the team door, and do not tell the customer that staff can book through that door, unless you have measured that path in this workspace.

menge on the package is the book quantity. The functions update it in the same run as the movement. stock_putaway adds zugang; the movement stores that booked quantity and has type putaway. stock_pick subtracts zugang; the movement stores that booked quantity and has type pick. stock_relocate leaves menge unchanged because the sign is zero; the movement stores the zugang you passed and has type relocate. stock_count sets menge to the counted ist; the movement stores that counted quantity and has type count. Do not edit menge with onto_object_set_property. After a count there is no second correction row; the gap is the difference between the previous book quantity and the new menge. Put the reason in grund. Each movement also stores bookingTime and bookedBy, the signed-in user's e-mail. Count also stores grund when prompted. The place from the prompt is written as movesTo. The movement is linked to the package as moves.

Each stock action has scan.ein_code and automatic execution. One code identifies the package, the package itself or a bound label that resolves to it. The action then prompts for quantity and, where required, the storage location, object_types storage, the client fills the UDID.

On the package, lot is a string and bestBefore is a date. Set them when the package is created or when the intake says to fill them, and quote them on read-back. They do not hang on the label or on the movement.

label_rebind conditions on labelStatus gebunden, prompts for paket, the new package, replaces identifies with replace source, leaves status gebunden, and reports the removed edge as links_entfernt. Install the action with require_scope_write on the warehouse scope so only leadership can run it.

The recipe: read prices and balance and save them with a timestamp; pick a test package with the TEST-5- prefix and two storage locations, set lot and bestBefore if empty, and quote the starting menge; as manager, run stock_putaway with a required place and a quantity, and read back the package, menge increased, and the new movement, type, menge, bookedBy, bookingTime, moves and movesTo; run stock_pick with a quantity, place optional, and read back menge decreased and movement type pick; run stock_relocate to the second place, pass the current book quantity as zugang, and read back package menge unchanged, movement type relocate, movesTo the new place; run stock_count with a counted quantity and a reason, optionally a place, and read back package menge equal to ist, one movement of type count with grund, and no extra correction row; take a bound test label, or bind a free one first, run label_rebind onto a second test package, and read back the label still gebunden, identifies pointing at the new package, the old package no longer having that edge, and the execute result listing links_entfernt; quote the credit balance again.

MCP checks: package menge after the four bookings matches the table; four stockMovement objects exist for the test package, with moves and movesTo as booked; lot and bestBefore are present on the test package when you set them; after rebind, onto_object_show on the label and onto_object_graph on both packages confirm the new identifies edge only. If a stock action returns 403 and the message says the function is not allowed in this scope, stop, fix the function's scopes, and do not grant warehouse rights to staff as a workaround.

In the browser, the four stock buttons appear on a package for the manager and are hidden behind require_scope_write. After a booking, the package quantity and a movement row are visible in the manager list or detail. label_rebind is available to the manager on a bound label and is not offered as a staff start-card action.

You may report "stage 5 of 5" only when the MCP checks above are quoted and the acceptance steps 5.1 to 5.8 have passed. Combined with phases 0 to 4, that is the end-to-end scenario.

Typical errors: pointing stock_putaway at a function registered only in another scope, HTTP 403, no movement; omitting zugang on relocate so the function rejects a missing required input — pass the current book quantity; setting menge by hand after a count to make it tidy and leaving no movement; running label_rebind without require_scope_write; claiming staff can book through the team door without a measured test in this workspace.

## Operating model

This is how an agent oversees the warehouse this skill sets up.

Goods sit in packages. Each package has a book quantity, a unit, and, when the business uses them, a lot and a best-before date. A printed label is only a pointer: its code binds to at most one package. Staff identify a package by scanning the sticker. Managers book quantity with a single scan of that package, or of a bound label that resolves to it. The action then prompts for the quantity and, where required, the storage location.

To identify the package, print a charge of labels, stick them on, and bind a free label to a package with a single scan. Quantity and unit sit on the package. Several labels may point at the same package. To lock a lost or replaced sticker, set labelStatus to gesperrt; the code must no longer resolve. Put-away books inbound quantity onto the package; menge increases by the booked amount; a stockMovement of type putaway is written, with moves to the package and movesTo to the place. Pick, or issue, books outbound quantity; menge decreases by the booked amount; a movement of type pick is written. Relocate names the new storage location; menge stays the same; a movement of type relocate is written, with movesTo on the new place. Count, or inventory, enters the counted quantity and a reason; menge is set to that quantity; a movement of type count is written; there is no second correction row; the gap is the difference between the previous book quantity and the new menge. Rebind lets leadership hang a bound label onto another package; identifies is replaced; the execute result lists the removed edge; status stays gebunden.

The package list a manager opens is the book quantity per package, plus the movement rows linked to it. That is the stock this skill maintains.

The agent's role: on a schedule agreed in the intake, list packages and quote menge, einheit, lot, bestBefore, name and site; list labels by labelStatus and charge; quote values from a read-back. Walk the moves links of a package and quote type, quantity, time, person, place and reason. Flag packages whose bestBefore falls inside an agreed window, or is already past, using the dates you read on the packages. After a count, compare the new menge with the previous book quantity and name the difference. Escalate to a human when a stop rule fires, when data is missing, or when a proposed action needs a decision. Leave the scan to people. Staff bind labels through the scan page. Managers book putaway, pick, relocate and count on the manager path. The agent does not pretend a tool write is a scan.

The control loop is measure, assess, act, check. Measure: list packages and labels, read a bound label's identifies edge, read movements on a package, read label_bind and stock-action rows in the action log. Assess: notice missing menge, a surprising number of frei or gesperrt labels, a best-before date inside the agreed window, or a count that differs from the previous book quantity. Act: print, bind, rebind and lock labels as the workspace allows; book putaway, pick, relocate or count as a manager; report findings to the human; do not edit menge to correct stock. Check: read back every write, label status, identifies, package fields, movement, action log.

Indicators a member can quote: the number of package objects and menge per package, which is the book quantity the bookings maintain; counts of frei, gebunden and gesperrt labels, plus labels per charge; successful label_bind rows, who, when, which label; the stockMovement rows linked with moves, with type, quantity, place, time and person; the count gap, previous book quantity versus the counted menge; and a best-before watch list, packages whose bestBefore is inside the agreed window, read from the package field. A manager list or CSV can show these columns. A formal report or chart is ordered through the service channel, not built by a member.

The agent never invents a label code or prints a QR that encodes a UDID; never puts menge on the label, and never edits menge on the package by hand; never grants staff warehouse read so a scan or a picker works; never tells the customer that staff can book putaway, pick, relocate or count through the team door unless that path has been measured in this workspace; never installs a stock action whose function is not allowed in the package's scope; never leaves label_rebind without require_scope_write; never places a supplier order, and never presents a package list as a legally required stock book; and never reports completion without quoting the checks in the skill and the acceptance scenario.

## Glossary, spoken briefly

The team door is the team scope. Staff hold read and write there, read and no write on master, and no membership on the warehouse scope. Every view a staff member opens and every button they press lives in the team scope; the bind writes back through a scope bridge. Stock bookings in this skill are run on the manager path.

A label is a printed sticker with its own code and charge, a pointer, not the stock, with status frei, gebunden or gesperrt. Several labels may point at the same package; each label points at at most one package. The code is the platform-assigned Crockford Base32 value, not the object UDID, printed as the public-app short URL, in groups of four. A charge is the batch identifier shared by the labels of one print sheet; a reprint of the same charge must not create new labels.

A package is the physical unit and carries menge, the book quantity, einheit, lot and bestBefore. Change menge only by booking a movement. Storage is a physical slot with a description. Bookings record the place on the movement link movesTo.

A stock movement is one booking row, with movementType putaway, pick, relocate or count, menge as the booked or counted quantity, bookingTime, bookedBy, and grund on a count, plus the links moves and movesTo. The type also has einheit and movesFrom; quote them when the read-back shows them.

identifies is the bind link, set by label_bind and moved by label_rebind. label_bind is the single-scan action on a free label. label_rebind hangs a bound label onto another package and reports the removed edge. The stock actions are the four single-scan keys on a package; they call stock_move_apply or stock_count_apply in the same scope as the package. The warehouse scope holds labels, print files, movement functions, stock actions and manager views; staff are not members of it. The preset lager is the catalogue bundle of package, storage, label, person and file; after the preset, install stockMovement, the two functions and the stock actions as phase 0 describes.

## Country module, Germany

Use this when intake 18 is German. It sets display labels only. Keys, option values and tool names stay as in the rest of the skill. This skill does not give legal advice.

Suggested labels: Etikett for label, Gebinde for package, Lagerplatz for storage, Lagerbewegung for stockMovement, Etikett-Status for labelStatus, Frei, Gebunden and Gesperrt for the three status values, with the values themselves unchanged, Code, Charge in brackets Druckcharge, Menge, Einheit, Charge in brackets Los for lot, Mindesthaltbar bis for bestBefore, Art der Bewegung for movementType, Einlagern, Entnehmen, Umlagern and Inventur for the four movement types, with the English values unchanged, Grund, Zeitpunkt, Gebucht von, Etikett binden, Etikett umhängen, and Inventur zählen for the count action.

The sheet is Avery Zweckform 3475. The charge id and the readable code stay on the sheet so staff can talk about a sticker without opening the app. A list of packages with menge is the book quantity per package, the figure the bookings maintain. Keep food-hygiene or customs records in the form the business already uses; do not treat the package list as a substitute for a statutory stock book unless the customer has decided that with their advisor.

## Acceptance scenario

This is one continuous test with fixed, fictional start data and expected values. A fresh agent runs it once from an empty account, or from an existing DemoRestaurant shift-planning master if the intake says to reuse it. The paid run is measured with measurement.md. Every step names the role that performs it and the expected result; a step passes only when the result is read back, tool read or browser, not when the write call returned OK.

The scenario has thirty-eight steps, 0.1 to 5.8, and fourteen negative tests, N1 to N14. Both counts are stated only in the acceptance file and, pointing at that file, in SKILL.md and definition-of-done.md. Do not restate them from memory.

The business is the fictional DemoRestaurant, slug demorestaurant, with scopes demorestaurant_master, demorestaurant_warehouse and demorestaurant_team, reusing master and team if they already exist. All test objects carry the prefix TEST-A-. All addresses in the document are at example.com. These are not activatable human mailboxes. Before login tests, obtain explicit permission for dedicated controlled test mailboxes or another supported active test-account route, and align their master person e-mails with those logins. If only example.com data is permitted, keep the synthetic fixtures and mark staff-login tests blocked. Never use real staff data. Role L is the manager or warehouse lead, warehouse.lead at example.com. Role M1 is warehouse staff, warehouse.staff at example.com. Times are entered in Europe/Berlin; tool reads return UTC; compare after conversion. Read the balance before and after each phase.

Start data: organisation TEST-A-DemoRestaurant, reuse if present; one building, TEST-A-DemoRestaurant Main Street; two storage locations, TEST-A-LOC-Kitchen-01 and TEST-A-LOC-Cellar-01, with description set. People: TEST-A-Warehouse Lead and TEST-A-Warehouse Staff, with the example.com logins. Packages: TEST-A-PKG-Flour-10kg, menge 10, unit kg, lot TEST-A-LOT-FLOUR, best-before the thirtieth of June 2027; and TEST-A-PKG-Oil-6, menge 6, unit piece, lot TEST-A-LOT-OIL, best-before the thirty-first of March 2027. One charge TEST-A-CHARGE-01 of free labels, a full twenty-four-up Avery 3475 sheet, or a smaller quoted test charge if that is what the platform issued. Codes are assigned by the platform.

A step passes only with a read-back. Mark "confirm" where the live path, print key, short-URL target, staff 404, must be quoted from this workspace.

Phase 0. Step 0.1, L checks health or skips it, auth_status, and the tool list; expected, member, every named tool present. 0.2, L reads pricing and balance; the full table is saved with a timestamp and the balance covers phase 0. 0.3, L shows the types label, package, storage, stockMovement, person, organisation, building and file; the fields of skill check 3 are quoted. 0.4, L reads the link index; identifies is label to package and unique; moves and movesTo are present. 0.5, L creates or reuses the three scopes; the index lists the three suffixes and you can manage all three. 0.6, L activates types, installs the two functions with the package scope in scopes, and installs the six actions; types, functions and actions are quoted, and label_rebind has require_scope_write. 0.7, L confirms labelStatus options are frei, gebunden and gesperrt, not replaced. 0.8, L sets the rights matrix; the team show matches phase 0. 0.9, M1 or a staff test opens the warehouse scope and gets 403 or no access; master read is allowed.

Phase 1. 1.1, L reuses or creates organisation and building; objects read back, bestehtAus present. 1.2, L reuses or creates persons L and M1; emails read back, lower case. 1.3, L creates the two storage locations; both read back with description. 1.4, L creates the two packages; menge, einheit, lot and bestBefore match the start data. 1.5, L creates the charge; the count is quoted, at least five codes match Crockford twelve plus check, every sampled label is frei and has no menge. 1.6, L prints the charge and opens the PDF; the 3475 layout is present, or the quoted test count with the same typography. 1.7, L builds manager views; label, package, storage and movement lists exist, access.require set.

Phase 2. 2.1, L binds one free label to the flour package; status gebunden, identifies to that package, menge still 10. 2.2, L binds three further labels to the same package; four labels identify it. 2.3, L binds again on a bound label; HTTP 400, precondition not met, still one identifies edge, status still gebunden. 2.4, L locks one label; status gesperrt, resolve no longer opens the package, which may be finished in phase 3 if the browser is only then available. 2.5, L reprints the charge; the label count is unchanged and the PDF still shows the same codes, spot-check three.

Phase 3. 3.1, L activates M1, sets rights, transfers credits; the team show is quoted and M1 has a balance. 3.2, L saves the team start card; the index shows the scan entry. 3.3, M1 single-scans a free label, runs label_bind, picks the oil package; status gebunden, identifies to that package, menge still 6. 3.4, anyone logged out opens the short URL of a bound label; no object name is shown. 3.5, M1 opens the same short URL after sign-in; the linked package is reachable, quote what opened. 3.6, M1 shows the warehouse label UDID; 404 or no access, confirm, and the bind in 3.3 remains.

Phase 4. 4.1, L reads the action log for label_bind; a success row for 2.1 and for 3.3, no success row for 2.3. 4.2, L opens the manager list or CSV; one row per test package, bound labels show gebunden and the package name, the locked label shows gesperrt. 4.3, L reads the credit balance for L and M1; balances quoted now.

Phase 5, starting book quantities before 5.2: flour 10, oil 6. Keep a written running quantity in the run log. 5.1, L reads lot and bestBefore on both packages, and fills them if empty; values as in the start data. 5.2, L puts away 5 kilograms of flour to the kitchen location; package menge is 15; one movement type putaway, menge 5, moves to flour, movesTo to the kitchen, bookedBy and bookingTime set. 5.3, L picks 3 kilograms from flour, kitchen place optional; package menge is 12; one movement type pick, menge 3. 5.4, L relocates flour to the cellar, passing zugang 12; package menge still 12; one movement type relocate, movesTo to the cellar. 5.5, L counts flour, ist 11, reason TEST-A count, place the cellar; package menge is 11; one movement type count, menge 11, grund set; no second correction row. 5.6, L rebinds one flour-bound label onto the oil package; the label stays gebunden, identifies points at oil, flour no longer has that edge, and the execute result lists links_entfernt. 5.7, L confirms the four stock actions and both functions, each function's scopes include the package scope; the lists are quoted; a mismatch would have failed 5.2 with HTTP 403. 5.8, L reads pricing and balance again; live keys and balances quoted now.

Negative tests, run as named tests even when a numbered step already covered the same fact. N1, L, a second label_bind on a bound label, 400, no change, same as 2.3. N2, L or M1, resolve or scan a gesperrt label, it does not open the package. N3, M1, scan a code that is not a platform label code, no action offered. N4, M1, membership or direct read on the warehouse scope, 403 or 404, no grant is added. N5, M1, a manager-only print or lock action, the button is invisible, direct execute is denied. N6, L, reprint the same charge, no new label objects. N7, L, read a label for menge and einheit, those properties are absent on the label and present on the package. N8, logged out, the short URL of a bound label, no object name, same as 3.4. N9, L, run a stock action whose function is registered only in another scope, or quote the 403 from a first mis-install, then fix and continue; expected HTTP 403, the platform message that the function is not allowed in this scope, menge unchanged, no new movement. N10, M1, bind with a zero-credit staff account, HTTP 402, no bind, top up only after the test is logged. N11, L, the label_rebind definition has require_scope_write, and the action is not on the staff start card. N12, M1, the start card does not open a warehouse-scope manager view. N13, L, after relocate in 5.4, package menge equals the quantity before the relocate. N14, L, after count in 5.5, only one new movement, type count, no extra correction row.

Clean-up: keep TEST-A- objects by default and list them in the run log. If the intake asks you to delete them, delete one id at a time, budget the delete price, and never delete by prefix. Do not move test objects into a live customer scope that already serves real staff.

## Measurement of a paid test run

This is how a fresh agent's build of this skill on a pay-per-use account is measured. The run follows the acceptance scenario. One run is a sample; claim "an agent builds this alone" only after a second run by another agent with comparable numbers.

Preconditions: onto_credit_balance shows billing_model is pay-per-use. Flat accounts book nothing; a run on them measures nothing. The credit cap is approved before the start, default twice the estimate you derive from the live price table. Reaching the cap means stop and ask. The run id is the date, the agent name and a number, plus the object prefix TEST-A-; every log row carries the run id. The price table is onto_credit_pricing read at the start, saved with date and time, Europe/Berlin. That table wins. All times are in Europe/Berlin with a zone label; tool times in UTC are converted. The fresh agent has no memory of earlier runs; it gets only this skill, the platform skill, the shift-planning skill for reuse, and the intake answers.

Stop criteria: three consecutive failed attempts at the same step, stop and ask, log the step. Credit cap reached, or a single phase above twice its estimate, stop and ask. Any stop point of the skill, missing type, missing print path, platform fault, unclear intake, stop, file the service request, log the wait. HTTP 402 on a staff account is not a failure of the agent if the top-up follows at once; log it as an intervention of kind credits only if a human had to act. N10 is an intentional 402 and must be logged as a passed negative test, not as a cap event.

Read onto_credit_balance of the manager account and every staff account before and after each phase. The difference is the phase cost. Also count the calls by price key to explain the difference. Estimates are order-of-magnitude only, because reuse of an existing master or team door changes the scope price, and because print and booking keys must be taken from the live table. Derive a cap from that table before you start; then fill the actuals.

Phase 0's main costs are new scopes at the volume price, plus reads. Phase 1's are people, storage, packages and labels as create_object, and print as action and or file_upload, which you confirm; a twenty-four-up sheet is twenty-four label objects plus print. Phase 2's are several action binds, one lock, one reprint. Phase 3's are account setup, transfers, a staff action bind, and reads; transfers are moved, not spent, log both. Phase 4's are reads, maybe a file download, small. Phase 5's are four stock bookings and one rebind, billed as action and whatever the functions book, which you confirm. Set the cap at twice the sum you calculated from the live table. Log operating costs of a live workspace only after you have measured a pay-per-use period; do not invent a monthly figure. The log template has one row per phase and account, with balance before and after, spent, calls by key, estimate, difference to estimate, and a note, including a row for phase 5 on the manager account.

Log wall-clock time per phase, Europe/Berlin, waits, for example an invitation e-mail, and every human intervention, kind credits, intake, login, print, scan, or other. A scanned bind that the agent could not perform because it has no camera is an intervention of kind scan, not a failed step, if a human ran the scan and the agent quoted the read-back.

After the run, quote total spent, cap, number of accepted steps of thirty-eight, number of passed negative tests of fourteen, interventions, and whether existing shift-planning scopes were reused. Do not publish live balances, account names or identifiers.

## Tool traps

Read the platform list first. This skill only adds traps that showed up while describing warehouse labels and stock bookings.

One. The label code is not the UDID. Inventing a code, or printing a QR that encodes the object id, builds a sticker the scan page will not bind the way this skill describes. Let the platform assign code. Verify Crockford Base32, twelve characters plus a check digit, no I, L, O or U, on the read-back.

Two. Quantity lives on the package. Writing menge onto the label, or treating a bind as a goods receipt, creates a second source of truth. Bind only sets identifies and labelStatus. Change menge only through the stock actions.

Three. Do not rename frei, gebunden or gesperrt. label_bind and label_rebind are proven against those values. English values look neater and break the condition.

Four. A function in the wrong scope looks like a rights problem. If stock_putaway, or pick, relocate or count, returns HTTP 403 with the message that the function is not allowed in this scope, the function's scopes list does not include the package's scope. Align the function; do not grant extra account rights.

Five. The platform scan chapter also describes other scan recipes. Use that chapter for the HTTP contract, ein_code, prompts and require. Use this skill for labels and for the single-scan stock actions.

Six. Checkpoint codes and warehouse codes are different features. Checkpoint rounds still document a QR that contains a raw UDID. Warehouse stickers print the short URL. Do not mix the two.

Seven. Do not keep a second current place next to the movement. The booking records the place on movesTo. Leave the package field place alone; do not also write a concerns slot link as a second truth.

Eight. A reprint of the same charge must not be treated as "print more labels". Count first, print, count again.

Nine. A second bind is a rejected precondition, not a bug. To hang the sticker on another package, use label_rebind. Do not delete identifies by hand to fix a second bind.

Ten. Scan is HTTP, not MCP. There is no tools/list entry for match_actions. Driving the scan only through onto_object_execute_action skips the page that staff will actually use; still run one real scan.

Eleven. The short URL hides the name before sign-in on purpose. Do not try to fix that with a public scope or an unauthenticated view.

Twelve. Staff 404 on a warehouse label is fail-closed, not a failed bind. Check the graph as manager before you grant extra rights.

Thirteen. remove true is still the general unlink trap. If you ever detach a link, use remove pair. label_rebind already replaces the edge.

Fourteen. Prices come from onto_credit_pricing today. A table in another skill or in yesterday's log is not the price. Quote the live key and the number you read; do not copy a figure from this skill.

Fifteen. ontouser_scope_types_set replaces the list. Activating label or stockMovement without including the types already on the scope will drop them.

Sixteen. Relocate still needs a quantity input for the function. The sign is zero, so menge on the package stays the same; the movement stores the quantity you passed. Pass the current book quantity, or another agreed figure, and read both values back.

Seventeen. Count sets the book quantity absolutely. There is no second correction row. The gap is the difference between the previous movement's resulting stock and the new menge. Put the reason in grund.

## Publication check, in brief

Before every push of this skill to the public repo, run the publication check. There must be no identifiers of a tested build, no scope numbers or object ids except placeholders such as the warehouse scope id in angle brackets, no e-mail addresses except example.com, no absolute paths of a build machine, no ticket, comment or notification numbers, no names of real people, agents or a reference business other than the fictional DemoRestaurant, no platform internals such as server files, line numbers, database tables or tool counts, no leftover draft markers, no German display labels or country-specific rules outside the country module, except the immutable platform values frei, gebunden and gesperrt, and no build-history wording for customers. All relative links must resolve. Private name and id lists, run logs, PDFs of a real charge, screenshots with live codes, and any MCP responses that contain identifiers stay outside the published tree. The line "Verified against platform schema" in SKILL.md must carry the date of the last schema read, which for this version is the thirteenth of November 2026.

That is the end of the warehouse-logistics skill, version 1.0.
