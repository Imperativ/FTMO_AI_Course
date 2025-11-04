# 📘 Kapitel 10 – Dokumentation, Versionierung und Auditierbarkeit

**Lernziel:**
Nach dieser Lektion kannst du deinen gesamten Agentenprozess so dokumentieren, dass er jederzeit reproduzierbar, überprüfbar und auditierbar bleibt.
Du lernst, wie du Versionsstände sicherst, Änderungen protokollierst und damit die Grundlage für professionelle Systembewertung schaffst.

---

## 🧩 Abschnitt 1 – Warum Dokumentation unverzichtbar ist

Sobald du beginnst, mit realen Strategien oder Testkonten zu arbeiten, reicht „Ich weiß noch, was ich geändert habe" nicht mehr aus.

**Dokumentation ist keine Bürokratie – sie ist Fehlerprävention.**

Sie schützt dich vor:

- unbeabsichtigten Logikänderungen,
- inkonsistenten Prompts,
- nicht reproduzierbaren Tests,
- und Missverständnissen bei späteren Anpassungen.

Kurz: Was dokumentiert ist, kann überprüft, verbessert oder delegiert werden.

---

## ⚙️ Abschnitt 2 – Struktur einer soliden Dokumentation

Lege in deinem Projektordner folgende Struktur an:

```
FTMO-Agent/
├── workflows/
│   ├── 01_base_flow.json
│   ├── 02_analysis_flow.json
│   └── 03_recovery_flow.json
├── logs/
│   ├── performance/
│   └── errors/
├── metrics/
│   ├── reports/
│   └── history.csv
├── docs/
│   ├── CHANGELOG.md
│   ├── VERSION.md
│   └── SYSTEM_OVERVIEW.md
└── prompts/
    ├── analyst_prompt.txt
    ├── risk_prompt.txt
    └── strategy_prompt.txt
```

Diese Struktur sorgt dafür, dass du Workflows, Logs, Metriken und Textprompts getrennt versionieren kannst.

---

## 🧠 Abschnitt 3 – Versionierung mit Git

Selbst wenn du Git bisher nur oberflächlich nutzt: Es ist das Standardwerkzeug für Nachvollziehbarkeit.

**Schritte:**

1. Initialisiere dein Repo im Hauptordner:

   ```bash
   git init
   git add .
   git commit -m "Initial commit – FTMO Agent baseline"
   ```

2. Bei jeder relevanten Änderung:

   ```bash
   git add .
   git commit -m "Update: Adjusted risk formula & Analyst prompt"
   ```

3. Optional: Tagge stabile Versionen
   ```bash
   git tag -a v1.0 -m "First stable release"
   ```

Damit kannst du jeden Zustand des Agenten exakt wiederherstellen.

**Debugging-Tipp:**
Wenn ein Flow plötzlich falsche Ergebnisse liefert, führe `git diff` aus – du siehst sofort, welcher Prompt oder Node sich verändert hat.

---

## 💡 Abschnitt 4 – CHANGELOG als zentrales Gedächtnis

Ein `CHANGELOG.md` ist kein „nice-to-have" – es ist dein Agententagebuch.

**Formatbeispiel:**

```markdown
## [1.2.0] – 2025-11-03

### Added

- Neue Risiko-Validierung für Drawdown > 3 %
- Automatisches Logging der Token-Nutzung

### Changed

- Analyst-Prompt: Trendbewertung auf 5-Stufen-Skala erweitert

### Fixed

- JSON-Parsing-Fehler bei leerer Volume-Angabe
```

Damit kannst du Wochen später genau nachvollziehen, warum sich Ergebnisse verändert haben.

---

## ⚙️ Abschnitt 5 – SYSTEM_OVERVIEW.md

Hier dokumentierst du **Architektur und Datenfluss** deines Agenten.
So kannst du ihn nicht nur selbst warten, sondern auch Dritten erklären (z. B. in Audit-Situationen).

**Empfohlener Aufbau:**

1. Überblick: Zweck und Hauptfunktionen
2. Flussdiagramm oder ASCII-Skizze
3. Rollenbeschreibung (Analyst, Risk, Strategy)
4. Datenspeicher und API-Verbindungen
5. Sicherheitsaspekte (API-Key-Verwaltung, Datenhaltung)
6. Bekannte Limitierungen

**Debugging-Hinweis:**
Verknüpfe in diesem Dokument Node-Namen mit konkreten Logs – das spart Stunden bei Fehlersuche.

---

## 🧩 Abschnitt 6 – Versionierung von Prompts

Prompts verändern sich über Zeit – und winzige Änderungen haben große Wirkung.
Behandle sie wie Code.

**Beispiel-Format (prompt_meta.json):**

```json
{
  "prompt_name": "risk_prompt.txt",
  "version": "1.3.2",
  "last_modified": "2025-11-03T22:00:00Z",
  "description": "Bewertet Trendvolatilität, liefert JSON mit risk_level & reversal_score",
  "checksum": "sha256:8a31df..."
}
```

Speichere jede Änderung mit neuem Commit.
So kannst du immer sehen, welcher Prompt bei welcher Workflow-Version im Einsatz war.

---

## ⚙️ Abschnitt 7 – Debugging und Audit-Logs

Füge in deinen n8n-Flows Audit-Informationen hinzu:

```javascript
const audit = {
  timestamp: new Date().toISOString(),
  workflow: $workflow.name,
  node: $node.name,
  version: "v1.3.2",
  user: "system-auto",
  message: "Flow executed successfully",
};
console.log(audit);
return [{ audit }];
```

Damit entsteht eine zentrale Audit-Spur, die du bei externen Prüfungen oder Systemvergleichen vorlegen kannst.

---

## 🧩 Abschnitt 8 – Praxis: Agenten-Release-Workflow

Erstelle in n8n einen Workflow namens **"Agent_Release_Manager"**, der beim Commit oder bei einem Button-Trigger Folgendes tut:

1. Prüft alle `.json`- und `.txt`-Dateien auf Änderungen (`git status`).
2. Schreibt automatisch eine neue Zeile in `VERSION.md`:
   ```
   Version: 1.3.2 – Date: 2025-11-03 – Summary: Risk module updated
   ```
3. Exportiert aktuelle Workflows (`n8n export:workflow` → `/workflows/`).
4. Sendet Telegram-Nachricht: _„Neuer Agenten-Release gespeichert."_

**Debugging-Hinweis:**
Führe einen Dry-Run (`git commit --dry-run`) aus, um sicherzustellen, dass keine sensiblen Daten (API-Keys, Logfiles) committet werden.

---

## 🧭 Abschnitt 9 – Reflexion

- Welche Komponenten deines Agenten ändern sich am häufigsten – Prompts, Logik oder Parameter?
- Wie könntest du Versionierung nutzen, um deine Experimente effizienter zu gestalten?
- Was wären Mindestanforderungen an Nachvollziehbarkeit, wenn du das System z. B. für ein Team dokumentierst?

---

## 🧩 Abschnitt 10 – Hausaufgabe / Experiment

1. Richte Git in deinem FTMO-Agent-Ordner ein.
2. Lege `CHANGELOG.md`, `VERSION.md` und `SYSTEM_OVERVIEW.md` an.
3. Versioniere deine Workflows und Prompts.
4. Führe mindestens einen Commit durch und überprüfe die Änderungsdifferenzen (`git diff`).
5. Teste deinen n8n-Release-Workflow und dokumentiere das Ergebnis.

---

## 🚨 Abschnitt 11 – Dokumentation & Versionierung Debugging

### Git-Workflow Debugging

**Problem:** Git-Commits enthalten sensible Daten
**Sicherheits-Checkliste:**

```bash
# .gitignore Template für FTMO-Agent
echo "# Sensible Daten
*.env
*.key
*_secret*
logs/sensitive/
.n8n/
node_modules/
temp/
*.log

# API Keys und Credentials
config/api_keys.json
credentials.json
.secrets/

# Persönliche Daten
personal_notes.txt
private_config.json" > .gitignore
```

**Pre-Commit Hook für Sicherheit:**

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "🔍 Checking for sensitive data..."

# Prüfe auf API Keys
if git diff --cached --name-only | xargs grep -l "api.*key\|token\|secret" 2>/dev/null; then
    echo "❌ Potential API keys or secrets found in staged files!"
    echo "Please remove sensitive data before committing."
    exit 1
fi

# Prüfe auf große Dateien
for file in $(git diff --cached --name-only); do
    if [ -f "$file" ] && [ $(stat -c%s "$file") -gt 1048576 ]; then
        echo "❌ Large file detected: $file (>1MB)"
        echo "Consider using Git LFS for large files."
        exit 1
    fi
done

echo "✅ Security checks passed"
```

### Versionierungs-Konsistenz Debugging

**Problem:** Inkonsistente Versionen zwischen verschiedenen Komponenten
**Version-Sync-Script:**

```javascript
// Automatische Versions-Synchronisation
const syncVersions = () => {
  const fs = require("fs");
  const path = require("path");

  const versionFile = path.join(__dirname, "VERSION.md");
  const packageFile = path.join(__dirname, "package.json");
  const changelogFile = path.join(__dirname, "CHANGELOG.md");

  // Lese aktuelle Version
  let currentVersion = "1.0.0";

  if (fs.existsSync(versionFile)) {
    const versionContent = fs.readFileSync(versionFile, "utf8");
    const match = versionContent.match(/Version:\s*(\d+\.\d+\.\d+)/);
    if (match) currentVersion = match[1];
  }

  // Aktualisiere package.json
  if (fs.existsSync(packageFile)) {
    const pkg = JSON.parse(fs.readFileSync(packageFile, "utf8"));
    if (pkg.version !== currentVersion) {
      pkg.version = currentVersion;
      fs.writeFileSync(packageFile, JSON.stringify(pkg, null, 2));
      console.log(`📦 Updated package.json to version ${currentVersion}`);
    }
  }

  // Validiere CHANGELOG.md
  if (fs.existsSync(changelogFile)) {
    const changelog = fs.readFileSync(changelogFile, "utf8");
    if (!changelog.includes(`[${currentVersion}]`)) {
      console.warn(
        `⚠️ CHANGELOG.md missing entry for version ${currentVersion}`,
      );
    }
  }

  return { version: currentVersion, synced: true };
};
```

### Dokumentations-Qualitätsprüfung

**Problem:** Dokumentation wird nicht aktuell gehalten
**Doc-Quality-Checker:**

```javascript
// Dokumentations-Konsistenz-Prüfung
const validateDocumentation = () => {
  const fs = require("fs");
  const issues = [];

  // Prüfe erforderliche Dateien
  const requiredDocs = [
    "README.md",
    "CHANGELOG.md",
    "VERSION.md",
    "SYSTEM_OVERVIEW.md",
  ];

  for (const doc of requiredDocs) {
    if (!fs.existsSync(doc)) {
      issues.push(`Missing required documentation: ${doc}`);
    } else {
      const content = fs.readFileSync(doc, "utf8");
      const age = Date.now() - fs.statSync(doc).mtime.getTime();
      const daysOld = Math.floor(age / (1000 * 60 * 60 * 24));

      if (daysOld > 30) {
        issues.push(`${doc} not updated for ${daysOld} days`);
      }

      if (content.length < 100) {
        issues.push(`${doc} appears to be incomplete (< 100 chars)`);
      }
    }
  }

  // Prüfe Workflow-Dokumentation
  const workflowFiles = fs
    .readdirSync("./workflows", { withFileTypes: true })
    .filter((dirent) => dirent.isFile() && dirent.name.endsWith(".json"))
    .map((dirent) => dirent.name);

  for (const workflow of workflowFiles) {
    const docFile = workflow.replace(".json", ".md");
    if (!fs.existsSync(`./docs/workflows/${docFile}`)) {
      issues.push(`Missing documentation for workflow: ${workflow}`);
    }
  }

  return {
    valid: issues.length === 0,
    issues,
    score: Math.max(0, 100 - issues.length * 10),
  };
};
```

### Audit-Trail Validierung

**Problem:** Lücken in der Audit-Spur
**Audit-Validator:**

```javascript
// Audit-Trail Konsistenz-Prüfung
const validateAuditTrail = (auditLogs) => {
  const validation = {
    gaps: [],
    duplicates: [],
    anomalies: [],
    coverage: 0,
  };

  // Sortiere Logs nach Zeitstempel
  const sortedLogs = auditLogs.sort(
    (a, b) => new Date(a.timestamp) - new Date(b.timestamp),
  );

  // Prüfe Zeitlücken
  for (let i = 1; i < sortedLogs.length; i++) {
    const currentTime = new Date(sortedLogs[i].timestamp);
    const previousTime = new Date(sortedLogs[i - 1].timestamp);
    const gap = currentTime - previousTime;

    // Mehr als 1 Stunde Lücke ist verdächtig
    if (gap > 3600000) {
      validation.gaps.push({
        from: previousTime.toISOString(),
        to: currentTime.toISOString(),
        duration_hours: gap / 3600000,
      });
    }
  }

  // Prüfe Duplikate
  const seen = new Set();
  for (const log of sortedLogs) {
    const key = `${log.timestamp}-${log.workflow}-${log.node}`;
    if (seen.has(key)) {
      validation.duplicates.push(log);
    }
    seen.add(key);
  }

  // Prüfe Anomalien in Execution-Zeiten
  const executionTimes = sortedLogs
    .filter((log) => log.execution_time_ms)
    .map((log) => log.execution_time_ms);

  if (executionTimes.length > 0) {
    const avg = executionTimes.reduce((a, b) => a + b) / executionTimes.length;
    const threshold = avg * 3; // 3x durchschnittliche Zeit

    for (const log of sortedLogs) {
      if (log.execution_time_ms && log.execution_time_ms > threshold) {
        validation.anomalies.push({
          timestamp: log.timestamp,
          execution_time: log.execution_time_ms,
          threshold,
          factor: log.execution_time_ms / avg,
        });
      }
    }
  }

  // Berechne Coverage (% der erwarteten Logs vorhanden)
  const expectedLogsPerDay = 24; // Annahme: stündliche Logs
  const daysCovered =
    sortedLogs.length > 0
      ? Math.ceil((new Date() - new Date(sortedLogs[0].timestamp)) / 86400000)
      : 0;
  const expectedTotal = daysCovered * expectedLogsPerDay;
  validation.coverage =
    expectedTotal > 0 ? (sortedLogs.length / expectedTotal) * 100 : 0;

  return validation;
};
```

---

## ✅ Zusammenfassung

Nach Kapitel 10 kannst du:

- deinen Agenten vollständig versionieren und dokumentieren,
- Änderungen transparent nachvollziehen,
- Audit-Trails führen und Debugging beschleunigen,
- Sicherheitslücken in der Versionierung vermeiden,
- und so den Grundstein für langfristige Systemqualität legen.

Im nächsten Kapitel (11) geht es um **Sicherheit, Zugriffsrechte und Deployment** – wie du deinen Agenten produktionsreif machst, ohne Risiken für Daten oder Infrastruktur.
