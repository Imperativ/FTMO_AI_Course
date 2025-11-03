# 📘 Kapitel 7 – Multi-Agent-System: Rollen, Kommunikation und Debugging

**Lernziel:**  
Nach dieser Lektion kannst du mehrere spezialisierte Agentenrollen (Analyst, Risiko-Manager, Strategieberater) koordinieren.  
Du lernst, wie diese Rollen in n8n zusammenarbeiten, wie Daten zwischen ihnen fließen und wie du komplexe Fehler zuverlässig aufspürst.

---

## 🧩 Abschnitt 1 – Warum Aufgabenteilung den Agenten stabiler macht

Ein einzelnes Modell, das „alles“ tun soll, neigt zu unklaren Ausgaben.  
Die Lösung: **Arbeitsteilung**.  
Jede Rolle bearbeitet nur einen Aspekt – das verhindert, dass sich Analysen, Risiko und Strategie gegenseitig beeinflussen.

Beispiel:  
- **Analyst:** interpretiert Marktdaten  
- **Risk-Manager:** berechnet Positionsgröße, prüft Drawdown  
- **Strategieberater:** bewertet Signal im Gesamtkontext  

Jede Rolle bekommt ihr eigenes Prompt-Template und eigenen Node-Pfad.  

---

## ⚙️ Abschnitt 2 – Grundstruktur eines Multi-Agent-Flows

Ein einfaches Multi-Agent-System in n8n sieht so aus:

```
[Webhook Trigger]
   ↓
[LLM Node → Analyst]
   ↓
[Function Node → Risikoformel]
   ↓
[LLM Node → Risk-Manager]
   ↓
[LLM Node → Strategieberater]
   ↓
[Decision Node (Switch)]
   ↓
[Data Store Insert]
   ↓
[Telegram Alert]
```

Damit alle Rollen sauber getrennt bleiben, nutzt du je Rolle ein eigenes Prompt-Template mit klaren Ein-/Ausgabe-Feldnamen.

---

## 🧠 Abschnitt 3 – Prompt-Design für Rollen

**Analyst-Prompt:**
```
Du bist ein Datenanalyst. 
Eingabe: Marktdaten (Open, High, Low, Close, Volume).
Ausgabe: JSON mit trend, momentum, volatility.
```

**Risk-Manager-Prompt:**
```
Du bist ein Risiko-Analyst. 
Eingabe: trend, momentum, volatility.
Berechne reversalscore (0–1) und Risiko-Level (low/medium/high).
```

**Strategieberater-Prompt:**
```
Du bist Strategieberater.
Eingabe: reversalscore, trend, momentum, Risiko-Level.
Empfehle Handlung: long, short oder no-trade.
```

Damit baust du eine **logische Pipeline** aus Sprache → Daten → Bewertung.

---

## 💡 Abschnitt 4 – Debugging-Werkzeuge in n8n

Da hier mehrere LLM-Nodes und Variablenketten beteiligt sind, ist Debugging Pflicht.  

### 🔍 Tipp 1: „Always log“
Nach jedem LLM-Node einen **Function-Node** mit
```javascript
console.log($json);
return $input.all();
```
So siehst du im *Execution-Log* genau, was jeder Agent geliefert hat.

### 🔍 Tipp 2: „Use Error Trigger“
Lege zusätzlich einen **Error Workflow** an (unter „Workflows → Error Workflow“).  
Dieser wird automatisch aufgerufen, wenn ein Node fehlschlägt.  
Füge dort z. B. eine Telegram-Benachrichtigung mit `{{$json.error.message}}` ein.

### 🔍 Tipp 3: „Test Data Outputs“
Nutze in jedem LLM-Node das Feld **Test Output**, um Prompt und Antwort vorab zu prüfen.  
So erkennst du Formatfehler, bevor du die Nodes verkettest.

### 🔍 Tipp 4: „JSON.parse() Validierung“
Füge nach jedem LLM-Node eine **Function-Node** ein:
```javascript
try {
  const data = JSON.parse($json.text || $json.content || "{}");
  return [data];
} catch (e) {
  console.error("Parsing Error:", e);
  return [{ error: "invalid JSON", raw: $json }];
}
```
Damit schützt du den Flow vor unstrukturierten Antworten.

---

## 🧩 Abschnitt 5 – Kommunikation zwischen Rollen

Rollen kommunizieren über JSON-Objekte.  
Damit keine Information verloren geht, verwende **eindeutige Feldnamen** und prüfe vor jedem Übergang die Struktur:

**Function-Node zur Validierung:**
```javascript
const req = $json;
if (!req.trend || !req.momentum) {
  throw new Error("Analyst-Output unvollständig");
}
return [req];
```

Dadurch bricht der Flow sauber ab, statt still falsche Werte weiterzugeben.

---

## ⚙️ Abschnitt 6 – Data Store Logging pro Rolle

Für transparente Fehleranalyse solltest du nach jeder Hauptrolle einen Eintrag im Data Store anlegen:

| Feld             | Bedeutung |
|------------------|------------|
| role             | Analyst / Risk / Strategy |
| timestamp        | Zeitpunkt der Ausführung |
| input_snapshot   | JSON der Eingabe |
| output_snapshot  | JSON der Ausgabe |
| error_message    | optional |

Das erleichtert spätere Re-Runs und Nachvollziehbarkeit bei abweichenden Ergebnissen.

---

## 🧩 Abschnitt 7 – Praxis: Multi-Agent-Flow mit Logging

1. Kopiere deinen Flow aus Kapitel 6.  
2. Erstelle drei LLM-Nodes mit unterschiedlichen Prompts (Analyst, Risk-Manager, Strategieberater).  
3. Nach jedem Node:  
   - Function-Node für JSON-Parsing.  
   - Data-Store-Insert (role-basiertes Logging).  
   - Telegram-Debug-Node, der bei Fehlern Nachricht sendet.  
4. Verbinde alles zu einem Gesamt-Flow.  
5. Führe Test-Run mit Dummy-Daten aus und kontrolliere Logs.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Vorteile hat die Aufteilung in spezialisierte Rollen für Analysequalität?  
- Wie beeinflusst Debugging-Disziplin deine Fehlerrate?  
- Könnte man die Rollen künftig in unterschiedlichen LLM-Instanzen parallel laufen lassen?

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Baue den kompletten Multi-Agent-Flow wie oben beschrieben.  
2. Teste mit mindestens drei verschiedenen Symbolen (BTC/USD, EUR/USD, XAU/USD).  
3. Prüfe das Debug-Log: Findest du unvollständige JSONs? Füge Parsing-Korrekturen hinzu.  
4. Ergänze im Data Store pro Rolle Felder `latency_ms` und `token_usage`, um Performance zu beobachten.  
5. Erstelle aus dem Data-Store-Log eine CSV-Datei als Mini-Report.

---

## ✅ Zusammenfassung

Nach Kapitel 7 kannst du:
- mehrere spezialisierte Agentenrollen orchestrieren,  
- Debugging systematisch einbauen,  
- Fehlermeldungen automatisch abfangen,  
- und Logs für Nachvollziehbarkeit speichern.  

Im nächsten Kapitel geht es um **Performance-Optimierung und Fehler-Recovery**, also wie du Flows effizienter machst und sie sich bei Ausfällen selbst heilen.