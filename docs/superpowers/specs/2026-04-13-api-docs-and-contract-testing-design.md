# Design: API Docs Website & Automatisiertes Contract Testing

**Datum:** 2026-04-13  
**Status:** Approved

## Ziel

1. Die OpenAPI-Spezifikation (`basketball-bund-net-api-V1.yaml`) auf einer interaktiven Website (GitHub Pages) veröffentlichen, damit die API direkt im Browser ausprobiert werden kann.
2. Täglich automatisiert prüfen, ob die echte `basketball-bund.net` API noch mit der OAS übereinstimmt — und bei Abweichungen sofort benachrichtigt werden.

## Architektur

Zwei unabhängige GitHub Actions Workflows:

### 1. `deploy-docs.yml` — Swagger UI auf GitHub Pages

- **Trigger:** Push auf `main`
- **Was passiert:** Eine statische `docs/index.html` (Swagger UI via CDN) referenziert die `basketball-bund-net-api-V1.yaml` direkt im Repo. GitHub Pages served beides.
- **Kein Build-Step nötig** — reine statische Dateien.
- **CORS:** Die `basketball-bund.net` API scheint in der Praxis ohne Proxy zu funktionieren (aus vergangenen Projekten bekannt). Falls "Try it out" im Browser nicht funktioniert, wird ein CORS-Proxy evaluiert.

### 2. `api-contract-check.yml` — Schemathesis Validierung

- **Trigger:** Täglich via Cron (`schedule`)
- **Was passiert:**
  - Installiert Schemathesis per `pip`
  - Führt Schemathesis gegen `https://www.basketball-bund.net` mit der lokalen OAS aus
  - Positive Tests: Nutzt dokumentierte `examples` aus der OAS
  - Negative Tests: Sendet ungültige Inputs, prüft ob die API korrekt mit Fehler-Responses antwortet
- **Bei Fehler:** GitHub schickt automatisch eine E-Mail (Standard GitHub-Benachrichtigung für fehlgeschlagene Workflows)
- **Status-Badge** im README zeigt den letzten Check-Status (grün/rot)

### Schemathesis-Konfiguration (`schemathesis.toml`)

- Base-URL: `https://www.basketball-bund.net`
- Timeouts angepasst (externe API kann langsam sein)
- POST-Endpoints werden über OAS-Beispiele abgedeckt (3 POST-Endpoints, alle nicht kritisch)
- Ggf. einzelne Endpoints ausschließen falls sie im CI-Kontext nicht sinnvoll testbar sind

## Dateistruktur

```
.github/
  workflows/
    deploy-docs.yml
    api-contract-check.yml
docs/
  index.html              ← Swagger UI (referenziert YAML via relativen Pfad)
schemathesis.toml         ← Schemathesis-Konfiguration
README.md                 ← Badge für Contract-Check-Status ergänzen
basketball-bund-net-api-V1.yaml  ← bleibt wo sie ist
```

## Benachrichtigungen & Sichtbarkeit

- **E-Mail:** GitHub sendet automatisch eine Mail bei fehlgeschlagenen Workflows (in GitHub-Account-Einstellungen unter "Notifications" aktivierbar)
- **README-Badge:** Workflow-Status-Badge direkt im README, sodass jeder der das Repo besucht sofort sieht ob die API noch konsistent ist
- **GitHub Actions Dashboard:** Vollständige Logs und Historie aller Checks

## Nicht im Scope

- Automatisierte Endpoint-Discovery (Fuzzing/Scanning nach unbekannten Endpoints) — separates Feature, manuell ausgeführt wenn konkrete Patterns vermutet werden
- Eigener CORS-Proxy (erst evaluieren wenn "Try it out" nicht funktioniert)
- Migration/Ersatz der bestehenden Pact-Tests
