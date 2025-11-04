# 📘 Kapitel 15 – Multi-Agent-Kollaboration und hybride Entscheidungsnetze

**Lernziel:**
Nach dieser Lektion kannst du ein System entwerfen, in dem mehrere spezialisierte Agenten zusammenarbeiten, Informationen austauschen, Konflikte lösen und gemeinsam Entscheidungen treffen – nachvollziehbar, sicher und skalierbar.

---

## 🧩 Abschnitt 1 – Warum Kooperation besser ist als Zentralisierung

Ein einziger Super-Agent klingt mächtig, ist aber anfällig:

- Fehler eines Moduls beeinflussen alle anderen.
- Komplexe Aufgaben überfordern einzelne Kontexte.
- Debugging wird schwierig, weil Zustände vermischt werden.

Mehrere **spezialisierte Agenten** dagegen:

- teilen Aufgaben klar auf,
- liefern robuste, modulare Ergebnisse,
- können unabhängig trainiert oder aktualisiert werden,
- ermöglichen parallele Verarbeitung und Skalierung.

### Beispiel aus der Praxis

**Agent-Rollen im FTMO-System:**

- **Analyst-Agent** → Marktinterpretation und Trend-Erkennung
- **Risk-Agent** → Risikobewertung und Portfolio-Protection
- **Strategie-Agent** → Positionsentscheidung und Entry/Exit-Timing
- **Execution-Agent** → Trade-Ausführung und Order-Management
- **Supervisor-Agent** → Prüft Gesamtlogik und Konsistenz

### Vorteile der Spezialisierung

- **Modulares Testing:** Jeder Agent kann isoliert getestet werden
- **Unabhängige Updates:** Risk-Logik ändern ohne Analyst zu beeinflussen
- **Fehler-Isolation:** Problem im Analyst betrifft nicht die Execution
- **Klare Verantwortlichkeiten:** Debugging wird einfacher

---

## ⚙️ Abschnitt 2 – Nachrichten-Routing zwischen Agenten

Verwende in n8n JSON-Nachrichten als Kommunikationsformat:

```json
{
  "trace_id": "trade_20250106_001",
  "sender": "Analyst",
  "receiver": "Risk",
  "priority": "high",
  "payload": {
    "symbol": "EURUSD",
    "trend": "bullish",
    "volatility": 1.2,
    "confidence": 0.85,
    "indicators": {
      "rsi": 65,
      "macd": "positive_crossover"
    }
  },
  "timestamp": "2025-11-06T10:00:00Z"
}
```

### Router-Node Implementation

**Function-Node: Message-Router**

```javascript
// Multi-Agent Message Router
const msg = $json;

// Validierung
if (!msg.receiver || !msg.sender || !msg.trace_id) {
  throw new Error("Invalid message format - missing required fields");
}

// Logging für Nachverfolgung
console.log(`[${msg.trace_id}] Routing: ${msg.sender} → ${msg.receiver}`);

// Route basierend auf Empfänger
switch (msg.receiver) {
  case "Risk":
    return [{ json: msg, destination: 0 }];
  case "Strategy":
    return [{ json: msg, destination: 1 }];
  case "Execution":
    return [{ json: msg, destination: 2 }];
  case "Supervisor":
    return [{ json: msg, destination: 3 }];
  default:
    throw new Error(`Unknown receiver: ${msg.receiver}`);
}
```

**Debugging-Hinweis:**
Füge jedem Datensatz ein Feld `trace_id` hinzu, um alle Schritte einer Kommunikation rückverfolgen zu können. Speichere alle Nachrichten in einem `message_log.json` für Audit-Zwecke.

---

## 💡 Abschnitt 3 – Konfliktlösung zwischen Agenten

Wenn zwei Agenten unterschiedliche Empfehlungen abgeben, entscheidet der **Supervisor-Agent**.

### Conflict-Resolution-Strategien

1. **Confidence-basiert:** Agent mit höherem Confidence-Score gewinnt
2. **Priority-basiert:** Risk-Agent hat immer Veto-Recht
3. **Consensus-basiert:** Mehrheitsentscheidung bei 3+ Agenten

**Beispiel:**

```json
{
  "analyst_signal": "long",
  "risk_signal": "avoid",
  "confidence": { "analyst": 0.8, "risk": 0.9 }
}
```

**Supervisor-Funktion:**

```javascript
const s = $json;
// Risk-Agent hat Veto-Recht bei hoher Confidence
if (s.risk_signal === "avoid" && s.confidence.risk > 0.7) {
  s.final_decision = "no_trade";
  s.reason = "Risk veto";
} else {
  s.final_decision = s.analyst_signal;
  s.reason = "Analyst recommendation";
}
return [s];
```

**Debugging-Hinweis:**
Logge bei jedem Konflikt sowohl alle Eingangssignale als auch die finale Entscheidung mit Begründung. Das erleichtert spätere Analyse der Entscheidungsqualität und ermöglicht Pattern-Erkennung.

---

## 🧠 Abschnitt 4 – Kommunikation über Queue-Systeme

Bei vielen Agenten empfiehlt sich asynchrone Kommunikation über eine Queue (z. B. Redis oder RabbitMQ).

### Redis Queue Setup

**Beispiel mit Redis:**

```bash
# n8n Queue-Mode aktivieren
export N8N_EXECUTIONS_MODE=queue
export N8N_QUEUE_BULL_REDIS_HOST=localhost
export N8N_QUEUE_BULL_REDIS_PORT=6379
export N8N_QUEUE_BULL_REDIS_DB=0
```

### Vorteile von Queue-basierter Kommunikation

- **Asynchrone Verarbeitung:** Agenten blockieren sich nicht gegenseitig
- **Load Balancing:** Mehrere Instanzen eines Agenten möglich
- **Retry-Mechanismen:** Fehlgeschlagene Nachrichten können wiederholt werden
- **Skalierbarkeit:** Einfaches Hinzufügen weiterer Agent-Instanzen

**Debugging-Tipp:**
Prüfe regelmäßig Queue-Länge und Processing-Rate:

```bash
# Queue-Länge prüfen
redis-cli LLEN agent_queue_analyst
redis-cli LLEN agent_queue_risk

# Queue-Stats
redis-cli INFO stats
```

Wenn Werte > 1000 auftreten → Bottleneck identifizieren:

- Zu langsame Verarbeitung?
- Zu viele eingehende Nachrichten?
- Agent-Instanz abgestürzt?

---

## ⚙️ Abschnitt 5 – Hybride Entscheidungsnetze

Kombiniere LLM-basierte Module mit deterministischen Regeln.

**Struktur:**

```
[LLM-Analyst] → Marktinterpretation
     ↓
[Rule-Engine] → Validierung (technische Indikatoren)
     ↓
[Supervisor] → finale Entscheidung
```

**Rule-Engine-Beispiel:**

```javascript
const llm = $json.llm_output;
const tech = $json.indicators;

// Validiere LLM-Signal mit technischen Daten
const valid =
  llm.trend === "bullish" &&
  tech.ema_20 > tech.ema_50 &&
  tech.atr < 0.015 &&
  tech.volume > tech.volume_avg * 0.8;

return [{ json: { ...llm, validation_passed: valid } }];
```

**Debugging-Hinweis:**
Logge die Eingaben der Rule-Engine getrennt – oft zeigen dort kleine Abweichungen, warum ein LLM-Signal abgelehnt wurde. Erstelle einen Validation-Report mit allen geprüften Regeln.

---

## 💡 Abschnitt 6 – Kommunikationsprotokoll und Zeitsteuerung

Führe ein strukturiertes Kommunikationsprotokoll ein für vollständige Nachvollziehbarkeit.

### Protokoll-Schema

| Feld        | Typ      | Beschreibung                           |
| ----------- | -------- | -------------------------------------- |
| `trace_id`  | String   | Eindeutige ID für jede Trading-Session |
| `sender`    | String   | Agentenname des Absenders              |
| `receiver`  | String   | Empfänger-Agent                        |
| `step`      | Integer  | Prozessschritt-Nummer                  |
| `timestamp` | ISO-8601 | Zeitstempel der Nachricht              |
| `payload`   | Object   | Übertragene Daten                      |
| `priority`  | String   | high / medium / low                    |
| `ttl`       | Integer  | Time-To-Live in Sekunden               |

**TTL-Prüfung:**

```javascript
const msg = $json;
const ageSeconds = (new Date() - new Date(msg.timestamp)) / 1000;

if (msg.ttl && ageSeconds > msg.ttl) {
  throw new Error("Message TTL exceeded");
}
return [msg];
```

---

## 🚨 Abschnitt 7 – Debug-Hinweise

**Problem 1: Nachrichten kommen nicht an**

- Prüfe Router-Logic und `receiver`-Feld
- Bei Redis: `redis-cli LLEN agent_queue_risk`
- Ist empfangender Workflow aktiv?

**Problem 2: Endlos-Schleifen**

```javascript
if (!msg.hop_count) msg.hop_count = 0;
msg.hop_count++;
if (msg.hop_count > 10) {
  throw new Error("Loop detected");
}
```

**Problem 3: Inkonsistente Konflikt-Resolution**

- Logge alle Entscheidungen mit Inputs
- Prüfe Confidence-Werte auf Stabilität

---

## 🧩 Abschnitt 8 – Praxis: Multi-Agent-Flow

**Aufbau:**

1. Erstelle 4 separate Workflows (Analyst, Risk, Strategy, Supervisor)
2. Verbinde über Webhooks oder Router-Nodes
3. Implementiere `trace_id` in allen Nachrichten
4. Aktiviere Queue-Mode für parallele Verarbeitung
5. Logge alle Entscheidungen und Konflikte

---

## 🧭 Abschnitt 9 – Reflexion

- Welche Vorteile hat asynchrone Kommunikation?
- Wie verhinderst du, dass ein Agent alle Entscheidungen dominiert?
- Könnten Agenten auf verschiedenen Maschinen laufen?

---

## 📋 Hausaufgaben

**Aufgabe 1: Multi-Agent-System (⭐⭐)**

- Baue System mit 3 Agenten (Analyst, Risk, Strategy)
- Implementiere JSON-Routing mit `trace_id`
- Teste mindestens ein Konflikt-Szenario

**Aufgabe 2: Conflict-Resolution (⭐⭐⭐)**

- Implementiere Supervisor mit 2 Resolution-Strategien
- Dokumentiere Konflikte in `conflicts.json`
- Erstelle wöchentlichen Report

**Aufgabe 3: Queue-Integration (⭐⭐⭐)**

- Migriere auf Redis-Queues
- Monitoring implementieren
- Load-Testing mit 100+ Messages

---

## ✅ Zusammenfassung

Nach Kapitel 15 kannst du:

- mehrere spezialisierte Agenten koordinieren,
- Kommunikationsstrukturen und Konfliktlösung implementieren,
- hybride Entscheidungslogik mit LLMs und Regeln kombinieren,
- Queue-basierte asynchrone Kommunikation nutzen,
- und Multi-Agent-Systeme stabil und nachvollziehbar betreiben.

Im nächsten Kapitel (16) geht es um **Echtzeit-Integration**: Wie du Live-Feeds (WebSockets, REST-Streaming, Broker-APIs) nutzt, um deinen Agenten in Echtzeit handeln zu lassen.
