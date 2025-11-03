# 📘 Kapitel 16 – Echtzeit-Integration und Live-Handel mit Agenten

**Lernziel:**  
Nach dieser Lektion kannst du deinen Agenten mit Live-Marktdaten versorgen, WebSocket- und Streaming-APIs korrekt anbinden, Latenz messen und handeln, ohne Synchronisationsfehler oder Deadlocks zu riskieren.  
Du lernst, wie man asynchrone Datenströme verarbeitet, Feeds überwacht, Signale validiert und mit Fehlertoleranz live reagiert.

---

## 🧩 Abschnitt 1 – Warum Echtzeit-Verarbeitung kritisch ist

Automatisierte Systeme sind nur so gut wie ihre Aktualität.  
Ein Signal, das 30 Sekunden alt ist, kann in volatilen Märkten wertlos oder gefährlich sein.  
Ziel ist nicht, *schneller als der Markt* zu sein – sondern **rechtzeitig, korrekt und resilient**.

Wichtige Eigenschaften von Echtzeit-Systemen:
- **Kontinuität:** Eingehende Daten ohne Unterbrechung.  
- **Determinismus:** Gleiche Eingabe → gleiche Reaktion.  
- **Fehlertoleranz:** Reconnects und Retry-Strategien.  
- **Überwachung:** Latenz und Datenintegrität messen.

---

## ⚙️ Abschnitt 2 – Arten von Echtzeit-Datenfeeds

### 1. **WebSockets (Push-Modell)**
Daten werden vom Anbieter an dich gesendet. Beispiel: Binance, Oanda, Alpaca.

```javascript
wss://stream.binance.com:9443/ws/btcusdt@trade
```

Nachricht:
```json
{
  "e": "trade",
  "p": "34215.22",
  "q": "0.004",
  "T": 1730803200000
}
```

### 2. **Streaming-REST (Long Polling)**
Client hält Verbindung offen, Server sendet bei Änderung neue Zeilen (z. B. Finnhub, AlphaVantage).

### 3. **Event-Webhooks (Server-seitige Pushs)**
Externes System ruft deine API auf, wenn Ereignis eintritt – ideal für TradingView-Signale.

---

## 🧠 Abschnitt 3 – Echtzeit-Architektur im Agentensystem

**Zielarchitektur:**

```
[Live Feed (WebSocket / TradingView / API)]  
      ↓  
[Data Parser → Validation → Cache]  
      ↓  
[Risk Check → Strategy Decision (LLM)]  
      ↓  
[Execution → Report → Logging / Telegram]
```

In n8n läuft das als kombinierter Flow:
- **Trigger-Node:** WebSocket / Webhook / Cron  
- **Parser-Node:** wandelt Daten in standardisiertes JSON  
- **Decision-Node:** LLM-Analyse oder Regelprüfung  
- **Execution-Node:** REST-Call an Broker-API  
- **Logger-Node:** Datenbank, CSV oder Telegram-Report

---

## 💡 Abschnitt 4 – Beispiel: Binance Live-Feed anbinden

1. In n8n: Node „HTTP Request“ → Methode: **WebSocket**  
   URL:  
   ```
   wss://stream.binance.com:9443/ws/btcusdt@trade
   ```

2. Function-Node zur Verarbeitung:
```javascript
const t = $json;
return [{
  symbol: "BTCUSDT",
  price: parseFloat(t.p),
  volume: parseFloat(t.q),
  timestamp: new Date(t.T).toISOString()
}];
```

3. Filter-Node:
```javascript
if ($json.volume < 0.001) return []; // ignoriere Mikro-Trades
return [$json];
```

4. Weiterleitung an den Entscheidungs-Agenten (LLM-Node oder eigenes Modul).

---

## ⚙️ Abschnitt 5 – Latenz-Management und Synchronisation

Echtzeit-Systeme brauchen präzise Zeitsteuerung.

### NTP-Synchronisation:
```bash
sudo timedatectl set-ntp true
```

### Zeitabweichung prüfen:
```javascript
const delay = Date.now() - $json.T;
if (delay > 200) throw new Error("Data delay too high");
```

**Debugging-Hinweis:**  
Wenn Latenz > 1 s → Ursache meist Netzjitter oder zu viele parallele Verbindungen.  
Nutze `ping -c 5 api.binance.com` um RTT zu messen.

---

## 🧩 Abschnitt 6 – Signal-Validierung und Fehlerbehandlung

Jedes Live-Signal muss geprüft werden:
- Zeitfenster < 100 ms  
- Wert im erwarteten Bereich  
- kein Duplikat  

Beispiel:
```javascript
if (Math.abs($json.price - $flow.get('lastPrice', 0)) > 5000) {
  throw new Error("Spike detected – ignoring anomaly");
}
$flow.set('lastPrice', $json.price);
```

**Debugging-Tipp:**  
Halte Anomalien getrennt im Log, um Feeds zu bewerten.

---

## ⚙️ Abschnitt 7 – Live-Handels-Execution (Simuliert)

Viele Broker (Oanda, Alpaca, Binance) erlauben Demo- oder Paper-Trading.

Beispiel-Flow:
```
[LLM Node] → erstellt JSON-Entscheidung  
   ↓
[Function → Order Formatierung]  
   ↓
[HTTP Node → Broker API]
```

**Order-Beispiel:**
```json
{
  "symbol": "BTCUSDT",
  "side": "BUY",
  "type": "MARKET",
  "quantity": 0.001
}
```

**Debugging-Hinweis:**  
Immer `testnet`-Umgebung nutzen (z. B. `api-testnet.binance.vision`).  
Fehler „INVALID_SYMBOL“ → falsche Schreibweise oder Testnetz ohne Symbolunterstützung.

---

## 🧩 Abschnitt 8 – Reconnects, Timeouts und Heartbeats

Live-Feeds reißen regelmäßig ab. Implementiere einen Heartbeat-Checker:

```javascript
const last = $flow.get('lastTick', 0);
if (Date.now() - last > 60000) {
  throw new Error("No tick in 60s – reconnecting");
}
$flow.set('lastTick', Date.now());
```

**Fehlerbild:**  
`ECONNRESET`, `1006: connection closed`  
→ Lösung: reconnect-Loop mit exponential backoff.  

```javascript
for (let i=0; i<5; i++){
  try { connect(); break; }
  catch(e){ await new Promise(r=>setTimeout(r, i*2000)); }
}
```

---

## 💡 Abschnitt 9 – Live-Monitoring und Alarmierung

Nutze `Error Workflow` in n8n:  
Bei Ausfall → Telegram oder Mail senden:

```javascript
if ($json.status === "disconnected") {
  sendTelegram("Feed offline – reconnecting...");
}
```

Oder extern über **Uptime Kuma**:
```
curl -fsS https://uptime.kuma/api/push/myfeed?status=up
```

**Debugging-Hinweis:**  
Hohe CPU-Last → WebSocket-Node in eigenem Container auslagern.

---

## ⚙️ Abschnitt 10 – Logging, Replay und Audit

Alle Live-Events in Ring-Buffer schreiben:

```javascript
const fs = require('fs');
fs.appendFileSync('./logs/live_feed.log', JSON.stringify($json)+"\n");
```

Für Post-Mortem-Analysen:
```bash
grep "BTCUSDT" logs/live_feed.log | tail -n 100
```

**Debugging-Tipp:**  
Regelmäßige Log-Rotation, sonst läuft Container-Speicher voll.

---

## 🧭 Abschnitt 11 – Reflexion

- Wie balancierst du Geschwindigkeit und Sicherheit?  
- Welche Quellen würdest du redundant anbinden (z. B. Binance + Oanda)?  
- Wie könntest du fehlerhafte Signale automatisch validieren, ohne echte Trades zu verlieren?

---

## 🧩 Abschnitt 12 – Hausaufgabe / Experiment

1. Erstelle einen n8n-Flow mit Binance WebSocket.  
2. Lasse ihn 15 Minuten laufen, speichere alle Ticks.  
3. Implementiere Spike-Filterung + Reconnect-System.  
4. Füge Telegram-Benachrichtigung bei Disconnect ein.  
5. Miss Latenz und logge Durchschnittswerte.  

Optional: Simuliere Market-Orders im Testnet und protokolliere alle Order-Antwortzeiten.

---

## ✅ Zusammenfassung

Nach Kapitel 16 kannst du:
- Echtzeit-Feeds mit WebSocket, REST-Stream oder Webhooks anbinden,  
- Latenz, Zeitabweichungen und Feed-Integrität messen,  
- Signale prüfen, reconnecten und debuggen,  
- Trades simuliert oder live ausführen,  
- und robuste Echtzeit-Workflows in n8n aufbauen.  

Im nächsten Kapitel wirst du lernen, **wie du diese Echtzeit-Systeme testest und reproduzierbar simulierst**, um Performance und Stabilität unter Laborbedingungen zu prüfen.