# API Docs Website & Automatisiertes Contract Testing — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Swagger UI auf GitHub Pages deployen und täglich per Schemathesis automatisiert prüfen, ob die echte basketball-bund.net API mit der OAS übereinstimmt.

**Architecture:** GitHub Pages served eine statische `docs/index.html` mit Swagger UI (CDN), die die vorhandene `basketball-bund-net-api-V1.yaml` direkt referenziert. Ein zweiter GitHub Actions Workflow läuft täglich, installiert Schemathesis per pip, und testet die echte API gegen die OAS. Status-Badge im README zeigt den letzten Check-Status.

**Tech Stack:** Swagger UI (CDN, kein Build), GitHub Actions, Schemathesis (Python), TOML-Konfiguration

---

## Dateistruktur

```
.github/
  workflows/
    deploy-docs.yml          ← NEU: deployt docs/ auf GitHub Pages bei Push auf main
    api-contract-check.yml   ← NEU: führt Schemathesis täglich aus
docs/
  index.html                 ← NEU: Swagger UI HTML-Seite
schemathesis.toml            ← NEU: Schemathesis-Konfiguration
README.md                    ← MODIFY: Badge + Swagger UI Link ergänzen
```

---

## Task 1: Swagger UI HTML-Seite erstellen

**Files:**
- Create: `docs/index.html`

- [ ] **Step 1: `docs/index.html` anlegen**

Erstelle die Datei mit folgendem Inhalt (Swagger UI via CDN, referenziert die YAML im Repo-Root über einen relativen Pfad):

```html
<!DOCTYPE html>
<html lang="de">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Basketball-Bund.net API</title>
    <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist@5/swagger-ui.css" />
  </head>
  <body>
    <div id="swagger-ui"></div>
    <script src="https://unpkg.com/swagger-ui-dist@5/swagger-ui-bundle.js"></script>
    <script>
      SwaggerUIBundle({
        url: "./basketball-bund-net-api-V1.yaml",
        dom_id: "#swagger-ui",
        presets: [SwaggerUIBundle.presets.apis, SwaggerUIBundle.SwaggerUIStandalonePreset],
        layout: "BaseLayout",
        deepLinking: true,
        tryItOutEnabled: true,
      });
    </script>
  </body>
</html>
```

- [ ] **Step 2: YAML-Datei in `docs/` kopieren**

GitHub Pages served nur den `docs/`-Ordner. Die YAML muss also dort liegen, damit der relative Pfad `./basketball-bund-net-api-V1.yaml` funktioniert:

```bash
cp basketball-bund-net-api-V1.yaml docs/basketball-bund-net-api-V1.yaml
```

**Wichtig:** Die Originaldatei im Repo-Root bleibt — die Kopie in `docs/` ist nur für Pages. Das bedeutet: bei Änderungen an der YAML muss auch `docs/basketball-bund-net-api-V1.yaml` aktualisiert werden. Der Deploy-Workflow (Task 2) übernimmt das automatisch.

- [ ] **Step 3: Lokal prüfen (optional)**

```bash
cd docs && python3 -m http.server 8080
```

Browser öffnen: `http://localhost:8080` — Swagger UI sollte die API-Dokumentation anzeigen.

- [ ] **Step 4: Commit**

```bash
git add docs/index.html docs/basketball-bund-net-api-V1.yaml
git commit -m "feat: add Swagger UI for GitHub Pages"
```

---

## Task 2: GitHub Actions Workflow — Swagger UI deployen

**Files:**
- Create: `.github/workflows/deploy-docs.yml`

- [ ] **Step 1: Workflow-Datei anlegen**

```bash
mkdir -p .github/workflows
```

Erstelle `.github/workflows/deploy-docs.yml`:

```yaml
name: Deploy Swagger UI to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Copy YAML to docs/
        run: cp basketball-bund-net-api-V1.yaml docs/basketball-bund-net-api-V1.yaml

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: "docs"

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: GitHub Pages im Repository aktivieren**

Im GitHub Repository:
1. `Settings` → `Pages`
2. Source: `GitHub Actions` auswählen (nicht `Deploy from a branch`)
3. Speichern

- [ ] **Step 3: Commit und pushen**

```bash
git add .github/workflows/deploy-docs.yml
git commit -m "feat: add GitHub Actions workflow for Swagger UI deployment"
git push origin main
```

- [ ] **Step 4: Deployment prüfen**

Im GitHub Repository → `Actions` → Workflow `Deploy Swagger UI to GitHub Pages` sollte grün sein. Die URL ist `https://<username>.github.io/<repo-name>/`.

---

## Task 3: Schemathesis-Konfiguration erstellen

**Files:**
- Create: `schemathesis.toml`

- [ ] **Step 1: `schemathesis.toml` anlegen**

Schemathesis liest diese Datei automatisch wenn sie im Repo-Root liegt:

```toml
[schemathesis]
# Basis-URL der echten API
base-url = "https://www.basketball-bund.net"

# Maximale Wartezeit pro Request (externe API kann langsam sein)
request-timeout = 30

# Anzahl generierter Testfälle pro Endpoint (niedrig halten für externe API)
max-examples = 10

# Nur dokumentierte Beispiele für POST-Endpoints verwenden
hypothesis-phases = ["explicit"]

# Beide Test-Modi: positive (OAS-Beispiele) und negative (invalide Inputs)
checks = ["not_a_server_error", "status_code_conformance", "content_type_conformance", "response_schema_conformance"]
```

**Hinweis zu `hypothesis-phases = ["explicit"]`:** Das bedeutet Schemathesis nutzt ausschließlich die in der OAS definierten `examples`. Dadurch werden keine zufälligen Requests generiert, was für eine externe API die man nicht kontrolliert deutlich sicherer ist.

- [ ] **Step 2: Schemathesis lokal testen**

```bash
pip install schemathesis
schemathesis run basketball-bund-net-api-V1.yaml --base-url https://www.basketball-bund.net
```

Erwartete Ausgabe: Liste der getesteten Endpoints mit PASSED/FAILED Status. Falls Fehler auftreten die auf Diskrepanzen in der OAS hinweisen (nicht auf echte API-Fehler), die OAS entsprechend korrigieren.

- [ ] **Step 3: Bei Bedarf Endpoints ausschließen**

Falls bestimmte POST-Endpoints im CI-Kontext nicht sinnvoll testbar sind (z.B. weil sie Schreiboperationen auslösen), können sie in `schemathesis.toml` ausgeschlossen werden:

```toml
[schemathesis]
# ... bestehende Konfiguration ...
exclude-path = "/rest/wam/data"  # Beispiel, nur wenn nötig
```

**Erst ausschließen wenn ein konkretes Problem auftritt.**

- [ ] **Step 4: Commit**

```bash
git add schemathesis.toml
git commit -m "feat: add schemathesis configuration for contract testing"
```

---

## Task 4: GitHub Actions Workflow — Täglicher API-Check

**Files:**
- Create: `.github/workflows/api-contract-check.yml`

- [ ] **Step 1: Workflow-Datei anlegen**

Erstelle `.github/workflows/api-contract-check.yml`:

```yaml
name: API Contract Check

on:
  schedule:
    # Täglich um 08:00 UTC
    - cron: "0 8 * * *"
  workflow_dispatch:

jobs:
  contract-check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install Schemathesis
        run: pip install schemathesis

      - name: Run Schemathesis against live API
        run: |
          schemathesis run basketball-bund-net-api-V1.yaml \
            --base-url https://www.basketball-bund.net \
            --report=junit \
            --output-file=schemathesis-report.xml

      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: schemathesis-report
          path: schemathesis-report.xml
```

- [ ] **Step 2: E-Mail-Benachrichtigungen in GitHub aktivieren**

In den GitHub Account-Einstellungen:
1. `Settings` → `Notifications`
2. Unter `GitHub Actions` → `Failed workflows only` aktivieren
3. E-Mail-Adresse verifizieren

Bei fehlgeschlagenem `API Contract Check` Workflow kommt automatisch eine Mail.

- [ ] **Step 3: Workflow manuell triggern zum Testen**

```bash
git add .github/workflows/api-contract-check.yml
git commit -m "feat: add daily API contract check via Schemathesis"
git push origin main
```

Im GitHub Repository → `Actions` → `API Contract Check` → `Run workflow` manuell ausführen und Ergebnis prüfen.

---

## Task 5: Status-Badge im README ergänzen

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Badge-URL ermitteln**

Das Badge-Format für GitHub Actions ist:
```
https://github.com/<USERNAME>/<REPO>/actions/workflows/api-contract-check.yml/badge.svg
```

Den tatsächlichen GitHub-Username und Repo-Namen einsetzen (aus der Repository-URL ablesen).

- [ ] **Step 2: README.md oben ergänzen**

Die ersten Zeilen von `README.md` anpassen — Badge und Link zur Swagger UI direkt nach der Hauptüberschrift einfügen:

```markdown
# DBB Basketball API - Dokumentation & Contract Tests

[![API Contract Check](https://github.com/<USERNAME>/<REPO>/actions/workflows/api-contract-check.yml/badge.svg)](https://github.com/<USERNAME>/<REPO>/actions/workflows/api-contract-check.yml)

**[Swagger UI — API interaktiv ausprobieren](https://<USERNAME>.github.io/<REPO>/)**

---
```

`<USERNAME>` und `<REPO>` durch die tatsächlichen Werte ersetzen.

- [ ] **Step 3: Commit und pushen**

```bash
git add README.md
git commit -m "docs: add API contract check badge and Swagger UI link to README"
git push origin main
```

- [ ] **Step 4: Badge prüfen**

README auf GitHub öffnen — der Badge sollte nach dem nächsten Workflow-Run den Status anzeigen (grün = API konsistent, rot = Abweichung gefunden).

---

## Self-Review Checklist

- [x] Swagger UI auf GitHub Pages: Task 1 + Task 2
- [x] Täglicher Schemathesis-Check: Task 3 + Task 4
- [x] E-Mail-Benachrichtigung: Task 4, Step 2
- [x] Status-Badge im README: Task 5
- [x] Keine Platzhalter (TBD/TODO) im Plan
- [x] Alle Dateinamen konsistent
- [x] YAML-Kopie in `docs/` wird vom Deploy-Workflow automatisch gemacht (kein manueller Schritt nötig nach dem ersten Mal)
