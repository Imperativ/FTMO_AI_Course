# 📘 Kapitel 14 – Adaptive Strategien und kontinuierliches Lernen

**Lernziel:**  
Nach dieser Lektion kannst du deinen Agenten so erweitern, dass er aus seinen eigenen Ergebnissen lernt, Strategien dynamisch anpasst und damit seine Performance im Zeitverlauf verbessert – unter voller Kontrolle und Nachvollziehbarkeit.

---

## 🧩 Abschnitt 1 – Was bedeutet „Adaptivität“ im Agentensystem?

Ein adaptives System erkennt Muster in den eigenen Erfolgen und Fehlschlägen und nutzt diese Information zur Selbstoptimierung.  
Das Ziel ist **kontrollierte Selbstanpassung**, nicht autonomes Umschreiben von Regeln.

In deinem FTMO-Agenten bedeutet das:
- Lernen aus historischen Trades und Evaluationslogs  
- dynamische Anpassung von Parametern wie Risiko, Zeithorizont, oder Entscheidungsschwellen  
- Erprobung kleiner Veränderungen mit klarer Dokumentation („experimentelle Branches“)

---

## ⚙️ Abschnitt 2 – Feedback-Loop aus Logs und Metriken

Dein System produziert bereits reichlich Daten: Winrate, Drawdown, Latenz, Tokenverbrauch.  
Diese Daten sind der Rohstoff für Lernen.

Ein typischer Feedback-Loop:
```
[Logs + Reports]
   ↓
[Evaluation Flow (aus Kap. 12)]
   ↓
[LLM Node → Musteranalyse]
   ↓
[Decision Node → Strategie-Update]
   ↓
[Commit & Versioning]
```

**Function-Node Beispiel für Parameter-Anpassung:**
```javascript
const perf = $json;
let riskMultiplier = 1.0;

if (perf.winrate > 0.65 && perf.profitFactor > 1.5) {
  riskMultiplier = 1.2; // vorsichtig erhöhen
} else if (perf.winrate < 0.5) {
  riskMultiplier = 0.8; // reduzieren
}

return [{ ...perf, newRiskMultiplier: riskMultiplier }];
```

**Debugging-Hinweis:**  
Achte darauf, dass Anpassungen erst nach ausreichender Datenmenge (> 30 Trades) erfolgen, um statistisches Rauschen zu vermeiden.

---

## 🧠 Abschnitt 3 – Experimentelles Lernen mit Versions-Branches

Erstelle für jede größere Anpassung einen neuen Branch:
```bash
git checkout -b experiment_risk_v2
```

Darin testest du alternative Parameter oder Prompts.  
Wenn die Ergebnisse stabil besser sind → Merge in Hauptzweig.  

**Debugging-Tipp:**  
Nutze in n8n ein Feld `experiment_tag` (z. B. „risk_v2“) in deinen Logs. So kannst du in der Evaluation zwischen Varianten unterscheiden.

---

## 💡 Abschnitt 4 – Selbstüberprüfung durch LLM-Analysen

Setze ein LLM-Node ein, das historische Logs analysiert und qualitative Trends beschreibt:

**Prompt:**
```
Analysiere die letzten 100 Trades.
Bewerte die Konsistenz der Entscheidungslogik und erkenne wiederkehrende Muster bei Fehlschlägen.
Gib Vorschläge für Parameteranpassungen im JSON-Format:
{"param":"risk_level","suggested_change":-0.1,"reason":"zu hohe Verlustserie bei hoher Volatilität"}
```

**Debugging-Hinweis:**  
Beschränke die Analyse auf objektive Metriken; vermeide unkontrollierte Veränderungen, indem du jeden Vorschlag manuell bestätigst oder in einem Review-Flow prüfst.

---

## ⚙️ Abschnitt 5 – Adaptive Regeln in n8n

Implementiere adaptive Regeln mit Switch- oder IF-Nodes, basierend auf dynamischen Feldern:

```javascript
if ($json.volatility > 1.8 && $json.sentiment < 0.3) {
  $json.position = "avoid";
} else if ($json.trend === "bullish" && $json.risk < 0.7) {
  $json.position = "long";
} else {
  $json.position = "neutral";
}
return [$json];
```

Diese Regeln können wöchentlich durch den Evaluations-Flow aktualisiert werden.

---

## 🧩 Abschnitt 6 – Langzeit-Monitoring und Drift-Erkennung

Über Zeit können sich Marktbedingungen ändern – dein Agent muss erkennen, wann alte Strategien nicht mehr passen.

### Erkenne Performance-Drift:
```javascript
const recent = $json.last30Days.winrate;
const previous = $json.last90Days.winrate;
if (recent < previous * 0.8) {
  throw new Error("Performance Drift detected – Recalibration required");
}
```

Füge diese Überwachung in den täglichen Cron-Report ein.

**Debugging-Hinweis:**  
Achte auf fehlerhafte Zeitfenster: Wenn die Logs Lücken enthalten, Drift-Vergleiche anhalten und Lücken im Data-Store schließen.

---

## 💡 Abschnitt 7 – Automatische Dokumentation von Lernschritten

Erstelle bei jeder Anpassung einen Eintrag in `LEARNING_LOG.md`:

```
### [2025-11-05]
- Grundlage: Evaluation vom 03.11.–04.11.
- Änderung: riskMultiplier +0.1
- Ergebnis nach 20 Trades: Winrate +4 %, Drawdown −2 %
- Bewertung: erfolgreich, in Version 1.4 übernommen
```

**Debugging-Hinweis:**  
Automatisiere Eintragserstellung via Function-Node:
```javascript
const fs = require('fs');
fs.appendFileSync('./docs/LEARNING_LOG.md', `\n${new Date().toISOString()} – ${$json.summary}`);
return $input.all();
```

---

## 🧩 Abschnitt 8 – Praxis: Mini-Selbstlern-Cycle

1. Führe Evaluation (Kap. 12) täglich automatisch aus.  
2. Analysiere Metriken → LLM schlägt Parameter-Anpassung vor.  
3. Schreibe Vorschläge in `LEARNING_LOG.md`.  
4. Bestätige manuell oder über Review-Node.  
5. Übernehme nach Validierung in `risk_config.json`.  

So entsteht ein lernfähiger, aber kontrollierter Agent.

---

## 🧭 Abschnitt 9 – Reflexion

- Wann ist Lernen sinnvoll, wann gefährlich (z. B. Überanpassung)?  
- Wie viel Automatisierung würdest du zulassen?  
- Wie stellst du sicher, dass Experimente nachvollziehbar bleiben?  

---

## ✅ Zusammenfassung

Nach Kapitel 14 kannst du:
- deinen Agenten zu einem adaptiven System erweitern,  
- Feedback-Loops aus Logs und Metriken einbauen,  
- automatische Lernvorschläge ausführen und prüfen,  
- und langfristig stabile, selbstkorrigierende Strategien entwickeln.  

Das nächste Kapitel (15) führt diese Idee in die Zukunft: **Multi-Agent-Kollaboration und hybride Entscheidungsnetze**, in denen mehrere spezialisierte Systeme gemeinsam Marktentscheidungen treffen – ähnlich wie Teams aus Analysten, Bots und Strategiemodulen.