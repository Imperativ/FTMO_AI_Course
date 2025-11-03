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
Ziel ist, dass dein Agent „weiß“, wann er richtig liegt — und warum.

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
const trades = $input.all().map(i => i.json);
const pnl = trades.map(t => t.profit || 0);
const wins = pnl.filter(v => v > 0).length;
const losses = pnl.filter(v => v <= 0).length;
const profitFactor = Math.abs(pnl.filter(v => v > 0).reduce((a,b)=>a+b,0)) /
                     Math.abs(pnl.filter(v => v <= 0).reduce((a,b)=>a+b,0) || 1);

return [{
  winrate: wins / (wins + losses),
  profitFactor,
  totalTrades: trades.length,
  avgProfit: pnl.reduce((a,b)=>a+b,0) / trades.length
}];
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
  {"symbol": "BTC", "sentiment": 0.72, "source": "FinBrain"}
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
  memory: process.memoryUsage().rss / 1e6
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

## ✅ Zusammenfassung

Nach Kapitel 12 kannst du:
- Evaluations-Flows aufbauen und Metriken automatisieren,  
- externe Datenquellen in deinen Agenten einbinden,  
- dein System skalieren und verteilen,  
- Debugging-Strategien für verteilte Umgebungen anwenden,  
- und datengestützt Verbesserungen ableiten.  

Das nächste Kapitel (13) wird den **Abschlussblock der Kursmappe** bilden: Dort lernst du, wie du dein gesamtes Projekt als **vollständiges Systemdossier** exportierst, dokumentierst und für eine Präsentation oder Bewerbung (z. B. bei FTMO oder in deinem Portfolio) aufbereitest.