# 📘 Kapitel 9 – Qualitätsmetriken und kontinuierliche Verbesserung

**Lernziel:**
Nach dieser Lektion kannst du aus Logs und Feedbacks automatisch **Leistungsmetriken** generieren.
Du lernst, wie du Trefferquote, Fehlerrate, Latenz und Tokenverbrauch berechnest und wie du daraus ableitest, wo dein System optimiert werden sollte.

---

## 🧩 Abschnitt 1 – Warum Qualitätsmetriken entscheidend sind

Ein Agent ohne Messgrößen ist wie ein Trader ohne Statistik.
Ohne Zahlen bleibt Leistung Gefühlssache – und das führt zu Fehleinschätzungen.

Qualitätsmetriken erlauben dir, systematisch zu prüfen:

- Arbeitet der Agent konsistent?
- Verbessern sich Entscheidungen über Zeit?
- Lohnt sich eine Prompt- oder Parameteränderung?
- Wie stabil ist der Workflow bei hoher Last?

Ziel: Objektive, wiederholbare Beurteilung statt subjektiver Eindrücke.

---

## ⚙️ Abschnitt 2 – Welche Metriken wichtig sind

**Primäre Metriken (Leistung):**

- `winrate` – Trefferquote (TP vs. SL)
- `expectancy` – Durchschnittsgewinn pro Trade
- `latency_ms` – durchschnittliche Laufzeit pro Run
- `error_rate` – Anteil fehlerhafter Ausführungen
- `consistency` – Varianz der Ergebnisse zwischen Tagen

**Sekundäre Metriken (Kosten & Effizienz):**

- `token_usage` – Verbrauchte Tokens pro Analyse
- `cost_per_decision` – geschätzte API-Kosten
- `retry_count` – wie oft ein Workflow wiederholt wurde

Alle Metriken werden aus bestehenden Logs (Kapitel 5–8) berechnet.

---

## 🧠 Abschnitt 3 – Berechnung von Metriken in n8n

Du kannst n8n direkt zur Auswertung verwenden.
Ein **Data Store Query + Function Node** reicht oft schon aus.

**Function-Node Beispiel:**

```javascript
const items = $input.all();
const metrics = {
  total: items.length,
  wins: 0,
  losses: 0,
  retries: 0,
  totalTokens: 0,
  totalDuration: 0,
};

for (const i of items) {
  const j = i.json;
  if (j.outcome === "TP") metrics.wins++;
  if (j.outcome === "SL") metrics.losses++;
  metrics.retries += j.retry_count || 0;
  metrics.totalTokens += j.token_usage || 0;
  metrics.totalDuration += j.latency_ms || 0;
}

metrics.winrate = metrics.total ? metrics.wins / metrics.total : 0;
metrics.avgDuration = metrics.total ? metrics.totalDuration / metrics.total : 0;
metrics.avgTokens = metrics.total ? metrics.totalTokens / metrics.total : 0;

return [metrics];
```

**Debugging-Tipp:**
Teste jede Metrik einzeln. Lass dir mit `console.log(metrics)` Zwischenschritte anzeigen, um Rundungsfehler und leere Felder früh zu erkennen.

---

## 🧩 Abschnitt 4 – Automatische Qualitätsreports

Mit n8n kannst du regelmäßige Qualitätsauswertungen automatisch generieren lassen.

**Beispiel-Flow:**

```
[Cron → Query Data Store]
   ↓
[Function → Metrics Calculation]
   ↓
[LLM Node → Natural-Language Summary]
   ↓
[Telegram / E-Mail Report]
```

**Beispieltext im LLM-Node:**

```
Analysiere folgende Kennzahlen:
Winrate: 0.63
Average Latency: 2.4s
Error Rate: 0.07
Gib eine kurze qualitative Bewertung und Handlungsempfehlung.
```

So bekommst du täglich einen „Agent Performance Report“ – kompakt, verständlich und aus deinen echten Daten berechnet.

---

## 💡 Abschnitt 5 – Debugging von Metriken

Fehler in Statistiken sind tückisch, weil sie selten als „Fehler“ auffallen.
Beachte daher:

### 🔍 Tipp 1: „Check for nulls“

In Function-Nodes immer prüfen:

```javascript
if (j.winrate == null) throw new Error("Winrate not calculated");
```

### 🔍 Tipp 2: „Cross-Validate“

Vergleiche deine Winrate mit dem tatsächlichen Verhältnis in der Datenbank:

```sql
SELECT COUNT(*) FILTER (WHERE outcome='TP')::float / COUNT(*) FROM ftmo_signals;
```

(n8n kann SQL-Queries an externe Datenbanken ausführen.)

### 🔍 Tipp 3: „Version Tagging“

Speichere in jeder Metrik den Prompt- oder Strategiestand:

```json
{ "winrate": 0.63, "strategy_version": "v1.3" }
```

Damit kannst du spätere Änderungen korrekt zuordnen.

---

## ⚙️ Abschnitt 6 – Visualisierung der Agentenleistung

Nutze Tools wie **Grafana**, **Metabase** oder **n8n-Dashboards**, um Metriken visuell darzustellen.
Alternativ kannst du per Function-Node HTML-Reports erzeugen.

**Mini-Beispiel:**

```javascript
const data = $json;
return [
  {
    html: `
  <h3>Agenten-Performance</h3>
  <ul>
    <li>Winrate: ${(data.winrate * 100).toFixed(2)}%</li>
    <li>Durchschnittliche Dauer: ${data.avgDuration.toFixed(0)} ms</li>
    <li>Tokenverbrauch: ${data.avgTokens.toFixed(1)}</li>
  </ul>
  `,
  },
];
```

Anschließend → **E-Mail Node** → „HTML format“ aktivieren.

---

## 🧩 Abschnitt 7 – Praxis: Automatischer Qualitätsreport

1. Erstelle in n8n einen Cron-getriggerten Workflow (z. B. täglich um 18:00 Uhr).
2. Lies den Data Store `ftmo_signals` aus.
3. Berechne Metriken mit Function-Node.
4. Sende den Report als:
   - Telegram-Nachricht (Kurzfassung)
   - E-Mail mit HTML-Report
5. Logge jede Report-Ausführung mit `timestamp` und `metrics_id`.

**Debugging-Hinweis:**
Teste Report-Flows mit kleinem Query-Limit (`limit: 5`), um Performance-Probleme und Syntaxfehler früh zu erkennen.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Metrik hat für dich den höchsten praktischen Wert?
- Wann gilt eine Winrate als „verlässlich" – 20 Trades oder 200?
- Wie könntest du das Feedback aus dem Report nutzen, um Prompts oder Parameter anzupassen?

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Erstelle deinen automatischen Qualitätsreport.
2. Miss mindestens drei Tage lang Metriken.
3. Prüfe, ob sich Winrate oder Laufzeit verändern.
4. Notiere in deiner Kursmappe:
   - mögliche Ursachen
   - geplante Anpassungen
   - beobachtete Verbesserungen
5. Optional: Lasse das LLM-Modell deine Reports selbst interpretieren (Meta-Analyse).

---

## 🚨 Abschnitt 10 – Qualitätsmetriken Debugging

### Häufige Metriken-Berechnungsfehler

**Problem:** Falsche Winrate-Berechnungen
**Debug-Strategie:**

```javascript
// Robuste Winrate-Berechnung mit Validation
const calculateWinrateRobust = (trades) => {
  if (!Array.isArray(trades) || trades.length === 0) {
    console.warn("Invalid trades data for winrate calculation");
    return { winrate: 0, error: "No valid trades data" };
  }

  const validTrades = trades.filter(
    (trade) => trade.outcome === "TP" || trade.outcome === "SL",
  );

  if (validTrades.length === 0) {
    console.warn("No completed trades found");
    return { winrate: 0, error: "No completed trades" };
  }

  const wins = validTrades.filter((trade) => trade.outcome === "TP").length;
  const winrate = wins / validTrades.length;

  console.log(
    `Winrate calculation: ${wins}/${validTrades.length} = ${winrate.toFixed(4)}`,
  );

  return {
    winrate: Math.round(winrate * 10000) / 10000, // 4 Dezimalstellen
    wins,
    total: validTrades.length,
    invalid_trades: trades.length - validTrades.length,
  };
};
```

### Statistische Signifikanz prüfen

**Problem:** Zu kleine Stichproben führen zu falschen Schlüssen
**Validation-Framework:**

```javascript
// Statistische Signifikanz-Prüfung
const validateStatisticalSignificance = (metrics, confidenceLevel = 0.95) => {
  const minSampleSize = 30; // Mindestanzahl für Normalverteilung
  const criticalValue = confidenceLevel === 0.95 ? 1.96 : 2.58; // 95% oder 99%

  const results = {
    valid: true,
    warnings: [],
    confidence_intervals: {},
  };

  // Prüfe Sample Size
  if (metrics.total < minSampleSize) {
    results.valid = false;
    results.warnings.push(
      `Sample size too small: ${metrics.total} < ${minSampleSize}`,
    );
  }

  // Berechne Konfidenzintervall für Winrate
  if (metrics.winrate !== undefined) {
    const p = metrics.winrate;
    const n = metrics.total;
    const standardError = Math.sqrt((p * (1 - p)) / n);
    const marginOfError = criticalValue * standardError;

    results.confidence_intervals.winrate = {
      lower: Math.max(0, p - marginOfError),
      upper: Math.min(1, p + marginOfError),
      margin_of_error: marginOfError,
    };

    if (marginOfError > 0.1) {
      // 10% Margin ist zu hoch
      results.warnings.push(
        `Winrate confidence interval too wide: ±${(marginOfError * 100).toFixed(1)}%`,
      );
    }
  }

  return results;
};
```

### Performance-Metriken Anomalie-Detection

**Problem:** Schleichende Performance-Degradation übersehen
**Monitoring-System:**

```javascript
// Anomalie-Detection für Performance-Metriken
const detectPerformanceAnomalies = (currentMetrics, historicalData) => {
  const anomalies = [];
  const thresholds = {
    latency_increase: 2.0, // 200% Erhöhung
    winrate_drop: 0.15, // 15 Prozentpunkte
    error_rate_spike: 0.1, // 10% Fehlerrate
  };

  // Berechne historische Baselines
  const baseline = calculateBaseline(historicalData);

  // Prüfe Latenz-Anomalien
  if (
    currentMetrics.avgDuration >
    baseline.avgDuration * thresholds.latency_increase
  ) {
    anomalies.push({
      type: "latency_spike",
      current: currentMetrics.avgDuration,
      baseline: baseline.avgDuration,
      increase_factor: currentMetrics.avgDuration / baseline.avgDuration,
      severity: "high",
    });
  }

  // Prüfe Winrate-Anomalien
  if (currentMetrics.winrate < baseline.winrate - thresholds.winrate_drop) {
    anomalies.push({
      type: "winrate_drop",
      current: currentMetrics.winrate,
      baseline: baseline.winrate,
      drop: baseline.winrate - currentMetrics.winrate,
      severity: "critical",
    });
  }

  // Prüfe Error-Rate-Anomalien
  if (currentMetrics.error_rate > thresholds.error_rate_spike) {
    anomalies.push({
      type: "error_spike",
      current: currentMetrics.error_rate,
      threshold: thresholds.error_rate_spike,
      severity: "high",
    });
  }

  return {
    anomalies,
    baseline,
    alert_required: anomalies.some((a) => a.severity === "critical"),
  };
};

const calculateBaseline = (historicalData) => {
  const recentData = historicalData.slice(-30); // Letzte 30 Einträge

  return {
    avgDuration:
      recentData.reduce((sum, d) => sum + d.avgDuration, 0) / recentData.length,
    winrate:
      recentData.reduce((sum, d) => sum + d.winrate, 0) / recentData.length,
    error_rate:
      recentData.reduce((sum, d) => sum + d.error_rate, 0) / recentData.length,
  };
};
```

### Metriken-Dashboard Generator

**Problem:** Metriken sind schwer zu visualisieren
**HTML-Report-Generator:**

```javascript
// Automatischer HTML-Dashboard-Generator
const generateMetricsDashboard = (metrics, anomalies) => {
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
        <title>FTMO Agent Metrics Dashboard</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .metric { background: #f5f5f5; padding: 15px; margin: 10px 0; border-radius: 5px; }
            .good { border-left: 5px solid #4CAF50; }
            .warning { border-left: 5px solid #FF9800; }
            .critical { border-left: 5px solid #F44336; }
            .number { font-size: 24px; font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>🤖 FTMO Agent Performance Dashboard</h1>
        <p>Generated: ${new Date().toISOString()}</p>

        <div class="metric ${getMetricClass(metrics.winrate, 0.6)}">
            <h3>📈 Win Rate</h3>
            <div class="number">${(metrics.winrate * 100).toFixed(2)}%</div>
            <p>${metrics.wins}/${metrics.total} trades</p>
        </div>

        <div class="metric ${getLatencyClass(metrics.avgDuration)}">
            <h3>⚡ Average Latency</h3>
            <div class="number">${metrics.avgDuration.toFixed(0)}ms</div>
        </div>

        <div class="metric ${getErrorRateClass(metrics.error_rate || 0)}">
            <h3>🚨 Error Rate</h3>
            <div class="number">${((metrics.error_rate || 0) * 100).toFixed(2)}%</div>
        </div>

        ${
          anomalies && anomalies.length > 0
            ? `
        <h2>⚠️ Detected Anomalies</h2>
        ${anomalies
          .map(
            (anomaly) => `
        <div class="metric critical">
            <h3>${anomaly.type.toUpperCase()}</h3>
            <p>Severity: ${anomaly.severity}</p>
            <p>Current: ${JSON.stringify(anomaly.current)}</p>
        </div>
        `,
          )
          .join("")}
        `
            : "<p>✅ No anomalies detected</p>"
        }

    </body>
    </html>
  `;

  return html;
};

const getMetricClass = (value, threshold) => {
  if (value >= threshold) return "good";
  if (value >= threshold * 0.8) return "warning";
  return "critical";
};

const getLatencyClass = (latency) => {
  if (latency < 1000) return "good";
  if (latency < 3000) return "warning";
  return "critical";
};

const getErrorRateClass = (errorRate) => {
  if (errorRate < 0.05) return "good";
  if (errorRate < 0.1) return "warning";
  return "critical";
};
```

---

## ✅ Zusammenfassung

Nach Kapitel 9 kannst du:

- Metriken automatisiert berechnen und loggen,
- Agentenleistung objektiv bewerten,
- Fehler in Statistiken erkennen und beheben,
- Anomalien automatisch detektieren,
- und Reports für kontinuierliche Verbesserung nutzen.

Im nächsten Kapitel (10) lernst du, wie du deinen gesamten Agentenprozess dokumentierst und versionierst – damit er als professionelles System nachvollziehbar, wartbar und auditierbar bleibt.
