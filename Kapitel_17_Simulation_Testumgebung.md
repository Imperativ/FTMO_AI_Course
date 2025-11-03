# 📘 Kapitel 17 – Test- und Simulationsumgebungen für KI-Agenten und Handelssysteme

**Lernziel:**  
Nach dieser Lektion kannst du dein Agentensystem unter kontrollierten Bedingungen testen, Marktstrategien reproduzierbar simulieren und fehlerfreie Backtests durchführen.  
Du lernst, wie man historische Daten korrekt einliest, Strategien bewertet, Testläufe automatisiert und Debugging-Informationen so aufzeichnet, dass sich jedes Ergebnis lückenlos nachvollziehen lässt.

---

## 🧩 Abschnitt 1 – Warum Simulationen unverzichtbar sind

Ein System, das direkt mit echtem Kapital getestet wird, ist kein Test – es ist Glücksspiel.  
Professionelle Automatisierer führen Simulationen durch, um Risiken vorherzusehen, Schwächen zu erkennen und Strategien objektiv zu bewerten.  

Wichtige Ziele:
- **Reproduzierbarkeit:** gleiche Eingaben → gleiche Ergebnisse.  
- **Validierung:** Prüfen, ob Entscheidungen logisch konsistent sind.  
- **Fehlerdiagnose:** jedes Zwischenergebnis kann analysiert werden.  
- **Optimierung:** Parameter iterativ anpassen, ohne Risiko.  

---

## ⚙️ Abschnitt 2 – Beschaffung historischer Marktdaten

Verlässliche Datenquellen:
- **Binance API** → bis zu 1000 Kerzen / Abfrage.  
- **Yahoo Finance** → Tagesdaten für Aktien und Indizes.  
- **Oanda / FXCM** → Forex-Daten.  
- **CryptoDataDownload / Kaggle** → CSV-Archive.  

**Beispiel REST-Abfrage (Binance):**
```bash
GET https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1m&limit=1000
```

**n8n-Variante:**
1. Node „HTTP Request“ → URL wie oben.  
2. Function-Node:  
```javascript
return $json.map(k => ({
  openTime: k[0],
  open: parseFloat(k[1]),
  high: parseFloat(k[2]),
  low: parseFloat(k[3]),
  close: parseFloat(k[4]),
  volume: parseFloat(k[5])
}));
```

**Debugging-Hinweis:**  
Fehler „429 Too Many Requests“ → Rate-Limit erreicht.  
→ Lösung: `await new Promise(r => setTimeout(r, 2000));` zwischen Requests.

---

## 🧠 Abschnitt 3 – Backtesting-Workflow in n8n

Ziel: deine Strategie so ausführen, als würde sie live handeln – aber ausschließlich auf gespeicherten Daten.

**Struktur:**
```
[Read Data] → [Preprocessing] → [Agent Decision] → [Simulated Execution] → [Log + Metrics]
```

**Implementation:**
1. Node „Read Binary File“ → CSV einlesen.  
2. Node „Spreadsheet File“ oder „Split In Batches“ → eine Kerze = ein Tick.  
3. Node „Function“ → an Agent-Flow senden:  
```javascript
for (const candle of $json.data) {
  sendToAgent(candle);
  await new Promise(r => setTimeout(r, 100)); // simulierte Zeit
}
```

**Debugging-Tipp:**  
Wenn Speicherverbrauch > 500 MB, verwende Stream-Verarbeitung (`readline` oder Batch-Size < 100).

---

## 💡 Abschnitt 4 – Virtuelle Handels-Engine

Die „Execution-Simulation“ ersetzt den Broker-Server.

```javascript
function executeTrade(signal, price) {
  const fee = 0.001; // 0,1 %
  const fill = price * (1 + (signal === "BUY" ? fee : -fee));
  return { fill, pnl: signal === "BUY" ? -fee : fee };
}
```

**Debugging-Hinweis:**  
Unrealistisch hohe Gewinne → meist fehlende Gebühren oder Slippage ignoriert.  
Slippage = Preisabweichung zwischen Signal und Ausführung.  

```javascript
const slippage = price * 0.0003; // 0.03 %
```

---

## ⚙️ Abschnitt 5 – Bewertungsmetriken

Zentrale Kennzahlen:
| Kennzahl | Bedeutung |
|-----------|-----------|
| **Winrate** | Anteil erfolgreicher Trades |
| **Profitfaktor** | Summe Gewinne / Summe Verluste |
| **Max Drawdown** | größter Kapitalrückgang |
| **Sharpe Ratio** | Rendite / Volatilität |
| **Expectancy** | Erwartungswert pro Trade |

**Beispiel-Berechnung:**
```javascript
const pnl = $json.trades.map(t => t.pnl);
const avg = pnl.reduce((a,b)=>a+b,0)/pnl.length;
const std = Math.sqrt(pnl.map(x=>(x-avg)**2).reduce((a,b)=>a+b)/pnl.length);
return [{ sharpe: avg/std }];
```

**Debugging-Hinweis:**  
Wenn `std=0` → keine Varianz → Testdaten zu kurz oder identische P&L-Werte.

---

## 🧩 Abschnitt 6 – Parameter-Optimierung

Nutze n8n-Loops oder externe Skripte, um Strategien mit verschiedenen Parametern zu testen:

```javascript
for (let stopLoss of [0.5, 1, 2]) {
  for (let takeProfit of [1, 2, 3]) {
    runTest({ stopLoss, takeProfit });
  }
}
```

Ergebnisse in JSON speichern und vergleichen:
```bash
jq '.[] | {stopLoss, takeProfit, sharpe}' results.json
```

**Debugging-Tipp:**  
Bei großen Parameterräumen → Parallelisierung aktivieren (`n8n queue mode`) und CPU-Last beobachten.

---

## 💡 Abschnitt 7 – Regressionstests

Damit spätere Änderungen deinen Flow nicht unbemerkt verfälschen:

```bash
n8n execute --id=23 --input=./data/btcusdt_test.json
```

Anschließend Output mit früherer Version vergleichen:
```bash
diff -u results_old.json results_new.json
```

**Debugging-Hinweis:**  
Unterschiede nur durch Rundungsfehler? → Float auf 4 Stellen runden (`toFixed(4)`).

---

## ⚙️ Abschnitt 8 – Visualisierung der Ergebnisse

Mit n8n-Chart-Node oder lokal via Python (Matplotlib).  
Beispiel:  
```python
import pandas as pd, matplotlib.pyplot as plt
df = pd.read_csv('results.csv')
plt.plot(df['time'], df['equity'])
plt.title('Equity Curve')
plt.show()
```

**Debugging-Tipp:**  
Abweichende Kurven → Zeitzonen oder Candle-Lücken prüfen (`NaN`-Werte interpolieren).

---

## 🧠 Abschnitt 9 – Automatisierte Reports

Am Ende jedes Testlaufs:
```javascript
return [{
  date: new Date().toISOString(),
  sharpe: sharpe,
  trades: pnl.length,
  max_drawdown: mdd,
  notes: "Backtest abgeschlossen"
}];
```
Dann per E-Mail oder Telegram senden:
```bash
sendTelegram("Backtest abgeschlossen: Sharpe =" + sharpe);
```

---

## 🧭 Abschnitt 10 – Reflexion

- Wann sind Simulationsergebnisse trügerisch?  
- Wie erkennst du Overfitting in deiner Strategie?  
- Welche Parameter würdest du bei Live-Daten anders wählen?  
- Wie dokumentierst du deine Tests, um sie Jahre später noch zu verstehen?

---

## 🧩 Abschnitt 11 – Hausaufgabe / Experiment

1. Lade 24 Stunden BTC-USDT-Kerzen von Binance.  
2. Baue einen Backtest-Flow mit 1-Min-Ticks.  
3. Implementiere Fee-Handling, Slippage und Stop-Loss.  
4. Berechne Sharpe-Ratio + Max-Drawdown.  
5. Erstelle daraus einen PDF-Report mit Diagrammen.  

Optional: Führe denselben Test mit zwei Strategien aus und vergleiche die Equity-Kurven.

---

## ✅ Zusammenfassung

Nach Kapitel 17 kannst du:
- historische Marktdaten einlesen und simuliert verarbeiten,  
- Strategien reproduzierbar testen und optimieren,  
- Performance-Metriken korrekt berechnen,  
- Regressionstests durchführen,  
- und alle Ergebnisse transparent dokumentieren.  

Im nächsten Kapitel (18) geht es darum, **wie du ethische und rechtliche Grenzen definierst** – damit dein System nicht nur effizient, sondern auch verantwortungsvoll handelt.