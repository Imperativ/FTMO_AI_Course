# Kapitel 5 – Gedächtnis und Feedback‑Loop

**Lernziel:** Du kannst persistenten Speicher in n8n anlegen, frühere Analysen archivieren und das aktuelle LLM‑Modell eigene Vorhersagen bewerten lassen. Du verstehst den Unterschied zwischen „Memory“ (Zustand) und „Learning“ (Anpassung).

## 5.1 Warum ein Agent Gedächtnis braucht
Ohne Gedächtnis wiederholt der Agent dieselben Fehler. Memory hält Analysen, Ergebnisse und Entscheidungen fest, um Muster zu erkennen und Verhalten zu kalibrieren.

## 5.2 Datenhaltung in n8n
A) Datastore‑Node (intern) – key‑value‑Speicher für kleine Mengen.  
B) Dateibasierter Speicher – „Write Binary File“ → z. B. `data/ftmo_memory.json`.

Schema:
```json
{
  "timestamp": "2025-11-03T21:00:00Z",
  "symbol": "BTC/USD",
  "analysis": {"trend": "bullish", "confidence": 0.72},
  "outcome_after_4h": "neutral",
  "success": false
}
```

## 5.3 Bewertung durch LLM
Prompt:
```
Du bist Evaluations-Agent.
Analysiere die Liste vergangener Analysen (JSON).
Zähle Treffer/Fehler, ermittle success_rate und gib eine Empfehlung.
Antwort im JSON-Format.
```

Beispiel‑Ergebnis:
```json
{"total_trades":28,"success_rate":0.64,"most_reliable_confidence_range":"0.6–0.75",
 "recommendation":"Nur Signale mit Confidence > 0.7 berücksichtigen."}
```

## 5.4 Theorie
Feedback ist nicht automatisch Lernen. Du entscheidest, welche Empfehlungen in die Regeln einfließen.

## 5.5 Praxis
Branch‑Node: Pfad A Analyse, Pfad B Evaluation (täglich). Telegram‑Bericht mit success_rate und Empfehlung.

## 5.6 Aufgabe
Erstelle einen Evaluation‑Workflow, der jede Analyse speichert und wöchentlich eine Zusammenfassung per Telegram versendet.

---

# Kapitel 6 – Multi‑Agent‑Koordination und Rollenaufteilung

**Lernziel:** Du orchestrierst mehrere LLM‑Instanzen mit Rollen (Analyst, Risikomanager, Kritiker) und führst ihre Antworten zusammen.

## 6.1 Motivation
Mehrere Perspektiven reduzieren Fehlurteile. Rollen: Analyst findet Signale, Risikomanager prüft Regeln, Kritiker moderiert das Urteil.

## 6.2 Aufbau in n8n
Parallele LLM‑Nodes und Merge‑Node („Wait for All“).
```
[Input]
  ├─ LLM-Analyst
  ├─ LLM-Risikomanager
  └─ LLM-Kritiker
         ↓
       Merge → Telegram
```

## 6.3 Rollen‑Prompts
Analyst:
```
Analysiere Kursdaten. Gib signal (long/short/neutral) und confidence zurück (JSON).
```
Risikomanager:
```
Bewerte das Signal anhand von Regeln (Kontogröße, Stop-Loss, Drawdown). Antworte mit approved, reason, max_position_size (JSON).
```
Kritiker:
```
Vergleiche die Antworten. Erkläre Widersprüche. Gib final_decision (trade/hold/skip) und rationale (JSON).
```

## 6.4 Merge‑Function
```javascript
const a = $items("Analyst")[0].json;
const r = $items("Risikomanager")[0].json;
const c = $items("Kritiker")[0].json;
return [{
  signal: a.signal,
  confidence: a.confidence,
  approved: r.approved,
  final_decision: c.final_decision,
  rationale: c.rationale,
  summary: `Analyst:${a.signal} (${a.confidence}) Risk:${r.approved ? 'ok' : 'abgelehnt'} Final:${c.final_decision}`
}];
```

## 6.5 Theorie
Unterschiedliche Rollenprompts erzeugen unterschiedliche Denkstile und erhöhen Robustheit.

## 6.6 Praxis
Erstelle den Dreifach‑Flow, teste mit historischen Daten und prüfe „skip“-Empfehlungen.

## 6.7 Aufgabe
Füge einen Statistik‑Agenten hinzu und implementiere ein Mehrheitsvotum:
```javascript
const votes = [analyst.signal, critic.final_decision, stats.suggestion];
const longVotes = votes.filter(v => v === "long").length;
const shortVotes = votes.filter(v => v === "short").length;
const decision = longVotes > shortVotes ? "long" : "short";
return [{ final_vote: decision }];
```

---

# Kapitel 7 – Ablauf‑Orchestrierung und Zeitsteuerung

**Lernziel:** Du startest Workflows zeit‑ oder ereignisgesteuert und steuerst deinen Agenten über Tage und Wochen.

## 7.1 Bedeutung der Zeit
Rhythmus macht das System auswertbar: stündliche Analysen, tägliche Risikoprüfung, wöchentliche Reviews.

## 7.2 Zeitsteuerung in n8n
Cron‑Trigger für regelmäßige Läufe; Webhook für Ereignisse; Interval/Schedule für Live‑Monitore. Beispiel: Cron alle 2 Stunden → Analyse‑Workflow → Telegram.

## 7.3 Tages‑ und Wochenzyklen
| Workflow        | Zeitplan | Zweck             |
|-----------------|----------|-------------------|
| Analyse-Agent   | stündlich| Kurzanalyse       |
| Risk-Agent      | 18:00    | Tagesabschluss    |
| Memory-Agent    | 23:00    | Log schreiben     |
| Evaluations‑Agent| So 20:00| Wochenreview      |

## 7.4 Ereignissteuerung
TradingView → Webhook → n8n. Beispiel‑Payload:
```json
{"symbol":"EURUSD","rsi":28,"timeframe":"1h"}
```

## 7.5 Theorie
Zeit strukturiert Information. Gleichmäßige Intervalle verhindern Rauschen und ermöglichen Statistik.

## 7.6 Praxis
Drei Workflows: Analysis_Day (alle 2 h), Risk_Report (18:00), Performance_Log (23:00). Gemeinsame Log‑Datei und Tages‑Bulletin per Telegram.

## 7.7 Aufgabe
Baue einen Wochencontroller: Sonntag Wochenausblick, Freitag Abschlussanalyse. Lasse nach mehreren Wochen die besten Wochentage ermitteln und passe Cron‑Zeiten an.
