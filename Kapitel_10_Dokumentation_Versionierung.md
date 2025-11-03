# 📘 Kapitel 10 – Dokumentation, Versionierung und Auditierbarkeit

**Lernziel:**  
Nach dieser Lektion kannst du deinen gesamten Agentenprozess so dokumentieren, dass er jederzeit reproduzierbar, überprüfbar und auditierbar bleibt.  
Du lernst, wie du Versionsstände sicherst, Änderungen protokollierst und damit die Grundlage für professionelle Systembewertung schaffst.

---

## 🧩 Abschnitt 1 – Warum Dokumentation unverzichtbar ist

Sobald du beginnst, mit realen Strategien oder Testkonten zu arbeiten, reicht „Ich weiß noch, was ich geändert habe“ nicht mehr aus.  

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

Ein `CHANGELOG.md` ist kein „nice-to-have“ – es ist dein Agententagebuch.  

**Formatbeispiel:**
```
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
  message: "Flow executed successfully"
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
4. Sendet Telegram-Nachricht: *„Neuer Agenten-Release gespeichert.“*

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

## ✅ Zusammenfassung

Nach Kapitel 10 kannst du:
- deinen Agenten vollständig versionieren und dokumentieren,  
- Änderungen transparent nachvollziehen,  
- Audit-Trails führen und Debugging beschleunigen,  
- und so den Grundstein für langfristige Systemqualität legen.  

Im nächsten Kapitel (11) geht es um **Sicherheit, Zugriffsrechte und Deployment** – wie du deinen Agenten produktionsreif machst, ohne Risiken für Daten oder Infrastruktur.