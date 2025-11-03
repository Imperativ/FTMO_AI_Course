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
- können unabhängig trainiert oder aktualisiert werden.  

Beispiel aus der Praxis:
- **Analyst-Agent** → Marktinterpretation  
- **Risk-Agent** → Risikobewertung  
- **Strategie-Agent** → Positionsentscheidung  
- **Execution-Agent** → Trade-Ausführung  
- **Supervisor-Agent** → Prüft Gesamtlogik und Konsistenz  

---

## ⚙️ Abschnitt 2 – Nachrichten-Routing zwischen Agenten

Verwende in n8n JSON-Nachrichten als Kommunikationsformat:

```json
{
  "sender": "Analyst",
  "receiver": "Risk",
  "payload": {
    "trend": "bullish",
    "volatility": 1.2
  },
  "timestamp": "2025-11-06T10:00:00Z"
}
```

Ein **Router-Node** (Function) entscheidet, welcher Agent die nächste Nachricht bekommt:

```javascript
const msg = $json;
switch (msg.receiver) {
  case "Risk": return [msg];
  case "Strategy": return [msg];
  default: throw new Error("Unknown receiver");
}
```

**Debugging-Hinweis:**  
Füge jedem Datensatz ein Feld `trace_id` hinzu, um alle Schritte einer Kommunikation rückverfolgen zu können.

---

## 💡 Abschnitt 3 – Konfliktlösung zwischen Agenten

Wenn zwei Agenten unterschiedliche Empfehlungen abgeben, entscheidet der **Supervisor-Agent**.

**Beispiel:**
```json
{
  "analyst_signal": "long",
  "risk_signal": "avoid",
  "confidence": {
    "analyst": 0.8,
    "risk": 0.9
  }
}
```

**Supervisor-Funktion:**
```javascript
const s = $json;
if (s.risk_signal === "avoid" && s.risk.confidence > 0.7) {
  s.final_decision = "no_trade";
} else {
  s.final_decision = s.analyst_signal;
}
return [s];
```

**Debugging-Hinweis:**  
Logge bei jedem Konflikt sowohl beide Eingangssignale als auch die finale Entscheidung. Das erleichtert spätere Analyse der Entscheidungsqualität.

---

## 🧠 Abschnitt 4 – Kommunikation über Queue-Systeme

Bei vielen Agenten empfiehlt sich asynchrone Kommunikation über eine Queue (z. B. Redis oder RabbitMQ).

**Beispiel mit Redis:**
```bash
export N8N_EXECUTIONS_MODE=queue
export N8N_QUEUE_BULL_REDIS_HOST=localhost
```

Jeder Agent schreibt Nachrichten in eine eigene Queue, die andere abonnieren können.  
So laufen Analysen parallel, ohne sich gegenseitig zu blockieren.

**Debugging-Tipp:**  
Prüfe regelmäßig Queue-Länge (`redis-cli LLEN agent_queue`). Wenn Werte > 1000 auftreten → Bottleneck identifizieren.

---

## ⚙️ Abschnitt 5 – Hybride Entscheidungsnetze

Kombiniere LLM-basierte Module mit deterministischen Regeln.

**Struktur:**
```
[LLM-Analyst] → liefert Marktinterpretation
     ↓
[Rule-Engine] → validiert Ergebnis (technische Indikatoren)
     ↓
[LLM-Strategieberater] → evaluiert Handlungsempfehlung
     ↓
[Supervisor] → bestätigt oder verwirft
```

So profitierst du von Sprachverständnis und Logik zugleich.

**Debugging-Hinweis:**  
Logge die Eingaben der Rule-Engine getrennt – oft zeigen dort kleine Abweichungen, warum ein LLM-Signal abgelehnt wurde.

---

## 💡 Abschnitt 6 – Kommunikationsprotokoll und Zeitsteuerung

Führe ein leichtgewichtiges Protokoll ein:

| Feld | Beschreibung |
|------|---------------|
| `trace_id` | Eindeutige ID für jede Session |
| `sender` | Agentenname |
| `receiver` | Empfänger-Agent |
| `step` | Prozessabschnitt |
| `timestamp` | ISO-Zeitstempel |
| `payload` | übertragene Daten |

Zeitsteuerung über Cron oder Message-TTL (Time-To-Live) sorgt dafür, dass alte Nachrichten verworfen werden.

---

## 🧩 Abschnitt 7 – Praxis: Ein einfacher Multi-Agent-Flow

1. Erstelle in n8n vier Subflows: Analyst, Risk, Strategy, Supervisor.  
2. Verbinde sie über Router- und Merge-Nodes.  
3. Implementiere Nachrichtenformat mit `trace_id`.  
4. Lasse alle Flows parallel laufen (Queue-Mode aktivieren).  
5. Logge Entscheidungen und Dauer pro Trace.  

**Debugging-Hinweis:**  
Fehler „Workflow recursion limit reached“ → zu tiefe Subflow-Verschachtelung. Lösung: einzelne Agenten in separaten Workflows ausführen und per Webhook triggern.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Vorteile hat asynchrone Kommunikation im Vergleich zu synchronen Chains?  
- Wie stellst du sicher, dass ein einzelner Agent keine falschen Entscheidungen dominiert?  
- Könntest du deine Risk- und Strategy-Module auch auf unterschiedlichen Maschinen laufen lassen?

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Baue ein einfaches Multi-Agent-Netz mit drei Rollen (Analyst, Risk, Strategy).  
2. Implementiere JSON-Routing mit `trace_id`.  
3. Protokolliere Laufzeiten und Konfliktentscheidungen.  
4. Simuliere Fehlverhalten (z. B. Analyst liefert leeres Signal) und beobachte, wie Supervisor reagiert.  
5. Erstelle eine Zusammenfassung aller Konflikte der letzten Woche als Report.

---

## ✅ Zusammenfassung

Nach Kapitel 15 kannst du:
- mehrere spezialisierte Agenten koordinieren,  
- Kommunikationsstrukturen und Konfliktlösung implementieren,  
- hybride Entscheidungslogik mit LLMs und Regeln kombinieren,  
- und Multi-Agent-Systeme stabil und nachvollziehbar betreiben.  

Im nächsten Kapitel (16) geht es um den **Übergang zur Echtzeit-Integration**: Wie du Live-Feeds (WebSockets, REST-Streaming, Broker-APIs) nutzt, um deinen Agenten in Echtzeit handeln oder reagieren zu lassen.