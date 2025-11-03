# 📘 Kapitel 8 – Performance-Optimierung und Fehler-Recovery

**Lernziel:**  
Nach dieser Lektion kannst du deine Workflows so strukturieren, dass sie **schneller**, **stabiler** und **ausfallsicherer** laufen.  
Du lernst, wie du Flaschenhälse erkennst, Logs zur Laufzeitanalyse nutzt und deinen Agenten nach Fehlern automatisch wiederherstellen lässt.

---

## 🧩 Abschnitt 1 – Warum Performance im Multi-Agent-System kritisch ist

Mit jedem zusätzlichen Node steigen:
- Ausführungszeit  
- API-Aufrufe  
- Tokenkosten  
- Fehlerwahrscheinlichkeit  

Ziel ist also **Effizienz ohne Funktionsverlust**.  
Ein performanter Flow ist nicht der kürzeste, sondern der, der die **richtigen Aufgaben am richtigen Ort** ausführt – möglichst parallel, mit klar definierten Rückfällen (Fallbacks), falls etwas schiefläuft.

---

## ⚙️ Abschnitt 2 – Performance messen

n8n liefert im *Execution Panel* bereits Basisdaten:
- **Execution time:** Gesamtlaufzeit pro Run  
- **Node timings:** Dauer pro Node  
- **Memory footprint:** angezeigt in Logs  

### Debugging-Tipp:
Ergänze in jedem kritischen Node eine Messung der Laufzeit:
```javascript
const start = Date.now();
// … dein Code …
const duration = Date.now() - start;
return [{ ...$json, duration_ms: duration }];
```
So kannst du im Data Store später Laufzeiten auswerten.

Optional: lege eine eigene Tabelle `performance_logs` an mit Feldern:
`workflow`, `node`, `duration_ms`, `timestamp`.

---

## 🧠 Abschnitt 3 – Parallelisierung: schneller durch Batches

Wenn dein Flow mehrere unabhängige Symbol-Analysen ausführt (z. B. BTC, EUR/USD, Gold), kannst du sie parallel laufen lassen.

Nutze:
- **Split In Batches Node:** um Daten auf mehrere Flows zu verteilen.  
- **Workflow Execute Node:** um Sub-Flows parallel zu starten.  

Beispiel:
```
[Webhook Trigger]
   ↓
[Function → Liste der Symbole]
   ↓
[Split In Batches → Workflow Execute]
```

Jeder Sub-Flow behandelt ein Symbol – so halbierst du Laufzeit bei stabiler API-Performance.

**Debugging-Hinweis:**  
Beobachte die *Execution IDs* im Dashboard. Wenn eine Batch fehlschlägt, kannst du sie gezielt neu starten, ohne den ganzen Run zu wiederholen.

---

## 💡 Abschnitt 4 – Fehler-Recovery: dein Agent heilt sich selbst

Fehler sind unvermeidlich, aber sie dürfen den Gesamtprozess nicht stoppen.  

### Strategie 1: Error Workflow  
Wie schon in Kapitel 7 erwähnt – dieser Workflow wird bei jedem Fehler getriggert.  
Erstelle ein standardisiertes Recovery-Schema:

```
[Error Trigger]
   ↓
[Function → Fehlerklassifikation]
   ↓
[Switch → Entscheidung]
   ↓
[Retry Workflow] oder [Notify Telegram]
```

**Function Node (Beispiel):**
```javascript
const e = $json.error || {};
if (e.message.includes("timeout")) return [{ type: "retry" }];
if (e.message.includes("invalid JSON")) return [{ type: "parse_error" }];
return [{ type: "notify" }];
```

So unterscheidet dein System automatisch zwischen „nochmal versuchen“ und „manuell eingreifen“.

---

## 🧩 Abschnitt 5 – Wiederholungslogik (Retries)

In normalen Nodes kannst du unter *Settings → Max. Tries* Wiederholungen aktivieren.  
Alternativ per Node-Logik:
```javascript
let attempts = 0;
let success = false;
while (!success && attempts < 3) {
  try {
    // API-Aufruf
    success = true;
  } catch (err) {
    attempts++;
    console.warn(`Retry ${attempts}`);
    await new Promise(r => setTimeout(r, 2000)); // Pause zwischen Versuchen
  }
}
return [{ attempts }];
```

Diese Schleifen-Nodes schützen dich vor kurzzeitigen Netzwerkproblemen.

---

## ⚙️ Abschnitt 6 – Logging für Debugging und Analyse

Lege für jede wichtige Ausführung eine Log-Datei an, z. B. im `logs/`-Ordner deines Projekts.

**Function Node:**
```javascript
const fs = require("fs");
const logPath = `/data/logs/run_${new Date().toISOString()}.json`;
fs.writeFileSync(logPath, JSON.stringify($json, null, 2));
return $input.all();
```

Alternativ: nutze den n8n-Data-Store oder ein externes Tool (z. B. Loki, Grafana, InfluxDB).  
Logs sind Gold – sie sind der Unterschied zwischen „Fehler“ und „Erkenntnis“.

---

## 🧩 Abschnitt 7 – Ressourcen sparen mit Conditional Execution

Jede LLM-Abfrage kostet Zeit und Tokens.  
Deshalb: nur dann anfragen, wenn neue Daten vorliegen oder sich Marktbedingungen stark geändert haben.

**Function Node (Vorprüfung):**
```javascript
if ($json.lastUpdate && Date.now() - new Date($json.lastUpdate).getTime() < 60000) {
  return []; // Überspringe, wenn zu frisch
}
return [$json];
```

So vermeidest du unnötige API-Calls.

---

## 🧩 Abschnitt 8 – Praxis: Recovery-Flow mit automatischer Wiederaufnahme

1. Baue deinen **Error Workflow**, der Timeouts erkennt und fehlgeschlagene Runs neu startet.  
2. Implementiere Retry-Schleifen in kritischen Nodes.  
3. Logge alle Fehler systematisch in `error_log` Data Store.  
4. Ergänze einen Telegram-Node, der bei dreimaligem Fehlschlag eine Nachricht sendet.  
5. Führe Stresstest aus: 10 parallele Ausführungen → prüfe Logs auf Ausfälle.  

**Debugging-Checkliste:**
- Werden Fehlermeldungen im Error Workflow sichtbar?  
- Funktioniert Retry korrekt?  
- Bleibt der Agent nach Recovery konsistent (keine doppelten Einträge)?  

---

## 🧭 Abschnitt 9 – Reflexion

- Wie erkennst du in Logs die wahren Ursachen (Logikfehler vs. externe Ausfälle)?  
- Welche Schwachstelle ist bei dir am anfälligsten für Timeouts?  
- Wie würdest du mit zu hohen Tokenkosten umgehen – eher caching oder weniger Detail im Prompt?

---

## 🧩 Abschnitt 10 – Hausaufgabe / Experiment

1. Erstelle einen Recovery-Workflow mit Telegram-Benachrichtigung.  
2. Simuliere absichtlich einen Fehler (z. B. ungültigen JSON-Response) und beobachte die Recovery.  
3. Miss die Laufzeiten einzelner Nodes und speichere sie.  
4. Reduziere Laufzeit durch Batch-Ausführung oder Vorprüfung.  
5. Notiere vor und nach der Optimierung: „Durchschnittliche Laufzeit“, „Tokenverbrauch“, „Fehlerrate“.

---

## ✅ Zusammenfassung

Nach Kapitel 8 kannst du:
- Flows auf Effizienz und Stabilität optimieren,  
- Fehler erkennen, loggen und automatisch behandeln,  
- Ressourcenverbrauch steuern,  
- und mit Logs gezielt Performance verbessern.  

Im nächsten Kapitel (9) lernst du, wie du aus diesen Logs und Feedbacks **automatische Qualitätsmetriken** generierst – um deinen Agenten langfristig zu bewerten und zu verbessern.