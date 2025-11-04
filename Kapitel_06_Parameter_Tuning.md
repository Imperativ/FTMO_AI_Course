# 📘 Kapitel 6 – Parameter-Tuning und A/B-Vergleiche

**Lernziel:**
Nach dieser Lektion kannst du deinen Agenten so erweitern, dass er **verschiedene Strategiekonfigurationen testet**, Ergebnisse protokolliert und bewertet.
Du lernst, wie man mit A/B-Vergleichen experimentiert, Parameter automatisch variiert und auf Basis der gespeicherten Daten Performance misst.

---

## 🧩 Abschnitt 1 – Warum Parameter-Tuning wichtig ist

Selbst gute Strategien verlieren, wenn sie starr bleiben.
Die Märkte verändern sich, Liquidität, Volatilität und Trader-Verhalten ebenfalls.
Dein Ziel ist es daher nicht, „die perfekte Einstellung“ zu finden, sondern ein **System**, das Anpassung ermöglicht.

Das Prinzip lautet:

> _Optimierung durch strukturierte Variation – nicht durch Intuition._

Mit anderen Worten: du testest Varianten gezielt, beobachtest Ergebnisse, und wählst jene Parameter, die langfristig Stabilität bringen.

---

## ⚙️ Abschnitt 2 – Parameter als variable Inputs

In n8n kannst du Parameter dynamisch übergeben – ideal für systematisches Testen.

Beispiel:
Du möchtest herausfinden, ob ein höherer Stop-Loss-Abstand die Winrate verbessert.

**Function-Node (Parameter-Generator):**

```javascript
return [
  { variant: "A", stopLossPoints: 30, riskPercent: 0.02 },
  { variant: "B", stopLossPoints: 50, riskPercent: 0.02 },
];
```

Verbinde diesen Node mit deinem bestehenden **Analyse-Flow** über eine „Split In Batches“-Node, damit jede Variante separat getestet wird.

---

## 🧠 Abschnitt 3 – A/B-Vergleichsstruktur

Ein typischer Testablauf sieht so aus:

```
[Trigger / Backtest-Start]
   ↓
[Function Node → Parameter-Varianten]
   ↓
[Loop über Varianten]
   ↓
[Analyse & Entscheidung (LLM)]
   ↓
[Simulation oder Logging]
   ↓
[Data Store Insert (mit Variant-ID)]
```

In jeder Schleife wird eine Variante getestet.
Speichere zusätzlich zur normalen Signal-Info ein Feld `variant: "A"` oder `"B"` im Data Store, um die Ergebnisse später trennen zu können.

---

## 🧩 Abschnitt 4 – Automatisierte Auswertung in n8n

Sobald du mehrere Varianten protokolliert hast, kannst du direkt in n8n die Performance vergleichen.

**Data Store Query + Function-Node:**

```javascript
const items = $input.all();
const grouped = {};
for (const i of items) {
  const v = i.json.variant || "unknown";
  grouped[v] = grouped[v] || { total: 0, wins: 0, losses: 0 };
  grouped[v].total++;
  if (i.json.outcome === "TP") grouped[v].wins++;
  if (i.json.outcome === "SL") grouped[v].losses++;
}
const report = Object.keys(grouped).map((v) => ({
  variant: v,
  winrate: grouped[v].wins / grouped[v].total,
  total: grouped[v].total,
}));
return report;
```

Damit erhältst du sofort die Erfolgsquote pro Variante.
Das Ergebnis kannst du per Telegram, E-Mail oder Dashboard visualisieren lassen.

---

## 💡 Abschnitt 5 – Parameterwahl durch LLM-Auswertung

Das aktuelle LLM-Modell kann selbst einfache Mustererkennung übernehmen.
Wenn du willst, dass dein Agent Verbesserungsvorschläge macht, gib ihm die aggregierten Ergebnisse als Input:

```
Analysiere folgende Testergebnisse:
Variante A: Winrate 0.52, Drawdown 3.1 %
Variante B: Winrate 0.64, Drawdown 5.5 %
Welche Variante bietet langfristig das bessere Chance/Risiko-Verhältnis?
```

Der Agent wird nicht „optimieren“, aber qualitativ einschätzen, welche Kombination robuster erscheint.
Solche Rückmeldungen helfen dir, menschliche und maschinelle Perspektiven zu kombinieren.

---

## ⚠️ Abschnitt 6 – Achte auf statistische Fallstricke

- **Zu kleine Stichprobe:** Unter 30 Trades pro Variante ist fast jede Erkenntnis Zufall.
- **Survivorship Bias:** Wenn du nur erfolgreiche Phasen analysierst, täuscht Stabilität.
- **Overfitting:** Je enger du an vergangene Daten anpasst, desto schlechter performt die Strategie live.
- **Paralleltests:** Führe nie zu viele Varianten gleichzeitig, sonst verwischst du die Ergebnisse.

Erfahrungsgemäß ist _langsames, kontrolliertes Testen_ der Schlüssel.

---

## 🧩 Abschnitt 7 – Praxis: Dein erster A/B-Workflow

1. Kopiere deinen Flow aus Kapitel 5.
2. Füge vor dem LLM-Node einen **Function-Node** hinzu, der zwei Parameter-Sets erzeugt (z. B. Stop-Loss 30 vs. 50).
3. Füge einen **Split In Batches-Node** ein, damit jede Variante separat läuft.
4. Ergänze im **Data Store Insert** ein Feld `variant`.
5. Nach einigen Durchläufen: Query-Node + Function-Node → Aggregation der Winrates.
6. Sende die Ergebnisse per Telegram.

Optional: Baue einen dritten Flow, der automatisch den besten Parameter anzeigt, sobald > 30 Trades pro Variante erreicht sind.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Parameter sind bei dir am einflussreichsten?
- Ab wann würdest du sagen, dass eine Variante „statistisch signifikant besser“ ist?
- Wie könntest du den Test planbar automatisieren, ohne die Kontrolle zu verlieren?

Solche Überlegungen machen den Unterschied zwischen Experiment und Optimierung.

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Führe einen A/B-Test mit zwei Varianten deines bisherigen Systems durch (z. B. Stop-Loss 30 vs. 50).
2. Dokumentiere mindestens 10 Signale je Variante.
3. Erstelle eine Winrate-Statistik in n8n.
4. Frage das LLM-Modell nach einer qualitativen Einschätzung.
5. Trage Beobachtungen in deine Kursmappe ein.

---

## 🚨 Abschnitt 10 – Parameter-Tuning Debugging

### Häufige Probleme bei A/B-Tests

**Problem:** Parameter-Varianten werden nicht korrekt getrennt
**Lösung:**

```javascript
// Sichere Variant-ID Generation
const variants = [
  { id: "A", stopLoss: 30, risk: 0.02, timestamp: Date.now() },
  { id: "B", stopLoss: 50, risk: 0.02, timestamp: Date.now() },
];

// Validation vor Speicherung
if (!variant.id || !variant.stopLoss) {
  throw new Error("Invalid variant configuration");
}
return variants;
```

### Split In Batches Node Debugging

**Problem:** Batches laufen nicht sequenziell
**Debug-Strategie:**

```javascript
// Debug Split-Verhalten
console.log("=== BATCH DEBUG ===");
console.log("Current Item:", JSON.stringify($item, null, 2));
console.log("Batch Info:", {
  currentIndex: $itemIndex,
  totalItems: $totalItems,
  batchSize: $batchSize,
});
console.log("=== END DEBUG ===");
return [$item];
```

### A/B-Test Validation Template

**Problem:** Ergebnisse statistisch nicht signifikant
**Validation-Code:**

```javascript
const validateABTest = (results) => {
  const minSampleSize = 30;
  const variants = Object.keys(results);

  for (const variant of variants) {
    const data = results[variant];
    if (data.total < minSampleSize) {
      return {
        valid: false,
        error: `Variant ${variant}: Only ${data.total} samples (min: ${minSampleSize})`,
      };
    }
  }

  // Berechne Konfidenzintervall
  const confidenceLevel = 0.95;
  const criticalValue = 1.96; // für 95% Konfidenz

  for (const variant of variants) {
    const data = results[variant];
    const winrate = data.wins / data.total;
    const standardError = Math.sqrt((winrate * (1 - winrate)) / data.total);
    const marginOfError = criticalValue * standardError;

    data.confidenceInterval = {
      lower: Math.max(0, winrate - marginOfError),
      upper: Math.min(1, winrate + marginOfError),
    };
  }

  return { valid: true, results };
};
```

### Performance-Optimierung für Parameter-Tests

**Problem:** Zu viele parallele LLM-Calls verursachen Rate-Limits
**Lösung:**

```javascript
// Rate-Limiting für Parameter-Tests
const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const processVariantsWithDelay = async (variants) => {
  const results = [];

  for (let i = 0; i < variants.length; i++) {
    try {
      // Verarbeite Variante
      const result = await processVariant(variants[i]);
      results.push(result);

      // Pause zwischen Varianten (Rate-Limit Schutz)
      if (i < variants.length - 1) {
        await delay(1000); // 1 Sekunde Pause
      }
    } catch (error) {
      console.error(`Variant ${variants[i].id} failed:`, error.message);
      results.push({
        id: variants[i].id,
        error: error.message,
        timestamp: new Date().toISOString(),
      });
    }
  }

  return results;
};
```

---

## 💡 Abschnitt 11 – Erweiterte Parameter-Optimierung

### Dynamische Parameter-Anpassung

Statt nur A/B-Tests kannst du Parameter auch dynamisch anpassen basierend auf Marktbedingungen:

```javascript
// Markt-adaptive Parameter-Logik
const adaptParametersToMarket = (marketData, currentParams) => {
  const volatility = calculateVolatility(marketData);
  const trend = marketData.trend || "neutral";

  let adaptedParams = { ...currentParams };

  // Anpassung basierend auf Volatilität
  if (volatility > 0.02) {
    adaptedParams.stopLoss = Math.min(currentParams.stopLoss * 1.5, 100);
    adaptedParams.riskPercent = Math.max(currentParams.riskPercent * 0.7, 0.01);
  }

  // Anpassung basierend auf Trend
  if (trend === "strong_bullish") {
    adaptedParams.confidenceThreshold = 0.6; // Niedrigere Schwelle bei klaren Trends
  } else if (trend === "sideways") {
    adaptedParams.confidenceThreshold = 0.8; // Höhere Schwelle bei unklaren Märkten
  }

  return {
    original: currentParams,
    adapted: adaptedParams,
    reason: `Volatility: ${volatility.toFixed(4)}, Trend: ${trend}`,
    timestamp: new Date().toISOString(),
  };
};
```

### Multi-Dimensional Parameter-Testing

**Erweiterte A/B/C/D-Tests:**

```javascript
// Kombinatorische Parameter-Tests
const generateParameterCombinations = () => {
  const stopLossValues = [30, 50, 70];
  const riskValues = [0.01, 0.02, 0.03];
  const confidenceThresholds = [0.6, 0.7, 0.8];

  const combinations = [];
  let id = 1;

  for (const sl of stopLossValues) {
    for (const risk of riskValues) {
      for (const conf of confidenceThresholds) {
        combinations.push({
          id: `V${id}`,
          stopLoss: sl,
          riskPercent: risk,
          confidenceThreshold: conf,
          expectedTrades: Math.ceil(100 / (risk * 100)), // Grobe Schätzung
          priority: calculatePriority(sl, risk, conf),
        });
        id++;
      }
    }
  }

  // Sortiere nach Priorität (vielversprechendste zuerst)
  return combinations.sort((a, b) => b.priority - a.priority);
};

const calculatePriority = (stopLoss, risk, confidence) => {
  // Heuristische Prioritätsberechnung
  // Niedrigeres Risiko = höhere Priorität
  // Moderater Stop-Loss = höhere Priorität
  // Höhere Confidence = höhere Priorität
  const riskScore = (0.03 - risk) / 0.02; // 0-1, höher = besser
  const stopLossScore = 1 - Math.abs(50 - stopLoss) / 50; // Optimal bei 50
  const confidenceScore = confidence;

  return (riskScore + stopLossScore + confidenceScore) / 3;
};
```

### Parameter-History und Trend-Analyse

```javascript
// Parameter-Performance über Zeit tracken
const trackParameterPerformance = (paramHistory) => {
  const analysis = {
    trends: {},
    recommendations: [],
    stability: {},
  };

  // Analysiere Trends pro Parameter
  const parameters = ["stopLoss", "riskPercent", "confidenceThreshold"];

  for (const param of parameters) {
    const values = paramHistory.map((h) => ({
      value: h.params[param],
      winrate: h.winrate,
      date: h.date,
    }));

    // Berechne Korrelation zwischen Parameter und Performance
    const correlation = calculateCorrelation(
      values.map((v) => v.value),
      values.map((v) => v.winrate),
    );

    analysis.trends[param] = {
      correlation,
      bestValue: values.sort((a, b) => b.winrate - a.winrate)[0].value,
      stability: calculateVariance(values.map((v) => v.value)),
    };

    // Generiere Empfehlungen
    if (Math.abs(correlation) > 0.7) {
      analysis.recommendations.push({
        parameter: param,
        action: correlation > 0 ? "increase" : "decrease",
        confidence: Math.abs(correlation),
        reason: `Strong ${correlation > 0 ? "positive" : "negative"} correlation with performance`,
      });
    }
  }

  return analysis;
};

// Hilfsfunktion für Korrelationsberechnung
const calculateCorrelation = (x, y) => {
  const n = x.length;
  const sumX = x.reduce((a, b) => a + b, 0);
  const sumY = y.reduce((a, b) => a + b, 0);
  const sumXY = x.reduce((sum, xi, i) => sum + xi * y[i], 0);
  const sumX2 = x.reduce((sum, xi) => sum + xi * xi, 0);
  const sumY2 = y.reduce((sum, yi) => sum + yi * yi, 0);

  const numerator = n * sumXY - sumX * sumY;
  const denominator = Math.sqrt(
    (n * sumX2 - sumX * sumX) * (n * sumY2 - sumY * sumY),
  );

  return denominator === 0 ? 0 : numerator / denominator;
};
```

---

## ✅ Zusammenfassung

Nach Kapitel 6 kannst du:

- Parameter dynamisch variieren und testen,
- A/B-Vergleiche in n8n strukturieren,
- Ergebnisse automatisch auswerten,
- und dein LLM-System als beratende Instanz einbinden.

Im nächsten Kapitel lernst du, wie du **mehrere spezialisierte Agentenrollen** kombinierst – Analyst, Risiko-Manager und Strategieberater – um deinen FTMO-Assistenten modular zu machen.
