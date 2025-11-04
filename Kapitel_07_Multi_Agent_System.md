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

So siehst du im _Execution-Log_ genau, was jeder Agent geliefert hat.

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

| Feld            | Bedeutung                 |
| --------------- | ------------------------- |
| role            | Analyst / Risk / Strategy |
| timestamp       | Zeitpunkt der Ausführung  |
| input_snapshot  | JSON der Eingabe          |
| output_snapshot | JSON der Ausgabe          |
| error_message   | optional                  |

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

## 🚨 Abschnitt 10 – Multi-Agent System Debugging

### Rollen-Kommunikation Debugging

**Problem:** Daten gehen zwischen Agent-Rollen verloren
**Debug-Strategie:**

```javascript
// Inter-Agent Data Validation
const validateAgentHandoff = (inputData, expectedFields, agentName) => {
  console.log(`=== ${agentName.toUpperCase()} INPUT VALIDATION ===`);
  console.log("Received data:", JSON.stringify(inputData, null, 2));

  const missing = expectedFields.filter((field) => !(field in inputData));
  if (missing.length > 0) {
    const error = `${agentName} missing required fields: ${missing.join(", ")}`;
    console.error(error);
    throw new Error(error);
  }

  console.log(`${agentName} validation passed ✓`);
  return inputData;
};

// Usage in each agent node
const analystData = validateAgentHandoff(
  $json,
  ["symbol", "ohlc_data"],
  "Analyst",
);
```

### JSON-Parsing Fehler beheben

**Problem:** Agent liefert inkonsistente JSON-Struktur
**Robuste Parsing-Lösung:**

````javascript
const parseAgentResponse = (response, agentName) => {
  let content =
    response.choices?.[0]?.message?.content || response.content || response;

  // Verschiedene Content-Formate abfangen
  if (typeof content !== "string") {
    content = JSON.stringify(content);
  }

  // Bereinige häufige LLM-Ausgabe-Probleme
  content = content
    .replace(/```json\n?/g, "")
    .replace(/```\n?/g, "")
    .replace(/^\s*```.*$/gm, "")
    .trim();

  try {
    const parsed = JSON.parse(content);
    console.log(`${agentName} JSON parsed successfully ✓`);
    return parsed;
  } catch (error) {
    console.error(`${agentName} JSON parsing failed:`, error.message);
    console.error("Raw content:", content);

    // Fallback: Versuche JSON aus Text zu extrahieren
    const jsonMatch = content.match(/\{[\s\S]*\}/);
    if (jsonMatch) {
      try {
        const extracted = JSON.parse(jsonMatch[0]);
        console.log(`${agentName} JSON extraction successful ✓`);
        return extracted;
      } catch (extractError) {
        console.error(
          `${agentName} JSON extraction also failed:`,
          extractError.message,
        );
      }
    }

    // Letzter Fallback: Strukturierte Fehlerantwort
    return {
      error: true,
      agent: agentName,
      message: "JSON parsing failed",
      raw_content: content,
      timestamp: new Date().toISOString(),
    };
  }
};
````

### Agent-Flow Monitoring

**Problem:** Unklare Performance-Engpässe zwischen Agenten
**Monitoring-Template:**

```javascript
// Performance-Tracking für Multi-Agent-Systeme
const trackAgentPerformance = (agentName, startTime, inputSize, outputData) => {
  const endTime = Date.now();
  const duration = endTime - startTime;
  const tokenEstimate = JSON.stringify(outputData).length / 4; // Grobe Token-Schätzung

  const metrics = {
    agent: agentName,
    execution_time_ms: duration,
    input_size_bytes: JSON.stringify(inputSize).length,
    output_size_bytes: JSON.stringify(outputData).length,
    estimated_tokens: Math.ceil(tokenEstimate),
    timestamp: new Date().toISOString(),
    success: !outputData.error,
  };

  console.log(`Agent Performance [${agentName}]:`, metrics);

  // Speichere in Data Store für spätere Analyse
  return {
    ...outputData,
    _performance: metrics,
  };
};

// Usage example
const startTime = Date.now();
const result = await processWithAgent(inputData);
return trackAgentPerformance("Risk-Manager", startTime, inputData, result);
```

### Error-Recovery für Agent-Ketten

**Problem:** Ein Agent-Fehler stoppt die ganze Kette
**Resilient Agent Chain:**

```javascript
const executeAgentChain = async (initialData, agents) => {
  let currentData = initialData;
  const executionLog = [];

  for (const agent of agents) {
    const stepStart = Date.now();

    try {
      console.log(`Executing agent: ${agent.name}`);

      // Retry-Logik für jeden Agent
      let attempts = 0;
      let success = false;
      let result = null;

      while (!success && attempts < 3) {
        try {
          result = await agent.execute(currentData);
          success = true;
        } catch (error) {
          attempts++;
          console.warn(
            `Agent ${agent.name} failed (attempt ${attempts}):`,
            error.message,
          );

          if (attempts < 3) {
            await new Promise((resolve) =>
              setTimeout(resolve, 1000 * attempts),
            ); // Exponential backoff
          }
        }
      }

      if (!success) {
        // Fallback: Verwende Default-Werte oder überspringe Agent
        console.error(
          `Agent ${agent.name} failed after 3 attempts, using fallback`,
        );
        result = agent.fallback ? agent.fallback(currentData) : currentData;
      }

      executionLog.push({
        agent: agent.name,
        duration_ms: Date.now() - stepStart,
        attempts,
        success,
        input_hash: hashObject(currentData),
        output_hash: hashObject(result),
      });

      currentData = result;
    } catch (criticalError) {
      console.error(`Critical error in agent ${agent.name}:`, criticalError);

      // Emergency fallback
      executionLog.push({
        agent: agent.name,
        duration_ms: Date.now() - stepStart,
        attempts: 1,
        success: false,
        error: criticalError.message,
      });

      // Entscheide: Abbrechen oder mit degraded mode fortfahren
      if (agent.critical) {
        throw new Error(
          `Critical agent ${agent.name} failed: ${criticalError.message}`,
        );
      }
    }
  }

  return {
    result: currentData,
    execution_log: executionLog,
    total_duration_ms: executionLog.reduce(
      (sum, log) => sum + log.duration_ms,
      0,
    ),
  };
};
```

### Agent-Kommunikation Protokoll

**Problem:** Inkonsistente Datenformate zwischen Agenten
**Standardisiertes Kommunikations-Schema:**

```javascript
// Agent Message Protocol
const createAgentMessage = (fromAgent, toAgent, data, messageType = "data") => {
  return {
    protocol_version: "1.0",
    message_id: generateUUID(),
    timestamp: new Date().toISOString(),
    from_agent: fromAgent,
    to_agent: toAgent,
    message_type: messageType, // 'data', 'error', 'status'
    payload: data,
    checksum: hashObject(data),
  };
};

const validateAgentMessage = (message, expectedFromAgent) => {
  const required = [
    "protocol_version",
    "message_id",
    "timestamp",
    "from_agent",
    "payload",
  ];
  const missing = required.filter((field) => !(field in message));

  if (missing.length > 0) {
    throw new Error(`Invalid agent message: missing ${missing.join(", ")}`);
  }

  if (expectedFromAgent && message.from_agent !== expectedFromAgent) {
    throw new Error(
      `Expected message from ${expectedFromAgent}, got from ${message.from_agent}`,
    );
  }

  // Checksum-Validierung
  const expectedChecksum = hashObject(message.payload);
  if (message.checksum !== expectedChecksum) {
    throw new Error(
      "Agent message checksum mismatch - data corruption detected",
    );
  }

  return message.payload;
};
```

---

## ✅ Zusammenfassung

Nach Kapitel 7 kannst du:

- mehrere spezialisierte Agentenrollen orchestrieren,
- Debugging systematisch einbauen,
- Fehlermeldungen automatisch abfangen,
- robuste Agent-zu-Agent-Kommunikation implementieren,
- und Logs für Nachvollziehbarkeit speichern.

Im nächsten Kapitel geht es um **Performance-Optimierung und Fehler-Recovery**, also wie du Flows effizienter machst und sie sich bei Ausfällen selbst heilen.
