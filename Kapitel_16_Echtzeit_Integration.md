# 📘 Kapitel 16 – Echtzeit-Integration und Live-Handel

**Lernziel:**  
Nach dieser Lektion kannst du deinen Agenten mit Live-Datenquellen verbinden, WebSocket- und Streaming-APIs nutzen und Strategien in Echtzeit ausführen – stabil, überwacht und nachvollziehbar.

---

## 🧩 Abschnitt 1 – Warum Echtzeitintegration entscheidend ist

Ein Trading-Agent, der auf veraltete Daten reagiert, ist kein Agent, sondern ein Archivar.  
Echtzeit bedeutet nicht nur „schnell“, sondern **zeitnah, konsistent und reaktiv**.  
Das Ziel: Ereignisse in Millisekunden erkennen und darauf reagieren – ohne Fehlentscheidungen durch Latenz oder Paketverlust.

---

## ⚙️ Abschnitt 2 – Grundlagen von WebSocket- und Streaming-APIs

Viele Broker und Plattformen (z. B. Binance, MetaTrader, Oanda, Alpaca) bieten WebSocket-Feeds.  
Beispiel Binance-Ticker:

```javascript
wss://stream.binance.com:9443/ws/btcusdt@trade
```

Nachricht (vereinfacht):
```json
{
  "e": "trade",
  "p": "34125.50",
  "q": "0.002",
  "T": 1698756543210
}
```

**n8n-Integration:**  
Mit der Node „Webhook“ → Typ: **WebSocket Receiver** (oder REST-Polling mit kurzem Intervall).

**Debugging-Hinweis:**  
Bei Verbindungsabbrüchen „ECONNRESET“ → Retry-Loop einbauen:
```javascript
let retries = 0;
while (retries < 5) {
  try { connect(); break; }
  catch (e) { retries++; await new Promise(r => setTimeout(r, 2000)); }
}
```

---

## 💡 Abschnitt 3 – Latenzmanagement und Zeitkorrektur

Bei Live-Handel zählt jede Millisekunde.  
Führe daher Zeitsynchronisierung über NTP (Network Time Protocol) durch:

```bash
sudo timedatectl set-ntp true
```

**Taktik gegen Latenzfehler:**
- Timestamp-Vergleich zwischen Feed und Serverzeit
- maximale Verzögerung < 100 ms
- bei Abweichung → Signal verwerfen

```javascript
if (Date.now() - $json.T > 100) {
  throw new Error("Data too old, ignoring trade signal");
}
```

---

## 🧠 Abschnitt 4 – n8n Flow für Echtzeitstrategie

```
[WebSocket Trigger] → [Function → Filter + Risk Check]
     ↓
[LLM Node → Entscheidung]
     ↓
[REST Node → Broker API / Order Execution]
     ↓
[Logger + Telegram Report]
```

**Debugging-Hinweis:**  
Füge nach jedem API-Call einen kurzen Delay (200–500 ms) ein, um Rate-Limits einzuhalten.

---

## 🧩 Abschnitt 5 – API-Sicherheit und Ausfallschutz

- Alle API-Keys in `.env`
- Retry + Circuit-Breaker-System:
```javascript
let failures = 0;
try { executeTrade(); }
catch (e) {
  failures++;
  if (failures >= 3) throw new Error("Circuit open – trading paused");
}
```

**Log-Alarm:** Wenn Circuit geöffnet → Telegram-Alert.

---

## 💡 Abschnitt 6 – Monitoring von Live-Handelsflüssen

Verwende Healthchecks oder Heartbeat-Flows:
```bash
curl -fsS https://myagent/health || echo "Feed down"
```

Oder automatisiert:
```javascript
if (!$json.lastSignal || Date.now() - $json.lastSignal > 60000) {
  throw new Error("No live feed updates in 60s");
}
```

**Debugging-Tipp:**  
Wenn in Docker: Portweiterleitungen prüfen (`-p 9443:9443`) und Logs mit `docker logs -f agent` verfolgen.

---

## 🧭 Abschnitt 7 – Reflexion

- Wie gehst du mit temporären Datenlücken um?  
- Welche Maßnahmen nutzt du, um Fehlalarme zu vermeiden?  
- Wie bewertest du das Verhältnis zwischen Geschwindigkeit und Sicherheit?  

---

## ✅ Zusammenfassung

Nach Kapitel 16 kannst du:
- WebSocket- und Streaming-APIs in deinen Agent integrieren,  
- Latenz und Synchronisation kontrollieren,  
- Ausfallschutz und Echtzeitüberwachung implementieren,  
- und Live-Handel sicher simulieren oder ausführen.