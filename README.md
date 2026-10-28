# Telesherpa Ontology Platform — Agent Skill

Onboarding and hands-on guide for AI agents on the **Telesherpa Ontology Platform**: objects,
scopes, actions, automation, marketplace, PWA and field service — all via MCP.

**MCP endpoint:** `https://mcp.telesherpa.com` (Streamable HTTP)

**Location:** all files live in the subfolder `telesherpa-ontology-platform/` — raw URLs without
that folder return 404.

## What is in here

```
SKILL.md                                  Entry point: onboarding, billing, capabilities, pitfalls
references/showcase-onboarding.md         Complete walkthrough (curl, step by step):
                                          invite code (agent) OR email + voucher
references/best_practices/                Recipes for typical business situations
  first-30-minutes.md                     From zero to a running system in 30 minutes
  scope-organization-per-company.md       Scope structure and permissions for a company
references/features/                      One topic per file, in depth
  tool-catalog.md                         All 177 MCP tools with roles
  auth-token-lifecycle.md                 Login, refresh, mandatory session, stateless mode
  automation-and-functions.md             Rules (triggers) + functions: read, create, run
  structured-image-ablage.md              Image categories per scope — read AND set
  pwa-and-session.md                      Web app on the home screen, persistent sign-in, push
  form-engine.md                          Forms + answers as event objects
  service-order-dispatch.md               Digital task → real field work
  marketplace-revenue-share.md            Offer your own scopes and earn
  app-user-ios-android.md                 iOS/Android app vs. web
  use-cases.md                            End-to-end verified examples
```

## Using it

**The starting point is `SKILL.md`.** It describes the onboarding route (registration, activation
by email code, login) and links a detailed reference for every topic.

**New here and unsure what the platform does for you?** →
`references/best_practices/first-30-minutes.md`. It holds the complete build in 30 minutes,
including the cost calculation.

**Which tool for what, and does my account even see it?** →
`references/features/tool-catalog.md` (all 177 tools with a role column).

**Building automation?** → `references/features/automation-and-functions.md`. Rules (triggers) and
functions can also be **created via MCP** — not just read.

**Images in categories?** → `references/features/structured-image-ablage.md`. Categories are
readable and settable via MCP; the file name has to match the `imagename` exactly.

**"Login keeps coming back" / "no push on the iPhone"?** →
`references/features/pwa-and-session.md`.

For agents that load the skill programmatically: read `SKILL.md` first, then pull in the linked
reference for the specific topic — the files are deliberately kept small.

## Versioning

The current version is in the frontmatter of `SKILL.md` and is raised on substantive changes.

## License

MIT — see `LICENSE`.
