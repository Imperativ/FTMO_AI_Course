# Kapitel 8 – Dashboard und Visualisierung

**Lernziel:** Du kannst Ergebnisse aus n8n visuell darstellen, z. B. mit Python oder Streamlit.

## 8.1 Warum Visualisierung
Daten werden erst nützlich, wenn sie auf einen Blick interpretierbar sind. Ein Dashboard zeigt Trends, Fehler und Risikoentwicklung.

## 8.2 Datenbasis
Beispieldatei:
```json
[{"timestamp":"2025-11-03T10:00","symbol":"BTC/USD","confidence":0.74,"success":true},
 {"timestamp":"2025-11-03T12:00","symbol":"BTC/USD","confidence":0.58,"success":false}]
```

## 8.3 Python-Renderer
```python
import pandas as pd, matplotlib.pyplot as plt
df = pd.read_json("data/ftmo_memory.json")
df["timestamp"] = pd.to_datetime(df["timestamp"])
plt.plot(df["timestamp"], df["confidence"])
plt.title("Signal-Confidence über Zeit")
plt.xlabel("Datum"); plt.ylabel("Confidence")
plt.show()
```

## 8.4 Streamlit-Alternative
```python
import streamlit as st, pandas as pd
df = pd.read_json("data/ftmo_memory.json")
st.line_chart(df, x="timestamp", y="confidence")
st.metric("Erfolgsrate", f"{df['success'].mean()*100:.1f}%")
```

## 8.5 Theorie
Visualisierung macht Fehlentwicklungen sichtbar. Sinkende Confidence-Kurve = Warnsignal.

## 8.6 Praxis
n8n → Python-Node → erzeugt Grafik → Telegram sendet PNG.

## 8.7 Aufgabe
Eigenes FTMO-Dashboard:
- Erfolgsquote über Zeit
- Durchschnittliches Risiko
- Anzahl abgelehnter Signale pro Tag

---

# Kapitel 9 – Fehlerbehandlung und Stabilität

**Lernziel:** Du fängst Fehler ab, verwaltest Logs und machst dein System robust.

## 9.1 Warum Fehlertoleranz wichtig ist
Ein Agent ohne Fehlerlogik stürzt bei API-Limits oder Netzwerkfehlern ab. Ziel: kontrolliertes Verhalten statt Absturz.

## 9.2 Logging in n8n
```javascript
try { /* API-Aufruf */ }
catch (error) { return [{ error: error.message, timestamp: new Date().toISOString() }]; }
```
Schreibe Fehler in Datei:
```bash
echo "$(date) – $error" >> logs/errors.txt
```

## 9.3 Retry-Logik
Node-Einstellungen → „Retry on Fail“. Alternativ:
```javascript
let attempt=0; let success=false;
while(attempt<3 && !success){try{/*Request*/success=true;}catch(e){attempt++;}}
```

## 9.4 Überwachung
```bash
grep -c "error" logs/errors.txt
```
Wenn >5 Fehler/Tag → Telegram-Alert.

## 9.5 Theorie
Fehler sind Information über Systemgrenzen.

## 9.6 Praxis
Erstelle „Heartbeat“-Flow (Cron jede Stunde, prüft Logs, sendet Statusbericht).

## 9.7 Aufgabe
Error-Dashboard:
- letzte 10 Fehler
- Fehlerquote letzte Woche
- Button „Reset Logs“

---

# Kapitel 10 – Backtesting und Simulation

**Lernziel:** Du nutzt historische Daten, um Regeln zu prüfen, bevor du ein Demokonto einsetzt.

## 10.1 Zweck
Backtesting prüft, ob Regeln rückwirkend Bestand haben.

## 10.2 Datenbeschaffung
Quellen: Binance Data Portal, Kaggle Datasets. Beispiel CSV:
```
timestamp,open,high,low,close
2025-10-01,67500,67620,67200,67480
```

## 10.3 Python-Simulation
```python
import pandas as pd
df = pd.read_csv("btc_hourly.csv")
signals=[]
for i in range(20,len(df)):
    if df["close"][i]>df["close"][i-10]:
        signals.append({"timestamp":df["timestamp"][i],"signal":"long"})
    else:
        signals.append({"timestamp":df["timestamp"][i],"signal":"short"})
pd.DataFrame(signals).to_json("signals.json",orient="records")
```

## 10.4 Vergleich mit LLM-Analysen
n8n-Flow: Lese Backtest-Signal → vergleiche mit LLM-Analyse → zähle Trefferquote.

## 10.5 Theorie
Simulation filtert schlechte Regeln, ersetzt aber kein Echtzeitverhalten.

## 10.6 Praxis
Erstelle backtest.py → vergleiche Regeln gegen historische Daten. Speichere JSON mit Trefferquote.

## 10.7 Aufgabe
Führe Backtest über 1 Monat durch. Berechne Gesamterfolgsquote und dokumentiere sie im Dashboard.
