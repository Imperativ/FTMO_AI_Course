# 📘 Kapitel 3 – ChatGPT-Integration vertiefen und Datenanalyse vorbereiten

**Lernziel:**
Nach dieser Lektion kannst du das aktuelle LLM-Modell gezielt als _Analystenkomponente_ in deinen Workflows einsetzen.
Du lernst, wie du strukturierte Marktdaten an die API übergibst, eine saubere Antwort erhältst und daraus verwertbare Informationen machst.

---

## 🧩 Abschnitt 1 – Warum KI im Trading nur so gut ist wie die Daten

Sprachmodelle können keine Kurse „sehen“, sie verstehen nur, was du ihnen beschreibst.
Damit sie also eine Marktsituation bewerten können, musst du sie **in Text oder JSON-Form** übersetzen.

Wenn du einem Modell bloß sagst:

> „Analysiere BTC/USD.“

dann weiß es nichts außer diesem Satz.
Wenn du hingegen sendest:

```json
{
  "symbol": "BTCUSD",
  "interval": "1h",
  "data": [
    {
      "time": "2025-11-03T10:00Z",
      "open": 68740,
      "high": 68810,
      "low": 68690,
      "close": 68795,
      "volume": 2410
    },
    {
      "time": "2025-11-03T11:00Z",
      "open": 68795,
      "high": 68950,
      "low": 68780,
      "close": 68920,
      "volume": 2554
    }
  ]
}
```

– dann hat es einen Kontext, um z. B. Trend, Momentum oder Volatilität zu beurteilen.

---

## ⚙️ Abschnitt 2 – Der GPT-Node in n8n

n8n stellt keinen eigenen GPT-Node „out of the box“ bereit, aber du kannst dieselbe Funktion auf zwei Arten erreichen:

**Variante 1:** Offizieller „OpenAI Node“ (falls installiert)
**Variante 2:** „HTTP Request“ Node – universell und flexibel.

Wir nutzen hier Variante 2, da sie immer verfügbar ist.

1. Füge in deinen Workflow einen **HTTP Request Node** ein.
2. Methode: `POST`
3. URL:
   ```
   https://api.openai.com/v1/chat/completions
   ```
4. Header:
   ```
   Authorization: Bearer {{$env.OPENAI_API_KEY}}
   Content-Type: application/json
   ```
5. Body:
   ```json
   {
     "model": "aktuelles LLM Modell",
     "messages": [
       { "role": "system", "content": "Du bist ein Finanzdaten-Analyst." },
       { "role": "user", "content": "Analysiere folgende Kursdaten: {{$json}}" }
     ]
   }
   ```

Ergebnis: Der Output des Nodes enthält ein JSON-Feld `choices[0].message.content` mit der Analyse.

---

## 🧠 Abschnitt 3 – Gute Prompts für strukturierte Ausgaben

Um aus LLMs mehr als nur Prosa zu bekommen, trainierst du sie über **Prompt-Struktur**.

Schlechtes Beispiel:

> „Was hältst du von diesen Kursen?“

Besser:

> „Analysiere die Kursdaten. Gib dein Ergebnis als JSON mit den Feldern
> `trend`, `momentum`, `risk_level`, `recommendation` zurück.“

Ergebnis:

```json
{
  "trend": "bullish",
  "momentum": 0.72,
  "risk_level": "low",
  "recommendation": "long"
}
```

Dieser Output kann im nächsten Node direkt weiterverarbeitet werden, z. B. in einem Telegram-Alert oder einem eigenen Dashboard.

---

## 💡 Abschnitt 4 – Versionen und Tests

Bevor du produktiv arbeitest:

- Prüfe die n8n-Version (`n8n --version` oder Menü „Help → About“).
- Teste API-Keys mit minimalen Requests, um Rate-Limits zu kennen.
- Achte darauf, dass das Modell-Feld in deinem HTTP-Node dem tatsächlich verfügbaren Modell entspricht.

So bleibt dein Workflow zukunftssicher, falls Anbieter Endpunkte ändern.

---

## ⚠️ Abschnitt 5 – Bewusster Umgang mit Risiken

Dieses System bewertet Marktdaten anhand von Sprachmodellen.
Es ist kein Garant für Profit, sondern ein Werkzeug, um **Hypothesen zu generieren**.
Du bleibst stets Entscheider.
Ein gutes Demokonto ist daher nicht nur Übungsumgebung, sondern Sicherheitsgurt.

---

## 🧩 Abschnitt 6 – Mini-Praxis: Dein erster Daten-Analyst

1. Baue in n8n folgenden Flow:
   - Node 1: **Webhook** (`POST`).
   - Node 2: **Function Node**, die Beispiel-Marktdaten erzeugt (wie oben gezeigt).
   - Node 3: **HTTP Request Node** (aktuelles LLM-Modell).
   - Node 4: **Telegram Node**, sendet `choices[0].message.content`.

2. Teste den Flow durch manuelles Auslösen des Webhooks.

3. Beobachte die Antwort:
   - Liefert sie eine Struktur?
   - Enthält sie quantitative Aussagen (z. B. „Momentum 0.8“)?
   - Kannst du das JSON weiterverwenden?

---

## 🧭 Abschnitt 7 – Reflexion

- Wie klar war deine Kommunikation mit dem Modell?
- Was passiert, wenn du die Datenmenge vergrößerst – wird die Antwort diffuser?
- Wie würdest du den Prompt umformulieren, um präzisere Analysen zu erhalten?

Schreibe drei Varianten deines Prompts auf und vergleiche die Outputs.
Das ist praktisches Prompt-Tuning – die Grundlage für deinen späteren Agenten.

---

## 🧩 Abschnitt 8 – Hausaufgabe / Experiment

1. Ergänze den Workflow um einen **Switch-Node**, der bei `"trend": "bullish"` automatisch eine andere Nachricht sendet als bei `"bearish"`.
2. Logge die Ausgabe zusätzlich in einer Datei (`Write Binary File` Node).
3. Dokumentiere in deiner Kursmappe, wie du Prompt- und Node-Einstellungen verändert hast, um konsistente JSON-Outputs zu erhalten.

---

## 🚨 Abschnitt 9 – API & LLM Debugging

### OpenAI API gibt 401 zurück

**Problem:** "Invalid API key"
**Diagnose-Schritte:**

1. API-Key in n8n Environment prüfen: Settings → Environment Variables
2. Test-Call mit curl:
   ```bash
   curl -H "Authorization: Bearer YOUR_KEY" \
        -H "Content-Type: application/json" \
        https://api.openai.com/v1/models
   ```
3. Rate-Limits checken: API-Response-Header `x-ratelimit-remaining`

### LLM gibt inkonsistente JSON zurück

**Problem:** Mal `{"trend":"bullish"}`, mal `Trend: bullish (Text)`
**Debugging-Prompt:**

```
Antworte ausschließlich im folgenden JSON-Format, ohne zusätzlichen Text:
{"trend": "bullish|bearish|neutral", "confidence": 0.XX}

Beispiel: {"trend": "bullish", "confidence": 0.73}
```

**Validation-Node hinzufügen:**

```javascript
try {
  const response = JSON.parse($json.choices[0].message.content);
  return [response];
} catch (e) {
  return [{ error: "Invalid JSON", raw: $json.choices[0].message.content }];
}
```

### Timeout-Probleme

**Problem:** HTTP-Request hängt oder bricht ab
**Lösung:**

- HTTP-Node → Options → Timeout auf 30000ms setzen
- Retry-Logik: Options → "Retry on Fail" aktivieren
- Fallback-Strategy mit Switch-Node für Error-Handling

### Error-Handling Template

Für kritische API-Calls:

```javascript
try {
  const result = $json.choices[0].message.content;
  const parsed = JSON.parse(result);
  return [{ success: true, data: parsed }];
} catch (error) {
  return [
    {
      success: false,
      error: error.message,
      timestamp: new Date().toISOString(),
      input: $json,
    },
  ];
}
```

---

## ✅ Zusammenfassung

Nach Kapitel 3 kannst du:

- das aktuelle LLM-Modell gezielt über n8n ansprechen,
- strukturierte Daten einspeisen und verwertbare JSON-Analysen erzeugen,
- und die Ergebnisse logisch weiterverarbeiten.

Im nächsten Kapitel geht es um den Schritt von **Datenanalyse zu Entscheidungslogik** – wie dein Agent Risiko kalkuliert, Positionsgrößen vorschlägt und sich in größere Strategien einbettet.
