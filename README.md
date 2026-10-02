# Telesherpa Ontology Platform — Agent Skill

Onboarding und Praxis-Anleitung für KI-Agenten auf der **Telesherpa Ontology Platform**:
Objekte, Scopes, Aktionen, Automatik, Marketplace, PWA und Feldservice — alles über MCP.

**MCP-Endpoint:** `https://mcp.telesherpa.com` (Streamable HTTP)

## Was hier drin ist

```
SKILL.md                                  Einstieg: Onboarding, Billing, Fähigkeiten, Pitfalls
references/showcase-onboarding.md         Kompletter Durchlauf (curl, Schritt für Schritt)
references/best_practices/                Rezepte für typische Geschäftssituationen
  first-30-minutes.md                     Von Null zu einem laufenden System in 30 Minuten
  scope-organization-per-company.md       Scope-Struktur und Rechte für eine Firma
references/features/                      Je ein Thema ausführlich
  tool-catalog.md                         Alle 172 MCP-Tools mit Rollen
  auth-token-lifecycle.md                 Login, Refresh, Session-Pflicht, stateless-Modus
  automation-and-functions.md             Regeln (Trigger) + Functions: lesen, anlegen, ausführen
  structured-image-ablage.md              Bildkategorien je Scope — lesen UND setzen
  pwa-and-session.md                      Web-App auf dem Home-Bildschirm, Dauer-Anmeldung, Push
  form-engine.md                          Formulare + Antworten als Event-Objekte
  service-order-dispatch.md               Digitale Aufgabe → reale Feldarbeit
  marketplace-revenue-share.md            Eigene Scopes anbieten und verdienen
  app-user-ios-android.md                 iOS/Android-App vs. Web
  use-cases.md                            End-to-end verifizierte Beispiele
```

## Nutzung

**Startpunkt ist `SKILL.md`.** Es beschreibt den Onboarding-Weg (Registrierung, Aktivierung per
E-Mail-Code, Login) und verlinkt für jedes Thema eine Detail-Referenz.

**Neu hier und unsicher, was die Plattform bringt?** → `references/best_practices/first-30-minutes.md`.
Dort steht der komplette Aufbau in 30 Minuten, inklusive Kostenrechnung.

**Welches Tool wofür, und sieht mein Konto es überhaupt?** →
`references/features/tool-catalog.md` (alle 172 Tools mit Rollen-Spalte).

**Automatik bauen?** → `references/features/automation-and-functions.md`. Regeln (Trigger) und
Functions lassen sich auch **über MCP anlegen** — nicht nur lesen.

**Bilder in Kategorien?** → `references/features/structured-image-ablage.md`. Kategorien sind
per MCP lesbar und setzbar; der Dateiname muss exakt dem `imagename` entsprechen.

**„Der Login kommt ständig" / „kein Push auf dem iPhone"?** →
`references/features/pwa-and-session.md`.

Für Agenten, die den Skill programmatisch laden: `SKILL.md` zuerst lesen, dann gezielt die
verlinkte Referenz nachladen — die Dateien sind bewusst klein gehalten.

## Versionierung

Die aktuelle Version steht im Frontmatter von `SKILL.md` und wird bei inhaltlichen Änderungen
erhöht.

## Lizenz

MIT — siehe `LICENSE`.
