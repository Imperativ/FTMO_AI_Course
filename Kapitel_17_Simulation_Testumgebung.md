# 📘 Kapitel 17 – Test- und Simulationsumgebungen

**Lernziel:**  
Nach dieser Lektion kannst du dein System gefahrlos testen, Strategien rückwirkend prüfen und Simulationen unter realistischen Bedingungen durchführen.

---

## 🧩 Abschnitt 1 – Warum Simulationen entscheidend sind

Kein seriöses Handelssystem geht „blind“ live.  
Backtesting ist der Windkanal deiner Strategie – ohne Risiko, aber mit realen Daten.

Ziele:
- Stabilität und Reproduzierbarkeit prüfen  
- Edge-Cases finden (z. B. Flash-Crash)  
- Debugging unter deterministischen Bedingungen  

---

## ⚙️ Abschnitt 2 – Datenquellen für Backtesting

Verwende historische Daten von:
- Binance, Oanda, Yahoo Finance, Quandl  
- APIs oder CSV-Export (`OHLCV`: Open, High, Low, Close, Volume)  

**n8n-Integration:**  
Über `Read Binary File → Function → LLM Analysis` oder direkt via REST:

```bash
GET https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1m&limit=1000
```

**Debugging-Hinweis:**  
Fehler „429 Too Many Requests“ → Pausen zwischen API-Calls (2 Sek).

---

## 💡 Abschnitt 3 – Replay-Engine in n8n

Simuliere Live-Daten mit einem Cron-Trigger + Delay:

```javascript
const candles = $json.data;
for (let i = 0; i < candles.length; i++) {
  sendToAgent(candles[i]);
  await new Promise(r => setTimeout(r, 500)); // 0.5 s zwischen Kerzen
}
```

**Debugging-Tipp:**  
Verwende separate DataStores für Simulation und Produktion, um Daten nicht zu vermischen.

---

## 🧠 Abschnitt 4 – Metriken und Auswertung

Berechne:
- Winrate  
- Profitfaktor  
- Max. Drawdown  
- Sharpe Ratio  

```javascript
const pnl = $json.trades.map(t => t.pnl);
const avg = pnl.reduce((a,b)=>a+b,0)/pnl.length;
const std = Math.sqrt(pnl.map(x=>Math.pow(x-avg,2)).reduce((a,b)=>a+b)/pnl.length);
const sharpe = avg/std;
return [{ sharpe }];
```

**Debugging-Hinweis:**  
Wenn Werte unrealistisch hoch sind (Sharpe > 5), wahrscheinlich Datenfehler oder fehlende Slippage berücksichtigt.

---

## ⚙️ Abschnitt 5 – Szenariensteuerung

Baue Simulationen für:
- Trendmärkte vs. Seitwärtsmärkte  
- Volatilitätsphasen  
- Nachrichtenereignisse  

```javascript
if ($json.volatility > 2.0) scenario = "crash_mode";
else scenario = "normal";
```

Lasse den Agenten adaptiv darauf reagieren.

---

## 🧩 Abschnitt 6 – Regressionstests für Flows

Erstelle Tests, die alte Flows mit denselben Daten erneut laufen lassen:
```bash
n8n execute --id=12 --input=./testdata/btc_2023.json
```

Vergleiche Resultate per JSON-Diff.

**Debugging-Tipp:**  
Bei Versionswechseln von n8n: alte Workflow-IDs dokumentieren, um Reproduzierbarkeit zu wahren.

---

## 🧭 Abschnitt 7 – Reflexion

- Welche Kennzahl misst den Erfolg deines Systems wirklich?  
- Wann sind Simulationen trügerisch?  
- Wie gehst du mit Overfitting um?  

---

## ✅ Zusammenfassung

Nach Kapitel 17 kannst du:
- Strategien gefahrlos testen,  
- realistische Simulationen mit historischen Daten erstellen,  
- Ergebnisse messen und vergleichen,  
- und Debugging in einer sicheren Umgebung durchführen.