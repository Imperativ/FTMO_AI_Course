# 📘 Kapitel 12 – Evaluation, Skalierung und externe Datenintegration

**Lernziel:**
Nach dieser Lektion kannst du deinen FTMO-Agenten datengetrieben verbessern, externe Feeds anbinden und skalieren, ohne Stabilität oder Nachvollziehbarkeit zu verlieren.

---

## 🧩 Abschnitt 1 – Warum Evaluation der Schlüssel zu Skalierung ist

Bevor man skaliert, muss man verstehen, **was funktioniert**.
Sonst vergrößert man nur Fehlerquellen.

Evaluation bedeutet:

- Leistung unter realen Bedingungen messen
- Qualität der Signale prüfen
- Verbesserungspotenzial erkennen

Ein performanter Flow ist kein schneller Flow, sondern ein **lernfähiger**.
Ziel ist, dass dein Agent „weiß", wann er richtig liegt — und warum.

---

## ⚙️ Abschnitt 2 – Aufbau eines Evaluations-Flows

Erstelle in n8n einen neuen Workflow namens `FTMO_Evaluation`.

Grundstruktur:

```
[Cron Trigger] → [Query Data Store → letzte Trades]
   ↓
[Function → Berechnung von Winrate, Profitfaktor, Sharpe]
   ↓
[LLM Node → qualitative Bewertung]
   ↓
[Decision Node → Skalieren oder Beibehalten]
   ↓
[Telegram / Report]
```

### Beispiel Function-Node:

```javascript
const trades = $input.all().map((i) => i.json);
const pnl = trades.map((t) => t.profit || 0);
const wins = pnl.filter((v) => v > 0).length;
const losses = pnl.filter((v) => v <= 0).length;
const profitFactor =
  Math.abs(pnl.filter((v) => v > 0).reduce((a, b) => a + b, 0)) /
  Math.abs(pnl.filter((v) => v <= 0).reduce((a, b) => a + b, 0) || 1);

return [
  {
    winrate: wins / (wins + losses),
    profitFactor,
    totalTrades: trades.length,
    avgProfit: pnl.reduce((a, b) => a + b, 0) / trades.length,
  },
];
```

**Debugging-Hinweis:**
Vergleiche deine Kennzahlen mit FTMO-Berichten oder Demokonto-Logs, um Rechenfehler zu vermeiden.

---

## 🧠 Abschnitt 3 – Externe Feeds: TradingView, News und Sentiment

### TradingView

- Webhook-Integration mit **Alert Messages**
  (Beispiel: `{{strategy.order.comment}}:{{ticker}}:{{close}}`)
- In n8n über `[Webhook Trigger]` empfangen.
- Funktion zur Umwandlung in JSON:
  ```javascript
  const [type, symbol, price] = $json.body.split(":");
  return [{ type, symbol, price: parseFloat(price) }];
  ```

### News-Feeds

- API-Beispiel: [NewsAPI.org](https://newsapi.org/)
  ```bash
  GET https://newsapi.org/v2/everything?q=EURUSD&apiKey=$env.NEWS_KEY
  ```
- Filtere nur Headlines mit relevanten Schlagwörtern (`rate`, `inflation`, `recession`).

### Sentiment-APIs

- Kostenlose Alternativen: AYLIEN, FinBrain, Alternative.me
- Typische Antwortstruktur:
  ```json
  { "symbol": "BTC", "sentiment": 0.72, "source": "FinBrain" }
  ```

**Debugging-Hinweis:**
Alle externen Feeds sollten in **einen eigenen Workflow** mit dediziertem Logging laufen — sonst riskierst du Flaschenhälse.

---

## ⚙️ Abschnitt 4 – Skalierung: von lokal zu verteilt

Sobald dein Agent mehrere Symbole oder Feeds parallel verarbeitet, wird es Zeit für **parallele Ausführung**.

### Variante 1 – n8n in mehreren Instanzen

```bash
docker-compose scale n8n=3
```

Jede Instanz verarbeitet separate Queues (z. B. BTC, EUR, Gold).

### Variante 2 – n8n mit Redis-Queue

Aktiviere den Queue-Modus:

```bash
export N8N_EXECUTIONS_MODE=queue
export N8N_QUEUE_BULL_REDIS_HOST=localhost
```

Dadurch werden Tasks verteilt, aber zentral überwacht.

### Variante 3 – Subflows als Micro-Services

Große Flows in kleinere Einheiten zerlegen:

- `DataFetcher`
- `Analyzer`
- `Trader`
- `Reporter`

**Debugging-Tipp:**
Füge jedem Subflow eine eindeutige ID und Status-Variable hinzu, um asynchrone Ausführungen zu verfolgen.

---

## 🧩 Abschnitt 5 – Selbstüberwachung des Agenten

Ein skalierter Agent muss sich selbst prüfen können.

**Health-Node-Beispiel:**

```javascript
const metrics = {
  uptime: process.uptime(),
  pendingJobs: $execution.id,
  memory: process.memoryUsage().rss / 1e6,
};
if (metrics.memory > 500) {
  throw new Error("Memory usage high");
}
return [metrics];
```

Sende die Daten an deinen Telegram-Channel oder Prometheus-Collector.

---

## 💡 Abschnitt 6 – Debugging bei Skalierung

Skalierung erzeugt neue Fehlerarten:

- Race-Conditions (zwei Flows greifen gleichzeitig auf dieselbe Ressource zu)
- inkonsistente Logs
- doppelte Einträge

### Prävention:

- Verwende eindeutige IDs (`uuid.v4()`) für jeden Trade.
- Sperre kritische Abschnitte mit Lock-Nodes (z. B. Redis-Lock oder Dateisperre).
- Prüfe regelmäßig auf doppelte Timestamps im Data Store.

---

## 🧩 Abschnitt 7 – Praxis: Evaluation + Feed-Integration

1. Aktiviere einen TradingView-Webhook mit Signalen für 3 Symbole.
2. Erstelle in n8n den Flow „FTMO_Evaluation“.
3. Kombiniere Market-Daten mit Sentiment-Score in einem JSON-Merge.
4. Berechne daraus neue Risikogewichte.
5. Logge alle Ergebnisse und schicke tägliche Reports.

**Debugging-Hinweis:**
Fehler „429 Too Many Requests“ → API-Limit überschritten. Lösung: Pausen oder Proxy-Rotation.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche externen Daten haben den größten Einfluss auf deine Ergebnisse?
- Wo siehst du Grenzen des Automatisierungsgrads (z. B. menschliche Einschätzung nötig)?
- Wie könntest du deine Evaluations-Metriken auf längere Zeitreihen ausdehnen?

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Baue einen kombinierten Feed-Flow mit TradingView + NewsAPI.
2. Erweitere den Evaluations-Flow um Profitfaktor-Analyse.
3. Teste Skalierung mit Redis-Queue oder mehreren Docker-Instanzen.
4. Prüfe Logs auf Latenzunterschiede.
5. Erstelle aus den Ergebnissen einen grafischen Vergleich (z. B. mit Recharts).

---

---

## 🚨 Abschnitt 10 – Evaluation & Skalierungs-Debugging

### Performance-Metriken Validierung

**Problem:** Evaluations-Berechnungen liefern unrealistische Werte
**Robuste Metriken-Berechnung:**

```javascript
// Sichere Evaluation mit Fehlerbehandlung
const calculateTradeMetrics = (trades) => {
  if (!Array.isArray(trades) || trades.length === 0) {
    return {
      error: "No valid trades data",
      winrate: 0,
      profitFactor: 0,
      sharpeRatio: 0,
    };
  }

  const validTrades = trades.filter(
    (t) => t.profit !== undefined && t.profit !== null && !isNaN(t.profit),
  );

  if (validTrades.length === 0) {
    return { error: "No trades with valid profit data" };
  }

  const profits = validTrades.map((t) => t.profit);
  const wins = profits.filter((p) => p > 0);
  const losses = profits.filter((p) => p < 0);

  const winrate = validTrades.length > 0 ? wins.length / validTrades.length : 0;

  const totalWins = wins.reduce((sum, p) => sum + p, 0);
  const totalLosses = Math.abs(losses.reduce((sum, p) => sum + p, 0));
  const profitFactor = totalLosses > 0 ? totalWins / totalLosses : 0;

  // Sharpe Ratio (vereinfacht)
  const avgReturn = profits.reduce((sum, p) => sum + p, 0) / profits.length;
  const variance =
    profits.reduce((sum, p) => sum + Math.pow(p - avgReturn, 2), 0) /
    profits.length;
  const stdDev = Math.sqrt(variance);
  const sharpeRatio = stdDev > 0 ? (avgReturn / stdDev) * Math.sqrt(252) : 0;

  return {
    winrate: Math.round(winrate * 10000) / 100,
    profitFactor: Math.round(profitFactor * 100) / 100,
    sharpeRatio: Math.round(sharpeRatio * 100) / 100,
    totalTrades: validTrades.length,
    avgProfit: Math.round(avgReturn * 100) / 100,
    totalProfit: Math.round(profits.reduce((a, b) => a + b, 0) * 100) / 100,
  };
};
```

### Load Testing für skalierte Systeme

**Problem:** System bricht unter Last zusammen
**Load-Test-Framework:**

```javascript
// n8n Load Testing Script
const runLoadTest = async (config) => {
  const {
    concurrentRequests = 10,
    totalRequests = 100,
    endpoint = "http://localhost:5678/webhook/test",
  } = config;

  const results = {
    successful: 0,
    failed: 0,
    totalTime: 0,
    responseTimes: [],
    errors: [],
  };

  const startTime = Date.now();
  const batches = Math.ceil(totalRequests / concurrentRequests);

  for (let batch = 0; batch < batches; batch++) {
    const batchRequests = [];

    for (let i = 0; i < concurrentRequests; i++) {
      const requestStart = Date.now();

      const request = fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          test: true,
          id: batch * concurrentRequests + i,
        }),
      })
        .then((response) => {
          const responseTime = Date.now() - requestStart;
          results.responseTimes.push(responseTime);

          if (response.ok) {
            results.successful++;
          } else {
            results.failed++;
            results.errors.push({
              status: response.status,
              message: response.statusText,
            });
          }
        })
        .catch((error) => {
          results.failed++;
          results.errors.push({ message: error.message });
        });

      batchRequests.push(request);
    }

    await Promise.all(batchRequests);

    // Pause zwischen Batches
    if (batch < batches - 1) {
      await new Promise((resolve) => setTimeout(resolve, 100));
    }
  }

  results.totalTime = Date.now() - startTime;
  results.avgResponseTime =
    results.responseTimes.length > 0
      ? results.responseTimes.reduce((a, b) => a + b) /
        results.responseTimes.length
      : 0;
  results.requestsPerSecond =
    (results.successful + results.failed) / (results.totalTime / 1000);

  return results;
};
```

### Redis Queue Debugging

**Problem:** Messages bleiben in Queue stecken
**Queue-Monitor:**

```javascript
// Redis Queue Health Monitor
const monitorQueue = async (redisClient) => {
  const queueStatus = {
    waiting: 0,
    active: 0,
    completed: 0,
    failed: 0,
    delayed: 0,
  };

  try {
    // Check verschiedene Queue-Stati
    queueStatus.waiting = await redisClient.llen("bull:n8n:wait");
    queueStatus.active = await redisClient.llen("bull:n8n:active");
    queueStatus.completed = await redisClient.zcard("bull:n8n:completed");
    queueStatus.failed = await redisClient.zcard("bull:n8n:failed");
    queueStatus.delayed = await redisClient.zcard("bull:n8n:delayed");

    // Alert bei Problemen
    if (queueStatus.failed > 10) {
      console.error("⚠️ High number of failed jobs:", queueStatus.failed);
    }

    if (queueStatus.waiting > 100) {
      console.warn("⚠️ Queue backlog detected:", queueStatus.waiting);
    }

    return queueStatus;
  } catch (error) {
    return { error: error.message, ...queueStatus };
  }
};
```

### External Feed Integration Debugging

**Problem:** Externe Datenfeeds liefern inkonsistente Daten
**Feed-Validator:**

```javascript
// External Data Feed Validator
const validateFeedData = (feedData, feedType) => {
  const validation = {
    valid: true,
    errors: [],
    warnings: [],
  };

  // TradingView Feed Validation
  if (feedType === "tradingview") {
    if (!feedData.symbol || typeof feedData.symbol !== "string") {
      validation.errors.push("Missing or invalid symbol");
    }
    if (!feedData.price || isNaN(feedData.price)) {
      validation.errors.push("Missing or invalid price");
    }
    if (feedData.price < 0) {
      validation.errors.push("Negative price detected");
    }
  }

  // News Feed Validation
  if (feedType === "news") {
    if (!feedData.articles || !Array.isArray(feedData.articles)) {
      validation.errors.push("Invalid articles format");
    }
    if (feedData.articles && feedData.articles.length === 0) {
      validation.warnings.push("No articles returned");
    }
  }

  // Sentiment Feed Validation
  if (feedType === "sentiment") {
    if (feedData.sentiment === undefined || isNaN(feedData.sentiment)) {
      validation.errors.push("Missing sentiment score");
    }
    if (feedData.sentiment < -1 || feedData.sentiment > 1) {
      validation.errors.push("Sentiment score out of range [-1, 1]");
    }
  }

  // Timestamp-Check (Daten nicht zu alt)
  if (feedData.timestamp) {
    const age = Date.now() - new Date(feedData.timestamp).getTime();
    if (age > 3600000) {
      // 1 Stunde
      validation.warnings.push(
        `Data is ${Math.floor(age / 60000)} minutes old`,
      );
    }
  }

  validation.valid = validation.errors.length === 0;
  return validation;
};
```

### Distributed System Race Condition Prevention

**Problem:** Mehrere Instanzen greifen gleichzeitig auf Ressourcen zu
**Distributed Lock Implementation:**

```javascript
// Redis-basierte Distributed Lock
class DistributedLock {
  constructor(redisClient, lockKey, ttl = 5000) {
    this.redis = redisClient;
    this.lockKey = `lock:${lockKey}`;
    this.ttl = ttl;
    this.lockId = `${Date.now()}-${Math.random()}`;
  }

  async acquire() {
    const result = await this.redis.set(
      this.lockKey,
      this.lockId,
      "PX", // Milliseconds
      this.ttl,
      "NX", // Only set if not exists
    );

    return result === "OK";
  }

  async release() {
    // Nur eigenen Lock freigeben
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
      else
        return 0
      end
    `;

    return await this.redis.eval(script, 1, this.lockKey, this.lockId);
  }

  async extend(additionalTime) {
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("pexpire", KEYS[1], ARGV[2])
      else
        return 0
      end
    `;

    return await this.redis.eval(
      script,
      1,
      this.lockKey,
      this.lockId,
      additionalTime,
    );
  }
}

// Usage in n8n
const lock = new DistributedLock(redisClient, "trade-execution");
if (await lock.acquire()) {
  try {
    // Critical section - nur eine Instanz führt das aus
    await executeTrade($json);
  } finally {
    await lock.release();
  }
} else {
  console.log("Lock already held by another instance, skipping...");
}
```

---

## ✅ Zusammenfassung

Nach Kapitel 12 kannst du:

- Evaluations-Flows aufbauen und Metriken automatisieren,
- externe Datenquellen in deinen Agenten einbinden,
- dein System skalieren und verteilen,
- Load-Tests durchführen und Performance-Bottlenecks identifizieren,
- Race-Conditions mit Distributed Locks verhindern,
- Debugging-Strategien für verteilte Umgebungen anwenden,
- und datengestützt Verbesserungen ableiten.

Das nächste Kapitel (13) wird den **Abschlussblock der Kursmappe** bilden: Dort lernst du, wie du dein gesamtes Projekt als **vollständiges Systemdossier** exportierst, dokumentierst und für eine Präsentation oder Bewerbung (z. B. bei FTMO oder in deinem Portfolio) aufbereitest.
