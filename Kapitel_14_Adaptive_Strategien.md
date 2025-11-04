# 📘 Kapitel 14 – Adaptive Strategien und kontinuierliches Lernen

**Lernziel:**
Nach dieser Lektion kannst du deinen Agenten so erweitern, dass er aus seinen eigenen Ergebnissen lernt, Strategien dynamisch anpasst und damit seine Performance im Zeitverlauf verbessert – unter voller Kontrolle und Nachvollziehbarkeit.

---

## 🧩 Abschnitt 1 – Was bedeutet „Adaptivität" im Agentensystem?

Ein adaptives System erkennt Muster in den eigenen Erfolgen und Fehlschlägen und nutzt diese Information zur Selbstoptimierung.
Das Ziel ist **kontrollierte Selbstanpassung**, nicht autonomes Umschreiben von Regeln.

In deinem FTMO-Agenten bedeutet das:

- Lernen aus historischen Trades und Evaluationslogs
- Dynamische Anpassung von Parametern wie Risiko, Zeithorizont, oder Entscheidungsschwellen
- Erprobung kleiner Veränderungen mit klarer Dokumentation („experimentelle Branches")
- Kontinuierliches Monitoring und Drift-Detection
- A/B-Testing verschiedener Strategievarianten

### Adaptive vs. Statische Systeme

**Statisches System:**

- Feste Regeln und Parameter
- Keine Anpassung an veränderte Marktbedingungen
- Manuelle Updates erforderlich

**Adaptives System:**

- Selbstlernende Parameter-Optimierung
- Automatische Erkennung von Regime-Changes
- Kontrollierte Evolution der Strategien
- Dokumentation aller Änderungen

---

## ⚙️ Abschnitt 2 – Feedback-Loop aus Logs und Metriken

Dein System produziert bereits reichlich Daten: Winrate, Drawdown, Latenz, Tokenverbrauch.
Diese Daten sind der Rohstoff für Lernen.

Ein typischer Feedback-Loop:

```
[Logs + Reports]
   ↓
[Evaluation Flow (aus Kap. 12)]
   ↓
[LLM Node → Musteranalyse]
   ↓
[Decision Node → Strategie-Update]
   ↓
[Commit & Versioning]
```

### Produktionsreifer Feedback-Loop in n8n

**Function-Node: Performance-Analyzer**

```javascript
// Feedback-Loop: Performance-Analyse und Parameter-Anpassung
const fs = require("fs");
const path = require("path");

class PerformanceFeedbackLoop {
  constructor() {
    this.dataPath = "./data/performance";
    this.configPath = "./config/adaptive_params.json";
    this.logPath = "./logs/feedback_loop.log";
    this.minSampleSize = 30; // Minimale Anzahl Trades für Anpassungen
  }

  // Lade historische Performance-Daten
  loadPerformanceData(days = 30) {
    try {
      const files = fs
        .readdirSync(this.dataPath)
        .filter((f) => f.endsWith(".json"))
        .sort()
        .slice(-days);

      const data = files.map((file) => {
        const content = fs.readFileSync(path.join(this.dataPath, file), "utf8");
        return JSON.parse(content);
      });

      return data;
    } catch (error) {
      this.log("ERROR", `Failed to load performance data: ${error.message}`);
      return [];
    }
  }

  // Berechne aktuelle Performance-Metriken
  calculateMetrics(data) {
    if (!data || data.length === 0) {
      return null;
    }

    const trades = data.flatMap((d) => d.trades || []);

    if (trades.length < this.minSampleSize) {
      this.log(
        "WARNING",
        `Insufficient sample size: ${trades.length}/${this.minSampleSize}`,
      );
      return null;
    }

    const wins = trades.filter((t) => t.profit > 0);
    const losses = trades.filter((t) => t.profit <= 0);

    const totalProfit = trades.reduce((sum, t) => sum + t.profit, 0);
    const avgWin =
      wins.length > 0
        ? wins.reduce((sum, t) => sum + t.profit, 0) / wins.length
        : 0;
    const avgLoss =
      losses.length > 0
        ? Math.abs(losses.reduce((sum, t) => sum + t.profit, 0) / losses.length)
        : 0;

    const winrate = wins.length / trades.length;
    const profitFactor = avgLoss > 0 ? avgWin / avgLoss : 0;

    // Berechne Drawdown
    let peak = 0;
    let maxDrawdown = 0;
    let runningTotal = 0;

    trades.forEach((trade) => {
      runningTotal += trade.profit;
      if (runningTotal > peak) peak = runningTotal;
      const drawdown = (peak - runningTotal) / peak;
      if (drawdown > maxDrawdown) maxDrawdown = drawdown;
    });

    // Berechne Volatilität
    const returns = trades.map((t) => t.profit);
    const avgReturn = returns.reduce((sum, r) => sum + r, 0) / returns.length;
    const variance =
      returns.reduce((sum, r) => sum + Math.pow(r - avgReturn, 2), 0) /
      returns.length;
    const volatility = Math.sqrt(variance);

    return {
      totalTrades: trades.length,
      winrate,
      profitFactor,
      totalProfit,
      avgWin,
      avgLoss,
      maxDrawdown,
      volatility,
      sharpeRatio: volatility > 0 ? avgReturn / volatility : 0,
    };
  }

  // Ermittle empfohlene Parameter-Anpassungen
  suggestAdjustments(currentMetrics, historicalMetrics) {
    const adjustments = [];
    const config = this.loadConfig();

    // Risk Multiplier Anpassung basierend auf Winrate und Profit Factor
    if (currentMetrics.winrate > 0.65 && currentMetrics.profitFactor > 1.5) {
      const newRisk = Math.min(config.riskMultiplier * 1.1, 2.0);
      adjustments.push({
        parameter: "riskMultiplier",
        currentValue: config.riskMultiplier,
        suggestedValue: newRisk,
        reason: "High winrate and profit factor - cautious risk increase",
        confidence: 0.85,
      });
    } else if (
      currentMetrics.winrate < 0.45 ||
      currentMetrics.profitFactor < 1.0
    ) {
      const newRisk = Math.max(config.riskMultiplier * 0.85, 0.5);
      adjustments.push({
        parameter: "riskMultiplier",
        currentValue: config.riskMultiplier,
        suggestedValue: newRisk,
        reason: "Low winrate or profit factor - risk reduction required",
        confidence: 0.95,
      });
    }

    // Drawdown Protection
    if (currentMetrics.maxDrawdown > 0.15) {
      adjustments.push({
        parameter: "maxDrawdownLimit",
        currentValue: config.maxDrawdownLimit,
        suggestedValue: Math.max(currentMetrics.maxDrawdown * 0.8, 0.1),
        reason: "High drawdown detected - tighten risk controls",
        confidence: 0.9,
      });
    }

    // Volatility Adjustment
    if (currentMetrics.volatility > historicalMetrics.avgVolatility * 1.5) {
      adjustments.push({
        parameter: "positionSizeMultiplier",
        currentValue: config.positionSizeMultiplier,
        suggestedValue: config.positionSizeMultiplier * 0.8,
        reason: "High volatility period - reduce position sizes",
        confidence: 0.88,
      });
    }

    return adjustments;
  }

  // Lade aktuelle Konfiguration
  loadConfig() {
    try {
      const content = fs.readFileSync(this.configPath, "utf8");
      return JSON.parse(content);
    } catch (error) {
      this.log("WARNING", `Config not found, using defaults: ${error.message}`);
      return {
        riskMultiplier: 1.0,
        maxDrawdownLimit: 0.15,
        positionSizeMultiplier: 1.0,
        adaptiveMode: true,
      };
    }
  }

  // Speichere aktualisierte Konfiguration
  saveConfig(config, adjustments) {
    try {
      config.lastUpdate = new Date().toISOString();
      config.adjustmentHistory = config.adjustmentHistory || [];
      config.adjustmentHistory.push({
        timestamp: new Date().toISOString(),
        adjustments,
      });

      // Behalte nur letzte 50 Einträge
      if (config.adjustmentHistory.length > 50) {
        config.adjustmentHistory = config.adjustmentHistory.slice(-50);
      }

      fs.writeFileSync(this.configPath, JSON.stringify(config, null, 2));
      this.log("INFO", "Configuration updated successfully");
    } catch (error) {
      this.log("ERROR", `Failed to save config: ${error.message}`);
      throw error;
    }
  }

  // Logging-Funktion
  log(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [${level}] ${message}\n`;

    try {
      fs.appendFileSync(this.logPath, logEntry);
    } catch (error) {
      console.error("Logging failed:", error.message);
    }
  }

  // Hauptausführung
  execute() {
    try {
      this.log("INFO", "Starting feedback loop execution");

      // Lade Daten
      const recentData = this.loadPerformanceData(30);
      const historicalData = this.loadPerformanceData(90);

      // Berechne Metriken
      const currentMetrics = this.calculateMetrics(recentData);
      const historicalMetrics = this.calculateMetrics(historicalData);

      if (!currentMetrics || !historicalMetrics) {
        return {
          status: "insufficient_data",
          message: "Not enough data for meaningful adjustments",
        };
      }

      // Ermittle Anpassungen
      const adjustments = this.suggestAdjustments(
        currentMetrics,
        historicalMetrics,
      );

      // Filtere nur hochkonfidente Anpassungen
      const approvedAdjustments = adjustments.filter(
        (adj) => adj.confidence >= 0.85,
      );

      if (approvedAdjustments.length > 0) {
        const config = this.loadConfig();

        // Wende Anpassungen an
        approvedAdjustments.forEach((adj) => {
          config[adj.parameter] = adj.suggestedValue;
        });

        this.saveConfig(config, approvedAdjustments);

        return {
          status: "adjustments_applied",
          metrics: currentMetrics,
          adjustments: approvedAdjustments,
        };
      }

      return {
        status: "no_adjustments_needed",
        metrics: currentMetrics,
        message: "Performance within acceptable parameters",
      };
    } catch (error) {
      this.log("ERROR", `Feedback loop failed: ${error.message}`);
      return {
        status: "error",
        error: error.message,
      };
    }
  }
}

// Ausführung
const feedbackLoop = new PerformanceFeedbackLoop();
const result = feedbackLoop.execute();

return [{ json: result }];
```

---

## 🧠 Abschnitt 3 – A/B-Testing Framework für Strategievarianten

Ein robustes A/B-Testing-System erlaubt dir, mehrere Strategievarianten parallel zu testen und die beste auszuwählen.

### Produktionsreifer A/B-Test Manager

**Function-Node: A/B-Test-Framework**

```javascript
// A/B-Testing Framework für Strategievarianten
const fs = require("fs");
const crypto = require("crypto");

class ABTestingFramework {
  constructor() {
    this.testsPath = "./data/ab_tests";
    this.resultsPath = "./data/ab_results";
    this.configPath = "./config/ab_tests.json";
  }

  // Erstelle neuen A/B-Test
  createTest(testConfig) {
    const testId = crypto.randomBytes(8).toString("hex");

    const test = {
      id: testId,
      name: testConfig.name,
      description: testConfig.description,
      variants: testConfig.variants,
      startDate: new Date().toISOString(),
      endDate: testConfig.duration
        ? new Date(
            Date.now() + testConfig.duration * 24 * 60 * 60 * 1000,
          ).toISOString()
        : null,
      status: "active",
      trafficSplit:
        testConfig.trafficSplit ||
        this.calculateEqualSplit(testConfig.variants.length),
      minSampleSize: testConfig.minSampleSize || 50,
      confidenceLevel: testConfig.confidenceLevel || 0.95,
      metrics: {
        primary: testConfig.primaryMetric || "profitFactor",
        secondary: testConfig.secondaryMetrics || ["winrate", "sharpeRatio"],
      },
    };

    this.saveTest(test);
    return test;
  }

  // Berechne gleichmäßige Traffic-Verteilung
  calculateEqualSplit(variantCount) {
    const split = {};
    const percentage = 1 / variantCount;

    for (let i = 0; i < variantCount; i++) {
      split[`variant_${String.fromCharCode(65 + i)}`] = percentage;
    }

    return split;
  }

  // Weise Trade einer Variante zu
  assignVariant(testId, tradeId) {
    const test = this.loadTest(testId);
    const random = Math.random();
    let cumulative = 0;

    for (const [variant, percentage] of Object.entries(test.trafficSplit)) {
      cumulative += percentage;
      if (random <= cumulative) {
        return variant;
      }
    }

    return Object.keys(test.trafficSplit)[0];
  }

  // Sammle Ergebnisse für Variante
  recordResult(testId, variant, tradeResult) {
    const resultFile = `${this.resultsPath}/${testId}_${variant}.json`;

    let results = [];
    if (fs.existsSync(resultFile)) {
      results = JSON.parse(fs.readFileSync(resultFile, "utf8"));
    }

    results.push({
      timestamp: new Date().toISOString(),
      ...tradeResult,
    });

    fs.writeFileSync(resultFile, JSON.stringify(results, null, 2));
  }

  // Analysiere Test-Ergebnisse
  analyzeTest(testId) {
    const test = this.loadTest(testId);
    const analysis = {
      testId,
      testName: test.name,
      status: test.status,
      variants: {},
    };

    // Lade Ergebnisse für jede Variante
    Object.keys(test.trafficSplit).forEach((variant) => {
      const resultFile = `${this.resultsPath}/${testId}_${variant}.json`;

      if (!fs.existsSync(resultFile)) {
        analysis.variants[variant] = { status: "no_data" };
        return;
      }

      const results = JSON.parse(fs.readFileSync(resultFile, "utf8"));

      if (results.length < test.minSampleSize) {
        analysis.variants[variant] = {
          status: "insufficient_data",
          sampleSize: results.length,
          required: test.minSampleSize,
        };
        return;
      }

      // Berechne Metriken
      const metrics = this.calculateVariantMetrics(results);
      analysis.variants[variant] = {
        status: "active",
        sampleSize: results.length,
        metrics,
      };
    });

    // Führe statistische Signifikanz-Tests durch
    if (
      this.allVariantsHaveSufficientData(analysis.variants, test.minSampleSize)
    ) {
      analysis.statisticalSignificance = this.performSignificanceTest(
        analysis.variants,
        test.metrics.primary,
        test.confidenceLevel,
      );

      analysis.winner = this.determineWinner(
        analysis.variants,
        test.metrics.primary,
      );
      analysis.recommendation = this.generateRecommendation(analysis);
    }

    return analysis;
  }

  // Berechne Metriken für Variante
  calculateVariantMetrics(results) {
    const trades = results.filter((r) => r.profit !== undefined);

    if (trades.length === 0) return null;

    const wins = trades.filter((t) => t.profit > 0);
    const losses = trades.filter((t) => t.profit <= 0);

    const totalProfit = trades.reduce((sum, t) => sum + t.profit, 0);
    const avgWin =
      wins.length > 0
        ? wins.reduce((sum, t) => sum + t.profit, 0) / wins.length
        : 0;
    const avgLoss =
      losses.length > 0
        ? Math.abs(losses.reduce((sum, t) => sum + t.profit, 0) / losses.length)
        : 0;

    const winrate = wins.length / trades.length;
    const profitFactor = avgLoss > 0 ? avgWin / avgLoss : 0;

    // Sharpe Ratio
    const returns = trades.map((t) => t.profit);
    const avgReturn = returns.reduce((sum, r) => sum + r, 0) / returns.length;
    const variance =
      returns.reduce((sum, r) => sum + Math.pow(r - avgReturn, 2), 0) /
      returns.length;
    const stdDev = Math.sqrt(variance);
    const sharpeRatio = stdDev > 0 ? avgReturn / stdDev : 0;

    return {
      totalProfit,
      winrate,
      profitFactor,
      avgWin,
      avgLoss,
      sharpeRatio,
      totalTrades: trades.length,
    };
  }

  // Prüfe ob alle Varianten genug Daten haben
  allVariantsHaveSufficientData(variants, minSize) {
    return Object.values(variants).every(
      (v) => v.sampleSize && v.sampleSize >= minSize,
    );
  }

  // Führe T-Test für statistische Signifikanz durch
  performSignificanceTest(variants, metric, confidenceLevel) {
    const variantNames = Object.keys(variants);
    if (variantNames.length < 2) return null;

    // Vereinfachter Two-Sample T-Test
    const baseVariant = variantNames[0];
    const testVariant = variantNames[1];

    const baseMetric = variants[baseVariant].metrics[metric];
    const testMetric = variants[testVariant].metrics[metric];

    const difference = testMetric - baseMetric;
    const relativeDifference =
      baseMetric !== 0 ? (difference / baseMetric) * 100 : 0;

    // Simplified significance check (in Produktion: echten T-Test verwenden)
    const isSignificant = Math.abs(relativeDifference) > 10; // > 10% Unterschied

    return {
      baseVariant,
      testVariant,
      metric,
      baseValue: baseMetric,
      testValue: testMetric,
      absoluteDifference: difference,
      relativeDifference: relativeDifference.toFixed(2) + "%",
      isSignificant,
      confidenceLevel,
    };
  }

  // Bestimme Gewinner-Variante
  determineWinner(variants, primaryMetric) {
    let bestVariant = null;
    let bestValue = -Infinity;

    Object.entries(variants).forEach(([name, data]) => {
      if (data.metrics && data.metrics[primaryMetric] > bestValue) {
        bestValue = data.metrics[primaryMetric];
        bestVariant = name;
      }
    });

    return {
      variant: bestVariant,
      value: bestValue,
      metric: primaryMetric,
    };
  }

  // Generiere Empfehlung
  generateRecommendation(analysis) {
    if (!analysis.statisticalSignificance) {
      return "Continue test - insufficient data for conclusion";
    }

    if (!analysis.statisticalSignificance.isSignificant) {
      return "No significant difference detected - consider extending test or using current strategy";
    }

    const winner = analysis.winner.variant;
    const improvement = analysis.statisticalSignificance.relativeDifference;

    return `Deploy variant ${winner} - shows ${improvement} improvement in ${analysis.statisticalSignificance.metric}`;
  }

  // Lade Test
  loadTest(testId) {
    const testFile = `${this.testsPath}/${testId}.json`;
    return JSON.parse(fs.readFileSync(testFile, "utf8"));
  }

  // Speichere Test
  saveTest(test) {
    const testFile = `${this.testsPath}/${test.id}.json`;
    fs.writeFileSync(testFile, JSON.stringify(test, null, 2));
  }

  // Schließe Test ab
  finalizeTest(testId) {
    const test = this.loadTest(testId);
    const analysis = this.analyzeTest(testId);

    test.status = "completed";
    test.endDate = new Date().toISOString();
    test.finalAnalysis = analysis;

    this.saveTest(test);
    return analysis;
  }
}

// Beispiel-Verwendung
const abFramework = new ABTestingFramework();

// Erstelle neuen Test
const testConfig = {
  name: "Risk Multiplier Test",
  description: "Testing risk multiplier values 1.0 vs 1.2",
  variants: [
    { name: "variant_A", config: { riskMultiplier: 1.0 } },
    { name: "variant_B", config: { riskMultiplier: 1.2 } },
  ],
  duration: 14, // 14 Tage
  minSampleSize: 50,
  primaryMetric: "profitFactor",
};

const result = abFramework.createTest(testConfig);

return [{ json: result }];
```

---

## 🔍 Abschnitt 4 – Drift-Detection und Regime-Change-Erkennung

Märkte ändern sich. Ein adaptives System muss erkennen, wann alte Strategien nicht mehr funktionieren.

### Drift-Detection-System

**Function-Node: Performance-Drift-Detector**

```javascript
// Drift-Detection: Erkenne Performance-Verschlechterung
const fs = require("fs");
const path = require("path");

class DriftDetector {
  constructor() {
    this.dataPath = "./data/performance";
    this.alertPath = "./alerts/drift_alerts.json";
    this.thresholds = {
      winrateDrift: 0.15, // 15% Abweichung
      profitFactorDrift: 0.2, // 20% Abweichung
      drawdownIncrease: 0.25, // 25% Erhöhung
      volatilityIncrease: 0.5, // 50% Erhöhung
    };
  }

  // Lade Performance-Daten für Zeitfenster
  loadTimeWindow(days) {
    const files = fs
      .readdirSync(this.dataPath)
      .filter((f) => f.endsWith(".json"))
      .sort();

    const startIdx = Math.max(0, files.length - days);
    const windowFiles = files.slice(startIdx);

    return windowFiles.map((file) => {
      const content = fs.readFileSync(path.join(this.dataPath, file), "utf8");
      return JSON.parse(content);
    });
  }

  // Berechne Metriken für Zeitfenster
  calculateWindowMetrics(data) {
    const trades = data.flatMap((d) => d.trades || []);

    if (trades.length === 0) return null;

    const wins = trades.filter((t) => t.profit > 0);
    const losses = trades.filter((t) => t.profit <= 0);

    const winrate = wins.length / trades.length;
    const avgWin =
      wins.length > 0
        ? wins.reduce((sum, t) => sum + t.profit, 0) / wins.length
        : 0;
    const avgLoss =
      losses.length > 0
        ? Math.abs(losses.reduce((sum, t) => sum + t.profit, 0) / losses.length)
        : 0;
    const profitFactor = avgLoss > 0 ? avgWin / avgLoss : 0;

    // Drawdown
    let peak = 0,
      maxDrawdown = 0,
      runningTotal = 0;
    trades.forEach((trade) => {
      runningTotal += trade.profit;
      if (runningTotal > peak) peak = runningTotal;
      const dd = peak > 0 ? (peak - runningTotal) / peak : 0;
      if (dd > maxDrawdown) maxDrawdown = dd;
    });

    // Volatilität
    const returns = trades.map((t) => t.profit);
    const avgReturn = returns.reduce((sum, r) => sum + r, 0) / returns.length;
    const variance =
      returns.reduce((sum, r) => sum + Math.pow(r - avgReturn, 2), 0) /
      returns.length;
    const volatility = Math.sqrt(variance);

    return {
      winrate,
      profitFactor,
      maxDrawdown,
      volatility,
      totalTrades: trades.length,
      totalProfit: trades.reduce((sum, t) => sum + t.profit, 0),
    };
  }

  // Erkenne Drift
  detectDrift() {
    const alerts = [];

    // Vergleiche aktuelle Performance (letzte 30 Tage) mit Baseline (30-90 Tage davor)
    const recentData = this.loadTimeWindow(30);
    const baselineData = this.loadTimeWindow(90).slice(0, 60); // Tag 30-90

    const recentMetrics = this.calculateWindowMetrics(recentData);
    const baselineMetrics = this.calculateWindowMetrics(baselineData);

    if (!recentMetrics || !baselineMetrics) {
      return {
        status: "insufficient_data",
        message: "Not enough data for drift detection",
      };
    }

    // Winrate Drift
    const winrateDrift =
      (baselineMetrics.winrate - recentMetrics.winrate) /
      baselineMetrics.winrate;
    if (Math.abs(winrateDrift) > this.thresholds.winrateDrift) {
      alerts.push({
        type: "winrate_drift",
        severity: winrateDrift > 0 ? "high" : "medium",
        baseline: baselineMetrics.winrate.toFixed(3),
        current: recentMetrics.winrate.toFixed(3),
        drift: (winrateDrift * 100).toFixed(1) + "%",
        message: `Winrate has ${winrateDrift > 0 ? "decreased" : "increased"} by ${Math.abs(winrateDrift * 100).toFixed(1)}%`,
      });
    }

    // Profit Factor Drift
    const pfDrift =
      (baselineMetrics.profitFactor - recentMetrics.profitFactor) /
      baselineMetrics.profitFactor;
    if (Math.abs(pfDrift) > this.thresholds.profitFactorDrift) {
      alerts.push({
        type: "profit_factor_drift",
        severity: pfDrift > 0 ? "high" : "low",
        baseline: baselineMetrics.profitFactor.toFixed(2),
        current: recentMetrics.profitFactor.toFixed(2),
        drift: (pfDrift * 100).toFixed(1) + "%",
        message: `Profit Factor has ${pfDrift > 0 ? "decreased" : "increased"} by ${Math.abs(pfDrift * 100).toFixed(1)}%`,
      });
    }

    // Drawdown Increase
    const ddIncrease =
      (recentMetrics.maxDrawdown - baselineMetrics.maxDrawdown) /
      baselineMetrics.maxDrawdown;
    if (ddIncrease > this.thresholds.drawdownIncrease) {
      alerts.push({
        type: "drawdown_increase",
        severity: "critical",
        baseline: (baselineMetrics.maxDrawdown * 100).toFixed(1) + "%",
        current: (recentMetrics.maxDrawdown * 100).toFixed(1) + "%",
        increase: (ddIncrease * 100).toFixed(1) + "%",
        message: `Max Drawdown increased by ${(ddIncrease * 100).toFixed(1)}% - Risk controls may need adjustment`,
      });
    }

    // Volatility Increase
    const volIncrease =
      (recentMetrics.volatility - baselineMetrics.volatility) /
      baselineMetrics.volatility;
    if (volIncrease > this.thresholds.volatilityIncrease) {
      alerts.push({
        type: "volatility_increase",
        severity: "medium",
        baseline: baselineMetrics.volatility.toFixed(2),
        current: recentMetrics.volatility.toFixed(2),
        increase: (volIncrease * 100).toFixed(1) + "%",
        message: `Volatility increased by ${(volIncrease * 100).toFixed(1)}% - Consider reducing position sizes`,
      });
    }

    // Regime Change Detection (kombinierte Signale)
    const regimeChangeScore = this.calculateRegimeChangeScore(alerts);

    if (regimeChangeScore > 0.7) {
      alerts.push({
        type: "regime_change",
        severity: "critical",
        score: regimeChangeScore.toFixed(2),
        message:
          "Multiple performance metrics degraded - Likely regime change detected. Strategy recalibration recommended.",
      });
    }

    // Speichere Alerts
    if (alerts.length > 0) {
      this.saveAlerts(alerts);
    }

    return {
      status: alerts.length > 0 ? "drift_detected" : "no_drift",
      timestamp: new Date().toISOString(),
      recentMetrics,
      baselineMetrics,
      alerts,
      regimeChangeScore,
      recommendation: this.generateRecommendation(alerts, regimeChangeScore),
    };
  }

  // Berechne Regime-Change-Score
  calculateRegimeChangeScore(alerts) {
    if (alerts.length === 0) return 0;

    const weights = {
      winrate_drift: 0.3,
      profit_factor_drift: 0.35,
      drawdown_increase: 0.25,
      volatility_increase: 0.1,
    };

    let score = 0;
    alerts.forEach((alert) => {
      const weight = weights[alert.type] || 0.1;
      const severityMultiplier =
        alert.severity === "critical"
          ? 1.5
          : alert.severity === "high"
            ? 1.2
            : 1.0;
      score += weight * severityMultiplier;
    });

    return Math.min(score, 1.0); // Cap at 1.0
  }

  // Generiere Empfehlung basierend auf Alerts
  generateRecommendation(alerts, regimeChangeScore) {
    if (alerts.length === 0) {
      return "Performance stable - Continue monitoring";
    }

    const criticalAlerts = alerts.filter((a) => a.severity === "critical");

    if (regimeChangeScore > 0.7) {
      return "CRITICAL: Regime change detected. Immediate strategy recalibration required. Consider halting trading until review completed.";
    }

    if (criticalAlerts.length > 0) {
      return `WARNING: ${criticalAlerts.length} critical alert(s) detected. Review risk parameters and consider reducing exposure.`;
    }

    return "Performance drift detected. Schedule strategy review within 48 hours.";
  }

  // Speichere Alerts
  saveAlerts(alerts) {
    const alertData = {
      timestamp: new Date().toISOString(),
      alerts,
      count: alerts.length,
    };

    try {
      let existingAlerts = [];
      if (fs.existsSync(this.alertPath)) {
        existingAlerts = JSON.parse(fs.readFileSync(this.alertPath, "utf8"));
      }

      existingAlerts.push(alertData);

      // Behalte nur letzte 100 Alert-Sets
      if (existingAlerts.length > 100) {
        existingAlerts = existingAlerts.slice(-100);
      }

      fs.writeFileSync(this.alertPath, JSON.stringify(existingAlerts, null, 2));
    } catch (error) {
      console.error("Failed to save alerts:", error.message);
    }
  }
}

// Ausführung
const driftDetector = new DriftDetector();
const result = driftDetector.detectDrift();

return [{ json: result }];
```

---

## 💡 Abschnitt 5 – LLM-basierte Strategie-Analyse

Nutze LLMs zur qualitativen Analyse historischer Trades und zur Identifikation nicht-offensichtlicher Muster.

### LLM-Analyse-Prompt für Trade-Patterns

**LLM-Node: Pattern-Recognition**

```javascript
// Vorbereitung der Daten für LLM-Analyse
const fs = require("fs");

class TradePatternAnalyzer {
  prepareSummary() {
    const performanceData = JSON.parse(
      fs.readFileSync("./data/performance/summary.json", "utf8"),
    );

    const summary = {
      totalTrades: performanceData.totalTrades,
      winrate: performanceData.winrate.toFixed(3),
      profitFactor: performanceData.profitFactor.toFixed(2),
      avgWinSize: performanceData.avgWin.toFixed(2),
      avgLossSize: performanceData.avgLoss.toFixed(2),
      maxDrawdown: (performanceData.maxDrawdown * 100).toFixed(1) + "%",
      recentLosses: performanceData.consecutiveLosses,
      volatilityTrend: performanceData.volatilityTrend,
    };

    return summary;
  }

  generateAnalysisPrompt(summary) {
    return `Analyze the following trading system performance data:

Total Trades: ${summary.totalTrades}
Win Rate: ${summary.winrate}
Profit Factor: ${summary.profitFactor}
Average Win: ${summary.avgWinSize}
Average Loss: ${summary.avgLossSize}
Max Drawdown: ${summary.maxDrawdown}
Recent Consecutive Losses: ${summary.recentLosses}
Volatility Trend: ${summary.volatilityTrend}

Tasks:
1. Identify any concerning patterns or anomalies
2. Assess whether the system is underperforming vs. expected behavior
3. Suggest specific parameter adjustments (if any)
4. Rate confidence level for each suggestion (0-100%)

Provide response in JSON format:
{
  "patterns_identified": ["pattern1", "pattern2"],
  "concerns": ["concern1", "concern2"],
  "adjustments": [
    {
      "parameter": "risk_multiplier",
      "current_implied": 1.0,
      "suggested": 0.8,
      "reason": "detailed reason",
      "confidence": 85
    }
  ],
  "overall_assessment": "brief summary"
}`;
  }
}

const analyzer = new TradePatternAnalyzer();
const summary = analyzer.prepareSummary();
const prompt = analyzer.generateAnalysisPrompt(summary);

return [{ json: { prompt } }];
```

**Integration in n8n:**

1. Function-Node bereitet Daten vor
2. LLM-Node (OpenAI/Anthropic) analysiert
3. Function-Node parst Vorschläge
4. Switch-Node: Bei confidence > 80% → Auto-Apply, sonst → Manual Review

---

## 🔧 Abschnitt 6 – Auto-Tuning mit Bayesian Optimization

Für fortgeschrittene Parameter-Optimierung kannst du Bayesian Optimization einsetzen.

### Konzept: Bayesian Optimization

Statt zufällig Parameter zu testen (Grid Search), lernt Bayesian Optimization aus vorherigen Tests und konzentriert sich auf vielversprechende Bereiche.

**Vereinfachter Bayesian Optimizer:**

```javascript
// Simplified Bayesian Optimization für Parameter-Tuning
const fs = require("fs");

class SimplifiedBayesianOptimizer {
  constructor() {
    this.historyPath = "./data/optimization_history.json";
    this.configPath = "./config/adaptive_params.json";
    this.parameters = {
      riskMultiplier: { min: 0.5, max: 2.0, step: 0.1 },
      stopLossMultiplier: { min: 0.8, max: 1.5, step: 0.05 },
      takeProfitMultiplier: { min: 1.0, max: 3.0, step: 0.1 },
    };
  }

  // Lade Optimierungs-Historie
  loadHistory() {
    try {
      return JSON.parse(fs.readFileSync(this.historyPath, "utf8"));
    } catch {
      return { trials: [] };
    }
  }

  // Speichere Optimierungs-Historie
  saveHistory(history) {
    fs.writeFileSync(this.historyPath, JSON.stringify(history, null, 2));
  }

  // Berechne Expected Improvement
  calculateExpectedImprovement(paramSet, history) {
    // Simplified: Basiere auf Distanz zu erfolgreichen Parametern
    const successfulTrials = history.trials.filter((t) => t.score > 0.7);

    if (successfulTrials.length === 0) {
      return Math.random(); // Exploration phase
    }

    // Berechne durchschnittliche Distanz zu erfolgreichen Parametern
    let totalDistance = 0;
    successfulTrials.forEach((trial) => {
      let distance = 0;
      Object.keys(paramSet).forEach((key) => {
        const normalized =
          (paramSet[key] - this.parameters[key].min) /
          (this.parameters[key].max - this.parameters[key].min);
        const trialNormalized =
          (trial.parameters[key] - this.parameters[key].min) /
          (this.parameters[key].max - this.parameters[key].min);
        distance += Math.pow(normalized - trialNormalized, 2);
      });
      totalDistance += Math.sqrt(distance);
    });

    const avgDistance = totalDistance / successfulTrials.length;

    // Kleinere Distanz = höherer Expected Improvement
    return 1 / (1 + avgDistance);
  }

  // Generiere nächsten Parameter-Kandidaten
  suggestNextParameters() {
    const history = this.loadHistory();

    // Generiere Kandidaten
    const candidates = [];
    for (let i = 0; i < 20; i++) {
      const candidate = {};
      Object.keys(this.parameters).forEach((key) => {
        const param = this.parameters[key];
        const range = param.max - param.min;
        const steps = Math.floor(range / param.step);
        const randomStep = Math.floor(Math.random() * steps);
        candidate[key] = param.min + randomStep * param.step;
      });

      const ei = this.calculateExpectedImprovement(candidate, history);
      candidates.push({ parameters: candidate, expectedImprovement: ei });
    }

    // Sortiere nach Expected Improvement
    candidates.sort((a, b) => b.expectedImprovement - a.expectedImprovement);

    return candidates[0].parameters;
  }

  // Speichere Trial-Ergebnis
  recordTrial(parameters, performance) {
    const history = this.loadHistory();

    // Berechne Score basierend auf Performance
    const score = this.calculateScore(performance);

    history.trials.push({
      timestamp: new Date().toISOString(),
      parameters,
      performance,
      score,
    });

    // Behalte nur letzte 100 Trials
    if (history.trials.length > 100) {
      history.trials = history.trials.slice(-100);
    }

    this.saveHistory(history);

    return score;
  }

  // Berechne Performance-Score
  calculateScore(performance) {
    // Weighted Score: Profit Factor (40%), Winrate (30%), Sharpe (30%)
    const pfScore = Math.min(performance.profitFactor / 2.0, 1.0) * 0.4;
    const wrScore = performance.winrate * 0.3;
    const srScore =
      Math.min(Math.max(performance.sharpeRatio, 0) / 2.0, 1.0) * 0.3;

    return pfScore + wrScore + srScore;
  }

  // Finde beste Parameter aus Historie
  getBestParameters() {
    const history = this.loadHistory();

    if (history.trials.length === 0) {
      return null;
    }

    let bestTrial = history.trials[0];
    history.trials.forEach((trial) => {
      if (trial.score > bestTrial.score) {
        bestTrial = trial;
      }
    });

    return bestTrial.parameters;
  }
}

// Verwendung
const optimizer = new SimplifiedBayesianOptimizer();
const nextParams = optimizer.suggestNextParameters();

return [{ json: { suggestedParameters: nextParams } }];
```

---

## 🚨 Abschnitt 7 – DEBUG-SEKTION: Adaptive Strategien

### Debug 1: Feedback-Loop funktioniert nicht

**Problem:** Konfiguration wird nicht aktualisiert trotz erkannter Performance-Probleme.

**Lösung:**

```javascript
// Prüfe Dateiberechtigungen
const fs = require("fs");

try {
  fs.accessSync("./config/adaptive_params.json", fs.constants.W_OK);
  console.log("✓ Config file is writable");
} catch (error) {
  console.error("✗ Config file not writable:", error.message);
  // Erstelle Verzeichnis falls nötig
  fs.mkdirSync("./config", { recursive: true });
}

// Prüfe ob minSampleSize erreicht
const performanceData = JSON.parse(
  fs.readFileSync("./data/performance/summary.json"),
);
console.log(`Trades: ${performanceData.totalTrades}, Required: 30`);

// Prüfe Confidence-Threshold
const adjustments = [
  /* ... */
];
const approved = adjustments.filter((a) => a.confidence >= 0.85);
console.log(
  `Total adjustments: ${adjustments.length}, Approved: ${approved.length}`,
);
```

### Debug 2: A/B-Test zeigt keine Varianten-Unterschiede

**Problem:** Beide Varianten haben identische Performance-Werte.

**Ursachen:**

- Traffic-Split nicht korrekt implementiert
- Varianten-Config wird nicht angewendet
- Zu kleine Sample-Size

**Lösung:**

```javascript
// Prüfe Traffic-Zuweisung
const testId = "abc123";
const assignments = {};

for (let i = 0; i < 1000; i++) {
  const variant = abFramework.assignVariant(testId, `trade_${i}`);
  assignments[variant] = (assignments[variant] || 0) + 1;
}

console.log("Traffic Distribution:", assignments);
// Sollte ca. 50/50 sein bei 2 Varianten

// Prüfe ob Varianten-Config angewendet wird
const variantConfig = test.variants[0].config;
console.log("Variant A Config:", variantConfig);
console.log(
  "Currently Applied Config:",
  JSON.parse(fs.readFileSync("./config/active_config.json")),
);
```

### Debug 3: Drift-Detection zu sensitiv / nicht sensitiv genug

**Problem:** Zu viele False Positives oder zu späte Erkennung.

**Lösung:** Threshold-Anpassung

```javascript
// In DriftDetector-Klasse:
this.thresholds = {
  winrateDrift: 0.15,      // Erhöhe für weniger Alerts
  profitFactorDrift: 0.20,
  drawdownIncrease: 0.25,
  volatilityIncrease: 0.50
};

// Füge Glättung hinzu (Moving Average)
calculateSmoothMetrics(data, windowSize = 7) {
  const metrics = [];

  for (let i = 0; i < data.length; i++) {
    const start = Math.max(0, i - windowSize + 1);
    const window = data.slice(start, i + 1);
    const windowMetrics = this.calculateWindowMetrics(window);
    metrics.push(windowMetrics);
  }

  return metrics;
}
```

### Debug 4: LLM-Vorschläge werden nicht geparst

**Problem:** JSON-Parsing schlägt fehl bei LLM-Response.

**Lösung:**

````javascript
// Robustes JSON-Parsing mit Fallback
function parseJsonSafely(text) {
  try {
    // Versuche direktes Parsing
    return JSON.parse(text);
  } catch (e1) {
    try {
      // Extrahiere JSON aus Markdown-Code-Block
      const match = text.match(/```json\n([\s\S]*?)\n```/);
      if (match) {
        return JSON.parse(match[1]);
      }
    } catch (e2) {
      try {
        // Extrahiere JSON zwischen { und }
        const start = text.indexOf("{");
        const end = text.lastIndexOf("}");
        if (start >= 0 && end > start) {
          return JSON.parse(text.substring(start, end + 1));
        }
      } catch (e3) {
        console.error("Failed to parse LLM response:", text);
        return null;
      }
    }
  }
  return null;
}

// Verwendung
const llmResponse = $json.output;
const parsed = parseJsonSafely(llmResponse);

if (parsed && parsed.adjustments) {
  return [{ json: parsed }];
} else {
  throw new Error("LLM response parsing failed - invalid format");
}
````

### Debug 5: Bayesian Optimizer konvergiert nicht

**Problem:** Vorgeschlagene Parameter verbessern Performance nicht.

**Lösung:**

```javascript
// Füge Exploration-Exploitation-Balance hinzu
suggestNextParameters(explorationRate = 0.2) {
  const history = this.loadHistory();

  // Mit Wahrscheinlichkeit explorationRate: zufällige Parameter
  if (Math.random() < explorationRate || history.trials.length < 10) {
    return this.generateRandomParameters();
  }

  // Sonst: Bayesian Optimization
  // ...
}

// Prüfe Score-Verteilung
const history = this.loadHistory();
const scores = history.trials.map(t => t.score);
console.log('Score Stats:', {
  min: Math.min(...scores),
  max: Math.max(...scores),
  avg: scores.reduce((a,b) => a+b, 0) / scores.length,
  count: scores.length
});

// Bei allen Scores < 0.5: Reset und neue Exploration
if (Math.max(...scores) < 0.5 && scores.length > 20) {
  console.log('Optimization stuck - resetting history');
  this.saveHistory({ trials: [] });
}
```

---

## 📝 Abschnitt 8 – Praxis-Übung: Vollständiger Adaptive-Learning-Cycle

### Übung 1: Implementiere wöchentlichen Performance-Review

**Aufgabe:** Erstelle einen n8n-Workflow, der jeden Sonntag:

1. Performance der letzten 7 Tage analysiert
2. Vergleich mit vorheriger Woche
3. Drift-Detection durchführt
4. Parameter-Anpassungen vorschlägt
5. Report per E-Mail sendet

**Workflow-Struktur:**

```
[Cron: Sonntag 00:00]
   ↓
[Function: Load Weekly Data]
   ↓
[Function: Calculate Metrics]
   ↓
[Function: Drift Detection]
   ↓
[Function: Suggest Adjustments]
   ↓
[IF: Critical Alerts?]
   ├─ Yes → [Send Immediate Alert]
   └─ No → [Generate Report]
        ↓
   [Send Email Node]
```

### Übung 2: A/B-Test Setup

**Aufgabe:** Teste 2 Risk-Multiplier-Varianten (1.0 vs. 1.2) über 2 Wochen.

**Schritte:**

1. Erstelle A/B-Test-Konfiguration
2. Implementiere Traffic-Split-Logic
3. Sammle Ergebnisse für 50+ Trades pro Variante
4. Führe Signifikanz-Test durch
5. Deploy Gewinner-Variante

**Erfolgs-Kriterium:** Statistisch signifikanter Unterschied (p < 0.05) im Profit Factor

### Übung 3: LLM-gestützte Strategie-Review

**Aufgabe:** Lasse LLM letzte 100 Trades analysieren und Muster identifizieren.

**Prompt-Template:**

```
Analysiere folgende 100 Trades:
[Trade-Daten als JSON]

Identifiziere:
1. Zeitpunkte mit gehäuften Verlusten
2. Korrelation zwischen Marktbedingungen und Performance
3. Optimal funktionierende vs. problematische Setups
4. Empfohlene Anpassungen

Format: Strukturierter Report mit konkreten Zahlen
```

---

## 📋 Hausaufgaben

### Aufgabe 1: Feedback-Loop Implementation (⭐⭐)

Implementiere den `PerformanceFeedbackLoop` aus Abschnitt 2 in deinem n8n-System.

**Anforderungen:**

- Läuft täglich automatisch
- Mindestens 30 Trades als Sample-Size
- Speichert Adjustment-History
- Logging aller Aktionen

**Abgabe:** Screenshot des Workflows + Beispiel-Log

### Aufgabe 2: A/B-Test Framework (⭐⭐⭐)

Implementiere das vollständige A/B-Testing-Framework und führe einen Test durch.

**Anforderungen:**

- 2 Varianten mit unterschiedlichen Parametern
- Mindestens 50 Trades pro Variante
- Statistische Auswertung
- Dokumentation in `AB_TEST_REPORT.md`

**Abgabe:** Test-Konfiguration, Ergebnisse, Entscheidung für Gewinner-Variante

### Aufgabe 3: Drift-Detection-System (⭐⭐⭐)

Implementiere Drift-Detection mit Alert-System.

**Anforderungen:**

- Vergleich 30-Tage vs. 60-Tage-Performance
- Mindestens 4 verschiedene Drift-Metriken
- Alert-System (E-Mail/Slack bei Critical)
- Wöchentlicher Drift-Report

**Abgabe:** Workflow-Export + Beispiel-Alert

### Bonus-Aufgabe: Bayesian Optimizer (⭐⭐⭐⭐)

Implementiere den Bayesian Optimizer und optimiere 3 Parameter gleichzeitig.

**Anforderungen:**

- Mindestens 50 Optimization-Trials
- Dokumentation der Score-Entwicklung
- Visualisierung der Parameter-Evolution
- Vergleich: Finale Performance vs. Start-Performance

**Abgabe:** Optimization-History, Performance-Vergleich, Lessons Learned

---

## ✅ Zusammenfassung

Nach Kapitel 14 kannst du:

✅ **Adaptive Systeme verstehen und implementieren**

- Unterschied zwischen statischen und adaptiven Systemen
- Kontrollierte Selbstanpassung vs. autonome Änderungen
- Best Practices für Adaptivität

✅ **Feedback-Loops aufbauen**

- Performance-Daten systematisch analysieren
- Parameter-Anpassungen basierend auf Metriken
- Confidence-basierte Auto-Apply-Logic

✅ **A/B-Testing durchführen**

- Strategievarianten objektiv vergleichen
- Traffic-Split und Varianten-Management
- Statistische Signifikanz bewerten

✅ **Drift-Detection implementieren**

- Performance-Verschlechterung frühzeitig erkennen
- Regime-Changes identifizieren
- Automatische Alerts und Empfehlungen

✅ **LLM-gestützte Analyse nutzen**

- Qualitative Pattern-Recognition
- Strukturierte Vorschläge generieren
- Robustes JSON-Parsing

✅ **Parameter-Optimierung automatisieren**

- Bayesian Optimization-Grundlagen
- Exploration-Exploitation-Balance
- Systematisches Trial-Management

✅ **Produktionsreife Debugging-Fähigkeiten**

- Häufige Probleme identifizieren und lösen
- Robuste Error-Handling-Strategien
- Performance-Monitoring und -Tuning

---

## 🔮 Ausblick auf Kapitel 15

Im nächsten Kapitel führen wir diese adaptiven Konzepte weiter: **Multi-Agent-Kollaboration**. Du lernst:

- Mehrere spezialisierte Agenten koordinieren
- Konsens-Mechanismen für Entscheidungen
- Hierarchische Agent-Architekturen
- Agent-to-Agent-Kommunikation
- Collective Intelligence für Trading-Systeme

Bis dahin: Implementiere mindestens den Feedback-Loop und experimentiere mit A/B-Tests! 🚀

```

```
