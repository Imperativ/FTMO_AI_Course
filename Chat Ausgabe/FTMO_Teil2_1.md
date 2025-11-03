Verstanden und gespeichert — alle Formatierungs- und Strukturmaßnahmen werden beim Export konsequent beachtet.
Ich mache nun mit **Teil 2 (Kapitel 5–7)** weiter, vollständig im Zed-kompatiblen Markdown-Stil und inhaltlich auf demselben Niveau wie der erste Abschnitt.

---

````markdown
# Kapitel 5 – Gedächtnis und Feedback-Loop

**Lernziel:**  
Du kannst in n8n persistenten Speicher anlegen, frühere Analysen archivieren und das aktuelle LLM-Modell eigene Vorhersagen bewerten lassen. Du lernst den Unterschied zwischen „Memory“ (Zustand) und „Learning“ (Anpassung).

## 5.1  Warum ein Agent Gedächtnis braucht
Ohne Gedächtnis wiederholt der Agent dieselben Fehler.  
Memory bedeutet: frühere Analysen, Ergebnisse, Erfolg oder Misserfolg speichern, Muster erkennen und Verhalten kalibrieren.

## 5.2  Datenhaltung in n8n
### A – Datastore-Node
Speichert key-value-Paare intern, ideal für kleine Mengen (z. B. letzte 100 Analysen).  
### B – Dateibasierter Speicher
„Write Binary File“-Node → z. B. `/data/ftmo_memory.json`.

Schema:
```json
{
  "timestamp": "2025-11-03T21:00:00Z",
  "symbol": "BTC/USD",
  "analysis": {"trend": "bullish","confidence": 0.72},
  "outcome_after_4h": "neutral",
  "success": false
}
````

## 5.3  Bewertung durch LLM

Prompt:

```
Du bist ein Evaluations-Agent.
Analysiere vergangene Analysen (JSON).
Zähle Treffer, Fehleinschätzungen und bestimme Erfolgsrate.
Antworte im JSON-Format mit success_rate und Empfehlung.
```

Beispiel-Ergebnis:

```json
{"total_trades":28,"success_rate":0.64,
 "recommendation":"Nur Signale mit Confidence > 0.7 verwenden."}
```

## 5.4  Theorie

Feedback ≠ Learning. Das System wertet aus, ändert sich aber nicht selbst.
Du bleibst der Regisseur und entscheidest, welche Empfehlungen übernommen werden.

## 5.5  Praxis

Branch-Node → Pfad A: Analyse, Pfad B: Evaluation.
Pfad B läuft täglich, berechnet Trefferquote und sendet Telegram-Bericht.

## 5.6  Aufgabe

Baue einen Evaluation-Workflow, der jede Analyse speichert und wöchentlich eine Zusammenfassung sendet.
Ergebnis: Ein laufendes Performance-Monitoring deines Agenten.
--------------------------------------------------------------

# Kapitel 6 – Multi-Agent-Koordination und Rollenaufteilung

**Lernziel:**
Du kannst mehrere LLM-Instanzen mit unterschiedlichen Rollen orchestrieren – Analyst, Risikomanager, Kritiker – und deren Ergebnisse zusammenführen.

## 6.1  Motivation

Ein einzelner Agent entscheidet schnell, aber ein Team denkt besser.
Rollenprinzip:

1. Analyst – findet Signale
2. Risikomanager – prüft Regeln
3. Kritiker – bewertet beide

## 6.2  Aufbau in n8n

Parallele GPT-Nodes + Merge-Node („Wait for All“).

```
[Input]
  ├─ LLM-Analyst
  ├─ LLM-Risikomanager
  └─ LLM-Kritiker
         ↓
       Merge → Telegram
```

## 6.3  Rollen-Prompts

**Analyst**

```
Du bist Analyst. Analysiere Kursdaten, gib Signal (long/short/neutral) + confidence.
```

**Risikomanager**

```
Bewerte Analysten-Signal nach Risikoregeln. Antworte approved true/false + reason.
```

**Kritiker**

```
Vergleiche beide Antworten und gib final_decision trade/hold/skip.
```

## 6.4  Merge-Function

```javascript
const a=$items("Analyst")[0].json;
const r=$items("Risikomanager")[0].json;
const c=$items("Kritiker")[0].json;
return [{
  signal:a.signal,confidence:a.confidence,
  approved:r.approved,
  decision:c.final_decision,
  summary:`Analyst:${a.signal} (${a.confidence}) Risk:${r.approved} Final:${c.final_decision}`
}];
```

## 6.5  Theorie

Mehrere Perspektiven erhöhen Zuverlässigkeit.
Unterschiedliche Prompts erzeugen unterschiedliche Denkstile – das bringt robustere Ergebnisse.

## 6.6  Praxis

Erstelle den Dreifach-Flow, teste mit historischen Daten, prüfe ob „skip“-Entscheidungen auftreten.

## 6.7  Aufgabe

Füge vierten Agenten „Statistik“ hinzu und implementiere ein Mehrheitsvotum:

```javascript
const votes=[analyst.signal,critic.final_decision,stats.suggestion];
const decision=votes.filter(v=>v==="long").length>1?"long":"short";
return[{final_vote:decision}];
```

---

# Kapitel 7 – Ablauf-Orchestrierung und Zeitsteuerung

**Lernziel:**
Du kannst Workflows automatisch zu bestimmten Zeiten oder Ereignissen ausführen und deinen Agenten über Tage und Wochen steuern.

## 7.1  Bedeutung der Zeit

Analyse, Risikoprüfung und Evaluation benötigen Rhythmus: z. B. stündlich, täglich, wöchentlich.
So wird das System vorhersehbar und auswertbar.

## 7.2  Zeitsteuerung in n8n

* Cron-Trigger → regelmäßig
* Webhook → Ereignisgesteuert
* Interval/Schedule → Live-Zyklen

Beispiel: Cron alle 2 h → Analyse-Workflow → Telegram.

## 7.3  Tages- und Wochenzyklen

| Workflow   | Zeitplan  | Zweck          |
| ---------- | --------- | -------------- |
| Analyse    | stündlich | Kurzanalyse    |
| Risk       | 18:00     | Tagesabschluss |
| Memory     | 23:00     | Log            |
| Evaluation | So 20:00  | Wochenreview   |

## 7.4  Ereignissteuerung

TradingView → Webhook → n8n:
Payload:

```json
{"symbol":"EURUSD","rsi":28,"timeframe":"1h"}
```

LLM bewertet und meldet sofort Ergebnis.

## 7.5  Theorie

Zeit strukturiert Information. Gleichmäßige Intervalle verhindern Rauschen und machen Statistik möglich.

## 7.6  Praxis

Drei Workflows:

* `Analysis_Day` (alle 2 h)
* `Risk_Report` (18:00)
* `Performance_Log` (23:00)
  Gemeinsame Log-Datei + Tages-Telegram-Bulletin.

## 7.7  Aufgabe

Baue Wochencontroller:

* Sonntag: Wochenausblick
* Freitag: Abschlussanalyse
  Lass LLM nach mehreren Wochen bewerten, an welchen Tagen höchste Trefferquote lag.

---

```

---

Das war **Teil 2 der Kursmappe (Kapitel 5–7)**, formatiert und inhaltlich konsistent mit Teil 1.  
Wenn du bestätigst, dass Aufbau und Stil weiterhin stimmen, erstelle ich **Teil 3 (Kapitel 8–10)** im gleichen Format.  
Möchtest du, dass ich fortfahre?
```
