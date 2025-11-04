# 📘 Kapitel 13 – Systemdossier, Export und Präsentation

**Lernziel:**
Nach dieser Lektion kannst du dein gesamtes Agentenprojekt strukturiert exportieren, nachvollziehbar dokumentieren und für Dritte verständlich präsentieren – ob als Proof-of-Concept, Portfolio-Demo oder FTMO-Bewerbungsbeilage.

---

## 🧩 Abschnitt 1 – Was ist ein Systemdossier?

Ein **Systemdossier** ist eine vollständige, lesbare Repräsentation deines Projekts – inklusive Architektur, Datenfluss, Kennzahlen, Sicherheits- und Qualitätsnachweise.
Es beantwortet drei Fragen:

1. _Was tut das System?_
2. _Wie ist es aufgebaut?_
3. _Wie zuverlässig arbeitet es?_

Damit kannst du dein Agentensystem wie ein Produkt vorstellen – nicht nur als Code.

---

## ⚙️ Abschnitt 2 – Struktur eines Systemdossiers

Erstelle einen neuen Ordner:

```
FTMO-Agent-Dossier/
├── 01_Executive_Summary.md
├── 02_System_Architecture.md
├── 03_Operational_Flow.md
├── 04_Security_and_Versioning.md
├── 05_Performance_Metrics.md
├── 06_Appendix/
│   ├── Screenshots/
│   ├── Charts/
│   ├── Logs/
│   └── Configs/
└── EXPORT_REPORT.pdf
```

Jede Datei kann automatisch aus deinen Markdown-Quellen und Logs generiert werden.

**Debugging-Hinweis:**
Wenn du beim Generieren „File not found“ bekommst, überprüfe relative Pfade (`./` vs. `../`). Viele n8n-Export-Nodes schreiben standardmäßig ins Home-Verzeichnis.

---

## 💡 Abschnitt 3 – Executive Summary (Kurzüberblick)

Schreibe hier eine kompakte Übersicht:

- Zweck und Ziel (FTMO-Strategie-Agent)
- Architektur (n8n + aktuelles LLM + TradingView + Telegram)
- Besonderheiten (Multi-Agent, Recovery-Mechanismus, Evaluations-System)
- Ergebnisse (z. B. durchschnittliche Winrate, Ausführungszeit, Stabilität)

Beispiel:

```
Der FTMO-Agent ist ein teilautomatisiertes Handelssystem,
das Marktdaten, Sentiment und Strategieregeln kombiniert.
Die Evaluations-Pipeline generiert tägliche Reports und
passt Risikoparameter adaptiv an. Alle Entscheidungen sind
durch Logs und Versions-Tags nachvollziehbar.
```

---

## 🧠 Abschnitt 4 – Systemarchitektur-Dokumentation

Beschreibe den Aufbau deines Systems:

```
+-------------+
| TradingView |───► Market Data
+-------------+
        │
        ▼
+-------------+
| n8n         |──► LLM-Analyst → Risk-Manager → Strategieberater
+-------------+
        │
        ▼
+-------------+
| Telegram Bot |  → Alerts & Reports
+-------------+
```

Füge Versions- und Abhängigkeitsinformationen hinzu:

```bash
n8n version: 1.64.0
Node.js: v18
Redis: 7.2.0
Docker image: n8nio/n8n:latest
```

**Debugging-Tipp:**
Lege `dependencies.json` an (per `npm list --json`) – so kannst du Systemstände exakt reproduzieren.

---

## 🧩 Abschnitt 5 – Export aller Workflows und Prompts

Erstelle mit n8n-CLI:

```bash
n8n export:workflow --all --output=./FTMO-Agent-Dossier/Workflows/
n8n export:credentials --all --decrypted --output=./FTMO-Agent-Dossier/Credentials/
```

Prompts:

```bash
cp ./prompts/*.txt ./FTMO-Agent-Dossier/Prompts/
```

Logs:

```bash
cp ./logs/*.json ./FTMO-Agent-Dossier/Logs/
```

**Debugging-Hinweis:**
Stelle sicher, dass keine sensiblen Schlüssel (`apiKey`, `token`) in exportierten JSONs enthalten sind. Verwende `grep` oder `rg` zur Prüfung:

```bash
grep -R "sk-" ./FTMO-Agent-Dossier
```

---

## ⚙️ Abschnitt 6 – Automatisierter PDF-Export

Nutze `pandoc` oder `pypandoc`, um Markdown zu PDF zusammenzuführen:

```bash
pandoc 01_Executive_Summary.md 02_System_Architecture.md 05_Performance_Metrics.md  -o EXPORT_REPORT.pdf --standalone --toc
```

Optionale Stilanpassung per CSS:

```bash
--css=report_style.css
```

**Debugging-Tipp:**
Wenn Grafiken fehlen: relative Pfade in `.md` prüfen (sie müssen mit `./Appendix/Screenshots/` beginnen).

---

## 🧩 Abschnitt 7 – Präsentation und Portfolio-Einbindung

Nutze dein Dossier als:

- Bewerbungs-Anhang (z. B. PDF-Summary)
- Online-Portfolio (GitHub Pages oder Notion-Embed)
- Team-Dokumentation

Beispiel-Kurzpräsentation:

```
Projekt: FTMO-Agent
Technologien: n8n, Docker, Redis, LLM Integration
Highlights: Adaptive Risikosteuerung, Multi-Agent Architektur, Self-Recovery
Ergebnis: 62 % Winrate bei 100 Trades, Durchschnittslatenz 2.3 s
```

---

## 🧩 Abschnitt 8 – Abschluss und Weiterentwicklung

Zum Abschluss:

1. Prüfe alle Module (Analyse, Risiko, Evaluation, Logging).
2. Exportiere dein vollständiges Systemdossier.
3. Lege eine finale Version (`v1.0`) in Git an.
4. Optional: öffne ein neues Branch „v2.0 – Realtime Trading Bot“.

**Debugging-Hinweis:**
Bei großen Repos (> 100 MB) Git LFS aktivieren:

```bash
git lfs install
git lfs track "*.json"
```

---

## 🧭 Abschnitt 9 – Reflexion

- Was war dein größter Lernfortschritt in diesem Kurs?
- Welche Module würdest du in Version 2.0 priorisieren?
- Welche Erkenntnisse würdest du in zukünftige KI-Projekte übernehmen?

---

## 🚨 Abschnitt 10 – Dokumentations- & Export-Debugging

### Export-Validierung

**Problem:** Exportierte Dateien sind unvollständig oder beschädigt
**Validation-Script:**

```javascript
// Dokumentations-Export Validator
const validateExport = (exportDir) => {
  const fs = require("fs");
  const path = require("path");

  const validation = {
    requiredFiles: [
      "01_Executive_Summary.md",
      "02_System_Architecture.md",
      "03_Operational_Flow.md",
      "04_Security_and_Versioning.md",
      "05_Performance_Metrics.md",
    ],
    missing: [],
    corrupted: [],
    warnings: [],
  };

  for (const file of validation.requiredFiles) {
    const filePath = path.join(exportDir, file);

    if (!fs.existsSync(filePath)) {
      validation.missing.push(file);
    } else {
      const content = fs.readFileSync(filePath, "utf8");

      // Check file size
      if (content.length < 100) {
        validation.warnings.push(
          `${file} seems too short (${content.length} chars)`,
        );
      }

      // Check for incomplete markdown
      const headingCount = (content.match(/^#+ /gm) || []).length;
      if (headingCount === 0) {
        validation.corrupted.push(`${file} has no markdown headings`);
      }
    }
  }

  return {
    valid: validation.missing.length === 0 && validation.corrupted.length === 0,
    ...validation,
  };
};
```

### Sensitive Data Leak Prevention

**Problem:** API-Keys oder Credentials in exportierten Dateien
**Security-Scanner:**

```bash
#!/bin/bash
# export-security-check.sh

EXPORT_DIR="./FTMO-Agent-Dossier"

echo "🔍 Scanning for sensitive data..."

# Patterns für verschiedene Secrets
PATTERNS=(
  "sk-[A-Za-z0-9]{48}"           # OpenAI
  "\d{9,10}:[A-Za-z0-9_-]{35}"   # Telegram
  "AKIA[0-9A-Z]{16}"             # AWS
  "ghp_[A-Za-z0-9]{36}"          # GitHub Token
  "password['\"]?\s*[:=]\s*['\"]?[^'\"\s]+" # Passwords
)

FOUND_LEAKS=0

for pattern in "${PATTERNS[@]}"; do
  if grep -rP "$pattern" "$EXPORT_DIR" 2>/dev/null; then
    echo "❌ Potential secret found matching: $pattern"
    FOUND_LEAKS=$((FOUND_LEAKS + 1))
  fi
done

if [ $FOUND_LEAKS -eq 0 ]; then
  echo "✅ No sensitive data detected"
  exit 0
else
  echo "❌ Found $FOUND_LEAKS potential security issues"
  echo "Please review and remove sensitive data before sharing!"
  exit 1
fi
```

### PDF-Generation Debugging

**Problem:** PDF-Export schlägt fehl oder hat falsche Formatierung
**PDF-Generator mit Error-Handling:**

```javascript
// Robuster PDF-Export mit Pandoc
const generatePDF = async (markdownFiles, outputPath) => {
  const { exec } = require("child_process");
  const util = require("util");
  const execPromise = util.promisify(exec);

  try {
    // Check pandoc installation
    await execPromise("pandoc --version");
  } catch (error) {
    return {
      success: false,
      error: "Pandoc not installed. Install with: apt-get install pandoc",
    };
  }

  const filesString = markdownFiles.join(" ");
  const command = `pandoc ${filesString} -o ${outputPath} --standalone --toc --pdf-engine=xelatex`;

  try {
    const { stdout, stderr } = await execPromise(command);

    if (stderr && !stderr.includes("warning")) {
      console.warn("Pandoc warnings:", stderr);
    }

    // Verify output exists
    const fs = require("fs");
    if (fs.existsSync(outputPath)) {
      const stats = fs.statSync(outputPath);
      return {
        success: true,
        fileSize: stats.size,
        path: outputPath,
      };
    } else {
      return {
        success: false,
        error: "PDF was not created",
      };
    }
  } catch (error) {
    return {
      success: false,
      error: error.message,
      command: command,
    };
  }
};
```

### Workflow-Export Integrity Check

**Problem:** Exportierte n8n-Workflows sind inkorrekt
**Workflow-Validator:**

```javascript
// n8n Workflow Export Validator
const validateWorkflowExport = (workflowJson) => {
  const validation = {
    valid: true,
    errors: [],
    warnings: [],
    stats: {},
  };

  try {
    const workflow =
      typeof workflowJson === "string"
        ? JSON.parse(workflowJson)
        : workflowJson;

    // Check required fields
    if (!workflow.name) {
      validation.errors.push("Missing workflow name");
    }

    if (!workflow.nodes || !Array.isArray(workflow.nodes)) {
      validation.errors.push("Missing or invalid nodes array");
    } else {
      validation.stats.nodeCount = workflow.nodes.length;

      // Check each node
      for (const node of workflow.nodes) {
        if (!node.type) {
          validation.errors.push(`Node ${node.name || "unknown"} missing type`);
        }

        // Check for credentials
        if (node.credentials) {
          validation.warnings.push(
            `Node ${node.name} contains credential references - ensure they're properly handled`,
          );
        }
      }
    }

    if (!workflow.connections) {
      validation.warnings.push(
        "No connections defined - workflow may be incomplete",
      );
    } else {
      validation.stats.connectionCount = Object.keys(
        workflow.connections,
      ).length;
    }

    validation.valid = validation.errors.length === 0;
  } catch (error) {
    validation.valid = false;
    validation.errors.push(`JSON parsing failed: ${error.message}`);
  }

  return validation;
};
```

### Documentation Completeness Checker

**Problem:** Systemdossier ist unvollständig
**Completeness-Audit:**

```javascript
// Systemdossier Vollständigkeits-Prüfung
const auditDossierCompleteness = (dossierPath) => {
  const fs = require("fs");
  const path = require("path");

  const checklist = {
    sections: {
      executive_summary: {
        required: ["Zweck", "Architektur", "Ergebnisse"],
        found: [],
      },
      architecture: {
        required: ["Komponenten", "Datenfluss", "Abhängigkeiten"],
        found: [],
      },
      security: {
        required: ["Umgebungsvariablen", "Zugriffsrechte", "Backup-Strategie"],
        found: [],
      },
      metrics: {
        required: ["Winrate", "Latenz", "Fehlerrate"],
        found: [],
      },
    },
    completeness: 0,
    missing: [],
  };

  // Check Executive Summary
  const summaryPath = path.join(dossierPath, "01_Executive_Summary.md");
  if (fs.existsSync(summaryPath)) {
    const content = fs.readFileSync(summaryPath, "utf8");
    for (const term of checklist.sections.executive_summary.required) {
      if (content.toLowerCase().includes(term.toLowerCase())) {
        checklist.sections.executive_summary.found.push(term);
      }
    }
  }

  // Check Architecture
  const archPath = path.join(dossierPath, "02_System_Architecture.md");
  if (fs.existsSync(archPath)) {
    const content = fs.readFileSync(archPath, "utf8");
    for (const term of checklist.sections.architecture.required) {
      if (content.toLowerCase().includes(term.toLowerCase())) {
        checklist.sections.architecture.found.push(term);
      }
    }
  }

  // Calculate completeness
  let totalRequired = 0;
  let totalFound = 0;

  for (const section of Object.values(checklist.sections)) {
    totalRequired += section.required.length;
    totalFound += section.found.length;

    const missing = section.required.filter((r) => !section.found.includes(r));
    checklist.missing.push(...missing);
  }

  checklist.completeness = Math.round((totalFound / totalRequired) * 100);

  return checklist;
};
```

---

## 🧩 Abschnitt 11 – Hausaufgabe / Experiment

**Basis-Aufgabe (15 Min):**

1. Erstelle die Ordnerstruktur für dein Systemdossier
2. Exportiere alle n8n-Workflows in den Workflows-Ordner
3. Führe den Security-Scanner aus

**Erweiterte Herausforderung (30 Min):**

1. Schreibe die Executive Summary mit deinen realen Metriken
2. Erstelle ein System-Architektur-Diagramm (ASCII oder draw.io)
3. Generiere ein PDF-Dokument mit Pandoc

**Bonus-Experiment (45 Min):**

1. Automatisiere den kompletten Export-Prozess mit einem n8n-Workflow
2. Implementiere automatische Validierung aller exportierten Dateien
3. Erstelle ein GitHub Actions Workflow für automatische Dossier-Erstellung

---

## ✅ Zusammenfassung

Nach Kapitel 13 kannst du:

- dein Projekt vollständig exportieren und präsentieren,
- alle technischen und konzeptionellen Teile nachvollziehbar dokumentieren,
- dein System als professionelles Dossier aufbereiten,
- und den Schritt vom Lernprojekt zum vorzeigbaren Produkt vollziehen.

Damit ist der Kurs „KI-Automatisierung für FTMO“ abgeschlossen.
Im **Anhang der Kursmappe** folgt noch die Revision: Dort wird die ursprüngliche Korrektur aus Kapitel 2 eingetragen – das Modell wird als _„aktuelles LLM Modell“_ bezeichnet, um die Zukunftssicherheit der Dokumentation zu gewährleisten.
