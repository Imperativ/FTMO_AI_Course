# 📘 Kapitel 17 – Test- und Simulationsumgebungen für KI-Agenten und Handelssysteme

**Lernziel:**
Nach dieser Lektion kannst du dein Agentensystem unter kontrollierten Bedingungen testen, Marktstrategien reproduzierbar simulieren und fehlerfreie Backtests durchführen.

---

## 🧩 Abschnitt 1 – Warum Simulationen unverzichtbar sind

Ein System, das direkt mit echtem Kapital getestet wird, ist kein Test – es ist Glücksspiel.
Professionelle Automatisierer führen Simulationen durch, um Risiken vorherzusehen, Schwächen zu erkennen und Strategien objektiv zu bewerten.

### Wichtige Ziele

- **Reproduzierbarkeit:** Gleiche Eingaben → gleiche Ergebnisse
- **Validierung:** Prüfen, ob Entscheidungen logisch konsistent sind
- **Fehlerdiagnose:** Jedes Zwischenergebnis kann analysiert werden
- **Optimierung:** Parameter iterativ anpassen, ohne Risiko
- **Risiko-Bewertung:** Worst-Case-Szenarien testen

### Simulation vs. Live-Trading

| Aspekt          | Simulation          | Live        |
| --------------- | ------------------- | ----------- |
| Risiko          | Kein echtes Kapital | Reales Geld |
| Geschwindigkeit | Beliebig schnell    | Echtzeit    |
| Daten           | Historisch          | Aktuell     |
| Slippage        | Geschätzt           | Real        |
| Emotionen       | Keine               | Faktor      |

---

## ⚙️ Abschnitt 2 – Historische Marktdaten beschaffen

### Datenquellen

- **Binance API:** Bis zu 1000 Kerzen/Abfrage (kostenlos)
- **Yahoo Finance:** Tagesdaten für Aktien/Indizes
- **Oanda/FXCM:** Forex-Daten mit hoher Qualität
- **CryptoDataDownload:** CSV-Archive für Krypto

### Binance API Implementation

```javascript
// Historical Data Downloader
const axios = require("axios");

async function downloadData(symbol, interval, limit = 1000) {
  const url = `https://api.binance.com/api/v3/klines`;
  const response = await axios.get(url, {
    params: { symbol, interval, limit },
  });

  return response.data.map((k) => ({
    timestamp: k[0],
    open: parseFloat(k[1]),
    high: parseFloat(k[2]),
    low: parseFloat(k[3]),
    close: parseFloat(k[4]),
    volume: parseFloat(k[5]),
  }));
}

// Verwendung mit Rate-Limiting
const data = await downloadData("BTCUSDT", "1m");
await new Promise((r) => setTimeout(r, 1000)); // Rate limit
return [{ json: { candles: data } }];
```

---

## 🧠 Abschnitt 3 – Backtesting-Engine

### Grundstruktur

```javascript
// Backtesting Engine
class BacktestEngine {
  constructor(initialCapital = 10000) {
    this.capital = initialCapital;
    this.initialCapital = initialCapital;
    this.position = null;
    this.trades = [];
    this.equity = [initialCapital];
  }

  executeTrade(signal, price, timestamp) {
    const fee = 0.001; // 0.1%
    const slippage = 0.0003; // 0.03%

    if (signal === "BUY" && !this.position) {
      const cost = price * (1 + fee + slippage);
      const quantity = this.capital / cost;

      this.position = {
        type: "LONG",
        entryPrice: cost,
        quantity: quantity,
        entryTime: timestamp,
      };

      this.capital = 0;
    } else if (signal === "SELL" && this.position?.type === "LONG") {
      const revenue = price * (1 - fee - slippage);
      const pnl = (revenue - this.position.entryPrice) * this.position.quantity;

      this.capital = this.position.quantity * revenue;

      this.trades.push({
        entryPrice: this.position.entryPrice,
        exitPrice: revenue,
        quantity: this.position.quantity,
        pnl: pnl,
        duration: timestamp - this.position.entryTime,
        returnPct: (revenue / this.position.entryPrice - 1) * 100,
      });

      this.position = null;
    }

    // Berechne aktuelles Equity
    let currentEquity = this.capital;
    if (this.position) {
      currentEquity = this.position.quantity * price;
    }

    this.equity.push(currentEquity);
  }

  getMetrics() {
    const wins = this.trades.filter((t) => t.pnl > 0);
    const losses = this.trades.filter((t) => t.pnl <= 0);

    const totalPnl = this.trades.reduce((sum, t) => sum + t.pnl, 0);
    const winrate = wins.length / this.trades.length;

    const avgWin =
      wins.length > 0 ? wins.reduce((s, t) => s + t.pnl, 0) / wins.length : 0;
    const avgLoss =
      losses.length > 0
        ? Math.abs(losses.reduce((s, t) => s + t.pnl, 0) / losses.length)
        : 0;
    const profitFactor = avgLoss > 0 ? avgWin / avgLoss : 0;

    // Max Drawdown
    let peak = this.equity[0],
      maxDD = 0;
    this.equity.forEach((eq) => {
      if (eq > peak) peak = eq;
      const dd = (peak - eq) / peak;
      if (dd > maxDD) maxDD = dd;
    });

    // Sharpe Ratio (vereinfacht)
    const returns = this.trades.map((t) => t.returnPct);
    const avgReturn = returns.reduce((a, b) => a + b, 0) / returns.length;
    const variance =
      returns.reduce((s, r) => s + Math.pow(r - avgReturn, 2), 0) /
      returns.length;
    const sharpe =
      Math.sqrt(variance) > 0 ? avgReturn / Math.sqrt(variance) : 0;

    return {
      totalTrades: this.trades.length,
      winrate: winrate,
      profitFactor: profitFactor,
      totalPnl: totalPnl,
      totalReturn: (this.capital / this.initialCapital - 1) * 100,
      maxDrawdown: maxDD,
      sharpeRatio: sharpe,
      avgWin: avgWin,
      avgLoss: avgLoss,
    };
  }
}

// Verwendung
const engine = new BacktestEngine(10000);
// Feed mit historischen Daten...
const metrics = engine.getMetrics();
return [{ json: metrics }];
```

---

## 💡 Abschnitt 4 – Monte-Carlo-Simulation

```javascript
// Monte Carlo Simulation - teste Robustheit
function monteCarloSimulation(trades, runs = 1000) {
  const results = [];

  for (let i = 0; i < runs; i++) {
    const shuffled = [...trades].sort(() => Math.random() - 0.5);
    let equity = 10000,
      peak = equity,
      maxDD = 0;

    shuffled.forEach((trade) => {
      equity += trade.pnl;
      if (equity > peak) peak = equity;
      const dd = (peak - equity) / peak;
      if (dd > maxDD) maxDD = dd;
    });

    results.push({ finalEquity: equity, maxDrawdown: maxDD });
  }

  results.sort((a, b) => a.finalEquity - b.finalEquity);

  return {
    worst: results[0],
    best: results[results.length - 1],
    median: results[Math.floor(results.length / 2)],
  };
}
```

---

## 🚨 Abschnitt 5 – Debug-Sektion

### Debug 1: Unrealistisch hohe Gewinne

**Problem:** Backtest zeigt 500% Gewinn in einer Woche.

**Ursachen:**

- Fehlende Gebühren
- Kein Slippage
- Look-ahead Bias (Zukunftsdaten verwendet)
- Übermäßiges Leverage

**Lösung:**

```javascript
// Realistische Kosten einbauen
const tradingCosts = {
  fee: 0.001, // 0.1%
  slippage: 0.0003, // 0.03%
  minFee: 0.1, // Mindestgebühr
};

const totalCost = Math.max(price * quantity * (fee + slippage), minFee);
```

### Debug 2: Equity-Kurve bricht plötzlich ein

**Problem:** Backtest läuft gut, dann Totalverlust.

**Ursachen:**

- Fehlende Stop-Loss-Implementation
- Margin Call nicht simuliert
- Daten-Lücke in Historie

**Lösung:**

```javascript
// Stop-Loss prüfen
if (position && price <= position.entryPrice * (1 - stopLossPct)) {
  executeTrade("SELL", price, timestamp);
  console.log("Stop-Loss triggered");
}
```

### Debug 3: Divergierende Ergebnisse

**Problem:** Gleiche Daten, unterschiedliche Ergebnisse.

**Lösung:** Verwende festen Random-Seed für Reproduzierbarkeit.

```javascript
// Seeded Random
const seed = 42;
Math.seedrandom = (s) => {
  /* Implementation */
};
Math.seedrandom(seed);
```

---

## 📊 Abschnitt 6 – Walk-Forward-Analyse

```javascript
// Walk-Forward Analysis - verhindert Overfitting
function walkForward(data, inPeriod, outPeriod) {
  const results = [];

  for (let i = 0; i < data.length - inPeriod - outPeriod; i += outPeriod) {
    const inSample = data.slice(i, i + inPeriod);
    const outSample = data.slice(i + inPeriod, i + inPeriod + outPeriod);

    const params = optimize(inSample);
    const perf = backtest(outSample, params);

    results.push({
      inReturn: params.return,
      outReturn: perf.return,
      degradation: params.return - perf.return,
    });
  }

  return results;
}
```

---

## 📋 Hausaufgaben

**Aufgabe 1: Backtest-Engine (⭐⭐⭐)**

- Implementiere vollständige Backtesting-Engine
- Unterstütze Long- und Short-Positionen
- Berechne alle Metriken (Sharpe, Drawdown, etc.)
- Teste mit 7 Tagen BTC-Daten

**Aufgabe 2: Monte-Carlo-Simulation (⭐⭐⭐)**

- Führe 1000 Monte-Carlo-Runs durch
- Visualisiere Equity-Verteilung
- Berechne 5%/95% Percentile
- Dokumentiere Worst-Case-Szenario

**Aufgabe 3: Walk-Forward-Test (⭐⭐⭐⭐)**

- Implementiere Walk-Forward-Analyse
- In-Sample: 60 Tage, Out-Sample: 30 Tage
- Vergleiche In-Sample vs. Out-Sample Performance
- Erstelle Report über Degradation

---

## ✅ Zusammenfassung

Nach Kapitel 17 kannst du:

- historische Marktdaten korrekt einlesen und verarbeiten,
- vollständige Backtesting-Engines mit realistischen Kosten implementieren,
- Monte-Carlo-Simulationen für Robustheitstests durchführen,
- Walk-Forward-Analysen zur Overfitting-Vermeidung nutzen,
- häufige Backtest-Fehler identifizieren und beheben,
- und reproduzierbare Test-Pipelines aufbauen.

Im nächsten Kapitel (18) geht es um **Ethik und Aufsicht** – wie du dein System verantwortungsvoll betreibst und regulatorische Anforderungen erfüllst.
