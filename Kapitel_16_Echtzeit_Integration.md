# 📘 Kapitel 16 – Echtzeit-Integration und Live-Handel mit Agenten

**Lernziel:**
Nach dieser Lektion kannst du deinen Agenten mit Live-Marktdaten versorgen, WebSocket- und Streaming-APIs korrekt anbinden, Latenz messen und handeln, ohne Synchronisationsfehler oder Deadlocks zu riskieren.

---

## 🧩 Abschnitt 1 – Warum Echtzeit-Verarbeitung kritisch ist

Automatisierte Systeme sind nur so gut wie ihre Aktualität.
Ein Signal, das 30 Sekunden alt ist, kann in volatilen Märkten wertlos oder gefährlich sein.

**Ziel:** Nicht _schneller als der Markt_ sein – sondern **rechtzeitig, korrekt und resilient**.

### Wichtige Eigenschaften von Echtzeit-Systemen

- **Kontinuität:** Eingehende Daten ohne Unterbrechung
- **Determinismus:** Gleiche Eingabe → gleiche Reaktion
- **Fehlertoleranz:** Reconnects und Retry-Strategien
- **Überwachung:** Latenz und Datenintegrität messen
- **Validierung:** Anomalie-Erkennung in Echtzeit

### Latenz-Anforderungen

- **HFT:** < 1 ms | **Day Trading:** < 100 ms | **FTMO:** < 1 s

---

## ⚙️ Abschnitt 2 – Arten von Echtzeit-Datenfeeds

### 1. WebSockets (Push-Modell)

Daten werden vom Anbieter an dich gesendet.

**Beispiel: Binance**

```
wss://stream.binance.com:9443/ws/btcusdt@trade
```

Vorteile: Low-latency | Nachteile: Reconnect-Handling nötig

### 2. REST-Streaming

Long-polling, Server sendet bei Updates neue Daten (Finnhub, AlphaVantage).

### 3. Webhooks

Externes System (z.B. TradingView) ruft deine API bei Events auf.

---

## 🧠 Abschnitt 3 – Echtzeit-Architektur im Agentensystem

### Zielarchitektur

```
[Live Feed (WebSocket/API)]
      ↓
[Data Parser → Validation]
      ↓
[Cache/Buffer (Redis)]
      ↓
[Risk Check → Strategy (LLM)]
      ↓
[Execution → Logging]
```

### n8n Flow-Struktur

```
[WebSocket Trigger]
   ↓
[Function: Parse & Validate]
   ↓
[IF: Anomaly Check]
   ├─ Pass → [LLM Decision]
   └─ Fail → [Log & Alert]
         ↓
[Function: Format Order]
   ↓
[HTTP: Broker API]
   ↓
[Set: Log Trade]
```

---

## 💡 Abschnitt 4 – Binance Live-Feed Implementation

### Function-Node: WebSocket Data Parser

```javascript
// Binance WebSocket Data Parser
const trade = $json;

// Normalisierung
const normalized = {
  symbol: trade.s || "BTCUSDT",
  price: parseFloat(trade.p),
  volume: parseFloat(trade.q),
  timestamp: new Date(trade.T).toISOString(),
  latency_ms: Date.now() - trade.T,
};

// Volume-Filter
if (normalized.volume < 0.001) return [];

return [{ json: normalized }];
```

### Spike-Detection

```javascript
// Price Spike Detection
const current = $json.price;
const lastPrice = $flow.get("lastPrice") || current;
const change = Math.abs((current - lastPrice) / lastPrice);

if (change > 0.02) {
  console.error(`Spike detected: ${(change * 100).toFixed(2)}%`);
  return []; // Verwerfe Signal
}

$flow.set("lastPrice", current);
return [$json];
```

---

## ⚙️ Abschnitt 5 – Latenz-Management und Synchronisation

### NTP-Synchronisation

```bash
# System-Zeit synchronisieren
sudo timedatectl set-ntp true

# Status prüfen
timedatectl status
```

### Latenz-Monitoring

```javascript
// Latency Monitor
const latencies = $flow.get("latencies") || [];
const currentLatency = Date.now() - new Date($json.timestamp).getTime();

latencies.push(currentLatency);
if (latencies.length > 100) latencies.shift();

$flow.set("latencies", latencies);

const avgLatency = latencies.reduce((a, b) => a + b, 0) / latencies.length;

if (avgLatency > 500) {
  throw new Error(`Latency too high: ${avgLatency.toFixed(2)}ms`);
}

return [{ json: { ...$json, avg_latency: avgLatency } }];
```

---

## 🧩 Abschnitt 6 – Reconnect-Strategie

### Exponential Backoff Implementation

```javascript
// Reconnect mit Exponential Backoff
async function reconnect(url, attempt = 0, maxRetries = 10) {
  if (attempt > maxRetries) {
    throw new Error("Max reconnect attempts reached");
  }

  try {
    console.log(`Connecting... (attempt ${attempt + 1})`);
    // WebSocket connect logic
    return true;
  } catch (error) {
    const delay = 1000 * Math.pow(2, attempt);
    console.warn(`Retry in ${delay}ms`);
    await new Promise((r) => setTimeout(r, delay));
    return reconnect(url, attempt + 1, maxRetries);
  }
}

// Verwendung
reconnect("wss://stream.binance.com:9443/ws/btcusdt@trade");
```

### Heartbeat-Monitoring

```javascript
// Heartbeat Checker
const lastTick = $flow.get("lastTick") || Date.now();
const timeSinceLastTick = Date.now() - lastTick;

if (timeSinceLastTick > 60000) {
  throw new Error("No data in 60s - reconnect required");
}

$flow.set("lastTick", Date.now());
```

---

## 🚨 Abschnitt 7 – Debug-Sektion: Echtzeit-Probleme

### Debug 1: WebSocket Disconnect

**Ursachen:** Netzwerk-Instabilität, Firewall, Rate-Limits

**Lösung:** Heartbeat alle 30s + Reconnect-Logic

### Debug 2: Hohe Latenz

**Prüfung:** `ping stream.binance.com`, `traceroute`

**Lösung:** Näher am Server hosten, CDN/Proxy entfernen

### Debug 3: Duplikate in Feed-Daten

**Lösung: Deduplizierung**

```javascript
const seen = $flow.get("seen") || new Set();
const msgId = `${$json.symbol}_${$json.timestamp}`;

if (seen.has(msgId)) return [];

seen.add(msgId);
if (seen.size > 1000) {
  const arr = Array.from(seen);
  $flow.set("seen", new Set(arr.slice(-500)));
} else {
  $flow.set("seen", seen);
}

return [$json];
```

---

## 💡 Abschnitt 8 – Paper Trading Order-Execution

```javascript
// Order Formatter
const decision = $json.decision;

if (decision === "no_trade") return [];

const order = {
  symbol: $json.symbol,
  side: decision === "long" ? "BUY" : "SELL",
  type: "MARKET",
  quantity: 0.001,
  testMode: true,
};

return [{ json: order }];
```

**HTTP Request Node:**

- Method: POST
- URL: `https://testnet.binance.vision/api/v3/order`
- Headers: `X-MBX-APIKEY: your_key`

---

## 📋 Hausaufgaben

**Aufgabe 1: WebSocket-Feed implementieren (⭐⭐)**

- Verbinde zu Binance WebSocket (BTCUSDT)
- Lasse Feed 15 Minuten laufen
- Speichere alle Trades in `live_feed.json`
- Miss durchschnittliche Latenz

**Aufgabe 2: Reconnect-System (⭐⭐⭐)**

- Implementiere Reconnect mit Exponential Backoff
- Simuliere Disconnect (WLAN aus/an)
- Logge alle Reconnect-Versuche
- Telegram-Alert bei > 3 Reconnects/Stunde

**Aufgabe 3: Spike-Detection (⭐⭐⭐)**

- Implementiere Preis-Spike-Erkennung (> 2%)
- Logge alle Spikes in separate Datei
- Erstelle Report mit Spike-Statistik
- Teste mit historischen Volatilitäts-Daten

---

## ✅ Zusammenfassung

Nach Kapitel 16 kannst du:

- Echtzeit-Feeds mit WebSocket und REST-Streaming anbinden,
- Latenz messen und optimieren,
- Reconnect-Strategien mit Exponential Backoff implementieren,
- Anomalien und Spikes in Echtzeit erkennen,
- Paper-Trading-Orders über Broker-APIs ausführen,
- und robuste Echtzeit-Workflows mit Fehlertoleranz aufbauen.

Im nächsten Kapitel lernst du, **wie du Echtzeit-Systeme testest und reproduzierbar simulierst**, um Performance und Stabilität unter Laborbedingungen zu prüfen.
