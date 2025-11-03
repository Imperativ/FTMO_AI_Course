# 📘 Kapitel 5 – Gedächtnis & Feedback: Deinen Agenten lernfähig machen

**Lernziel:**
Nach dieser Lektion kannst du deinem Agenten **Persistenz** (Speicher) geben, Ergebnisse **protokollieren** und einfache **Feedback-Schleifen** bauen.
Du lernst, wie du Signale, Entscheidungen und Outcomes speicherst und daraus Regeln justierst – ohne den Agenten autonom handeln zu lassen.

---

## 🧩 Abschnitt 1 – Warum ein Gedächtnis unverzichtbar ist

Ohne Speicher gibt es keine Verbesserung. Dein Agent muss „wissen", was er **wann** empfohlen hat und **wie es ausging**.
Wir speichern daher für _jeden_ Durchlauf mindestens:

- `timestamp` – Zeitpunkt der Empfehlung
- `symbol`, `interval` – Kontext
- `analysis` – strukturierter LLM-Output (z. B. `trend`, `momentum`, `reversal_risk`)
- `risk_params` – verwendete Risikowerte (z. B. `risk`, `position_size`)
- `decision` – Empfehlung (long/short/no-trade)
- `outcome` – späteres Ergebnis (TP/SL/neutral), wird nachgetragen

So entstehen **Daten für Evaluierung** und **Parameter-Tuning**.

---

## ⚙️ Abschnitt 2 – Einfache Persistenz in n8n (Data Store)

n8n bietet einen **Data Store** (eingebauter Key-Value/Tabellen-Speicher). Für unseren Zweck ideal, weil schnell & ohne externen Dienst.

**Setup-Schritte (einmalig):**

1. In n8n: Menü → _Resources_ → _Data Stores_ → **Create Data Store**
   - Name: `ftmo_signals`
   - Schema (Beispiel):
     ```json
     {
       "timestamp": "string",
       "symbol": "string",
       "interval": "string",
       "trend": "string",
       "momentum": "number",
       "reversal_risk": "number",
       "risk": "number",
       "position_size": "number",
       "decision": "string",
       "outcome": "string"
     }
     ```
2. Notiere dir die **Data Store ID** (brauchst du im Node).

**Im Workflow:**
Füge einen **Data Store - Insert** Node hinzu und übergib die Felder aus deinem Flow.

---

## 🧠 Abschnitt 3 – Flow mit Speicher & Update-Mechanik

Wir erweitern den Flow aus Kapitel 4 um Speicherung und nachträgliches Update des Outcomes.

```
[Webhook Trigger]
   ↓
[Get Market Data]
   ↓
[HTTP Request → aktuelles LLM Modell]
   ↓
[Function → Risiko-/Positionsgröße]
   ↓
[Switch → Entscheidung]
   ↓
[Data Store Insert → (record_id)]
   ↓
[Telegram → Empfehlung inkl. record_id]
```

**Beispiel: Insert-Payload (Function Node davor):**

```javascript
const now = new Date().toISOString();
const rec = {
  timestamp: now,
  symbol: $json.symbol || "BTCUSD",
  interval: $json.interval || "1h",
  trend: $json.trend || "neutral",
  momentum: $json.momentum ?? null,
  reversal_risk: $json.reversal_risk ?? null,
  risk: $json.risk ?? null,
  position_size: $json.position_size ?? null,
  decision: $json.decision || "observe",
  outcome: "", // wird später gesetzt
};
return [rec];
```

Der **Data Store Insert** gibt dir eine `record_id` zurück. Sende diese im Telegram-Text mit, damit du später genau diesen Eintrag updaten kannst.

---

## 🧩 Abschnitt 4 – Outcome nachtragen (Update-Path)

Sobald du weißt, wie der Trade ausgegangen wäre (z. B. nach X Stunden), kannst du per **Webhook** eine Outcome-Aktualisierung anstoßen.

**Zweiter Mini-Workflow:**

```
[Webhook /update_outcome]
   ↓
[Data Store - Update by ID]
```

**Erwarteter `POST`-Body:**

```json
{
  "record_id": "abc123",
  "outcome": "TP" // TP | SL | neutral
}
```

**Update-Node (Data Store - Update by ID):**

- Data Store: `ftmo_signals`
- Key: `record_id`
- Fields: `outcome`

Damit wächst dein Datensatz über Zeit – perfekt für spätere Auswertungen.

---

## 💡 Abschnitt 5 – Mini-Evaluierung: einfache Kennzahlen in n8n

Mit ein paar Zeilen JavaScript kannst du direkt in n8n einen **Report** erzeugen.

**Data Store - Query** → hole letzte N Einträge → **Function Node**:

```javascript
const items = $input.all(); // Data Store Records
const total = items.length;
const wins = items.filter((i) => i.json.outcome === "TP").length;
const losses = items.filter((i) => i.json.outcome === "SL").length;
const neutral = items.filter((i) => i.json.outcome === "neutral").length;
const winrate = total ? wins / total : 0;

return [
  {
    total,
    wins,
    losses,
    neutral,
    winrate,
  },
];
```

Sende das Ergebnis an **Telegram** oder speichere es als CSV.

---

## 🧭 Abschnitt 6 – Reflexion

- Welche Felder fehlen dir, um Entscheidungen später besser beurteilen zu können (z. B. ATR, Spread, Session)?
- Wie häufig willst du Outcomes nachtragen (zeitlich vs. per Event)?
- Ab welcher Winrate/Expectancy würdest du Parameter ändern – und **welche** zuerst?

Diese Fragen schärfen deinen Evaluationsprozess, bevor du „Parameter-Tuning“ automatisierst.

---

## 🧩 Abschnitt 7 – Hausaufgabe / Experiment

1. Erstelle den **Insert-Flow** mit Data Store und Telegram-Alert (inkl. `record_id`).
2. Baue den **Update-Flow** (`/update_outcome`) und trage Outcomes für drei Signale nach.
3. Erzeuge einen **Mini-Report** (Winrate etc.) und sende ihn an dich.
4. Bonus: Logge den Report zusätzlich als CSV-Datei (für spätere Analysen in Python/R).

---

## 🚨 Abschnitt 8 – Persistenz & Data Store Debugging

### Data Store Insert schlägt fehl

**Problem:** "Schema validation failed"
**Debugging-Schritte:**

1. Schema vs. Daten vergleichen:

   ```javascript
   // Debug: Zeige was geschickt wird
   const payload = {
     timestamp: new Date().toISOString(),
     symbol: $json.symbol?.toString() || "UNKNOWN",
     trend: $json.trend?.toString() || "neutral",
     momentum: parseFloat($json.momentum) || 0,
   };
   console.log("Data Store Payload:", payload);
   return [payload];
   ```

2. Data Store Schema prüfen: Resources → Data Stores → Schema anschauen
3. Datentyp-Konvertierung hinzufügen

### Update by ID funktioniert nicht

**Problem:** Record wird nicht gefunden
**Debug-Query:**

```javascript
// Prüfe ob record_id existiert
const recordId = $json.record_id;
if (!recordId) {
  return [{ error: "Missing record_id" }];
}
return [{ record_id: recordId, outcome: $json.outcome }];
```

### Telegram Bot antwortet nicht

**Problem:** Bot-Token oder Chat-ID falsch
**Quick-Test:**

```bash
# Test Bot-Token
curl "https://api.telegram.org/bot[YOUR_TOKEN]/getMe"

# Test Nachricht senden
curl -X POST "https://api.telegram.org/bot[YOUR_TOKEN]/sendMessage" \
     -d "chat_id=[YOUR_CHAT_ID]&text=Test"
```

### Data Store Backup & Recovery

**Wichtiger Tipp:** Regelmäßig exportieren!

```javascript
// Export-Function für Backup
const items = $input.all();
const backup = {
  export_date: new Date().toISOString(),
  record_count: items.length,
  data: items.map((item) => item.json),
};
return [{ backup_data: JSON.stringify(backup, null, 2) }];
```

---

## ⚠️ Abschnitt 9 – Hinweise & Grenzen

- Persistenz ≠ Wahrheit: Wenn du Outcomes zu spät oder falsch nachträgst, verzerrst du die Statistik.
- Halte dich an deine **Risikoregeln**. Der Agent ist ein Berater.
- Denke an **Versionierung**: Notiere größere Prompt-/Regeländerungen in deiner Kursmappe, damit du Ergebnisse einordnen kannst.

---

## ✅ Zusammenfassung

Nach Kapitel 5 kannst du:

- Ergebnisse in n8n **persistieren**,
- **Outcomes** nachtragen und Berichte erzeugen,
- die Grundlage für **Feedback- und Lernschleifen** legen.

Im nächsten Kapitel baust du darauf auf: **Parameter-Tuning und einfache A/B-Vergleiche** – damit dein Agent systematisch besser wird, statt zufällig.
