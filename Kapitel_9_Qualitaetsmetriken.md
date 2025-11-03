# 📘 Kapitel 9 – Qualitätsmetriken und kontinuierliche Verbesserung

**Lernziel:**  
Nach dieser Lektion kannst du aus Logs und Feedbacks automatisch **Leistungsmetriken** generieren.  
Du lernst, wie du Trefferquote, Fehlerrate, Latenz und Tokenverbrauch berechnest und wie du daraus ableitest, wo dein System optimiert werden sollte.

---

## 🧩 Abschnitt 1 – Warum Qualitätsmetriken entscheidend sind

Ein Agent ohne Messgrößen ist wie ein Trader ohne Statistik.  
Ohne Zahlen bleibt Leistung Gefühlssache – und das führt zu Fehleinschätzungen.

Qualitätsmetriken erlauben dir, systematisch zu prüfen:
- Arbeitet der Agent konsistent?  
- Verbessern sich Entscheidungen über Zeit?  
- Lohnt sich eine Prompt- oder Parameteränderung?  
- Wie stabil ist der Workflow bei hoher Last?

Ziel: Objektive, wiederholbare Beurteilung statt subjektiver Eindrücke.

---

## ⚙️ Abschnitt 2 – Welche Metriken wichtig sind

**Primäre Metriken (Leistung):**
- `winrate` – Trefferquote (TP vs. SL)
- `expectancy` – Durchschnittsgewinn pro Trade
- `latency_ms` – durchschnittliche Laufzeit pro Run
- `error_rate` – Anteil fehlerhafter Ausführungen
- `consistency` – Varianz der Ergebnisse zwischen Tagen

**Sekundäre Metriken (Kosten & Effizienz):**
- `token_usage` – Verbrauchte Tokens pro Analyse
- `cost_per_decision` – geschätzte API-Kosten
- `retry_count` – wie oft ein Workflow wiederholt wurde

Alle Metriken werden aus bestehenden Logs (Kapitel 5–8) berechnet.

---

## 🧠 Abschnitt 3 – Berechnung von Metriken in n8n

Du kannst n8n direkt zur Auswertung verwenden.  
Ein **Data Store Query + Function Node** reicht oft schon aus.

**Function-Node Beispiel:**
```javascript
const items = $input.all();
const metrics = {
  total: items.length,
  wins: 0,
  losses: 0,
  retries: 0,
  totalTokens: 0,
  totalDuration: 0
};

for (const i of items) {
  const j = i.json;
  if (j.outcome === "TP") metrics.wins++;
  if (j.outcome === "SL") metrics.losses++;
  metrics.retries += j.retry_count || 0;
  metrics.totalTokens += j.token_usage || 0;
  metrics.totalDuration += j.latency_ms || 0;
}

metrics.winrate = metrics.total ? metrics.wins / metrics.total : 0;
metrics.avgDuration = metrics.total ? metrics.totalDuration / metrics.total : 0;
metrics.avgTokens = metrics.total ? metrics.totalTokens / metrics.total : 0;

return [metrics];
```

**Debugging-Tipp:**  
Teste jede Metrik einzeln. Lass dir mit `console.log(metrics)` Zwischenschritte anzeigen, um Rundungsfehler und leere Felder früh zu erkennen.

---

## 🧩 Abschnitt 4 – Automatische Qualitätsreports

Mit n8n kannst du regelmäßige Qualitätsauswertungen automatisch generieren lassen.  

**Beispiel-Flow:**
```
[Cron → Query Data Store]
   ↓
[Function → Metrics Calculation]
   ↓
[LLM Node → Natural-Language Summary]
   ↓
[Telegram / E-Mail Report]
```

**Beispieltext im LLM-Node:**
```
Analysiere folgende Kennzahlen:
Winrate: 0.63
Average Latency: 2.4s
Error Rate: 0.07
Gib eine kurze qualitative Bewertung und Handlungsempfehlung.
```

So bekommst du täglich einen „Agent Performance Report“ – kompakt, verständlich und aus deinen echten Daten berechnet.

---

## 💡 Abschnitt 5 – Debugging von Metriken

Fehler in Statistiken sind tückisch, weil sie selten als „Fehler“ auffallen.  
Beachte daher:

### 🔍 Tipp 1: „Check for nulls“  
In Function-Nodes immer prüfen:
```javascript
if (j.winrate == null) throw new Error("Winrate not calculated");
```

### 🔍 Tipp 2: „Cross-Validate“  
Vergleiche deine Winrate mit dem tatsächlichen Verhältnis in der Datenbank:
```sql
SELECT COUNT(*) FILTER (WHERE outcome='TP')::float / COUNT(*) FROM ftmo_signals;
```
(n8n kann SQL-Queries an externe Datenbanken ausführen.)

### 🔍 Tipp 3: „Version Tagging“  
Speichere in jeder Metrik den Prompt- oder Strategiestand:
```json
{ "winrate": 0.63, "strategy_version": "v1.3" }
```
Damit kannst du spätere Änderungen korrekt zuordnen.

---

## ⚙️ Abschnitt 6 – Visualisierung der Agentenleistung

Nutze Tools wie **Grafana**, **Metabase** oder **n8n-Dashboards**, um Metriken visuell darzustellen.  
Alternativ kannst du per Function-Node HTML-Reports erzeugen.

**Mini-Beispiel:**
```javascript
const data = $json;
return [{
  html: `
  <h3>Agenten-Performance</h3>
  <ul>
    <li>Winrate: ${(data.winrate * 100).toFixed(2)}%</li>
    <li>Durchschnittliche Dauer: ${data.avgDuration.toFixed(0)} ms</li>
    <li>Tokenverbrauch: ${data.avgTokens.toFixed(1)}</li>
  </ul>
  `
}];
```

Anschließend → **E-Mail Node** → „HTML format“ aktivieren.

---

## 🧩 Abschnitt 7 – Praxis: Automatischer Qualitätsreport

1. Erstelle in n8n einen Cron-getriggerten Workflow (z. B. täglich um 18:00 Uhr).  
2. Lies den Data Store `ftmo_signals` aus.  
3. Berechne Metriken mit Function-Node.  
4. Sende den Report als:
   - Telegram-Nachricht (Kurzfassung)
   - E-Mail mit HTML-Report
5. Logge jede Report-Ausführung mit `timestamp` und `metrics_id`.

**Debugging-Hinweis:**  
Teste Report-Flows mit kleinem Query-Limit (`limit: 5`), um Performance-Probleme und Syntaxfehler früh zu erkennen.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Metrik hat für dich den höchsten praktischen Wert?  
- Wann gilt eine Winrate als „verlässlich“ – 20 Trades oder 200?  
- Wie könntest du das Feedback aus dem Report nutzen, um Prompts oder Parameter anzupassen?  

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Erstelle deinen automatischen Qualitätsreport.  
2. Miss mindestens drei Tage lang Metriken.  
3. Prüfe, ob sich Winrate oder Laufzeit verändern.  
4. Notiere in deiner Kursmappe:  
   - mögliche Ursachen  
   - geplante Anpassungen  
   - beobachtete Verbesserungen  
5. Optional: Lasse das LLM-Modell deine Reports selbst interpretieren (Meta-Analyse).

---

## ✅ Zusammenfassung

Nach Kapitel 9 kannst du:
- Metriken automatisiert berechnen und loggen,  
- Agentenleistung objektiv bewerten,  
- Fehler in Statistiken erkennen und beheben,  
- und Reports für kontinuierliche Verbesserung nutzen.  

Im nächsten Kapitel (10) lernst du, wie du deinen gesamten Agentenprozess dokumentierst und versionierst – damit er als professionelles System nachvollziehbar, wartbar und auditierbar bleibt.