# 📘 Kapitel 4 – Risikomanagement und Entscheidungslogik

**Lernziel:**
Nach dieser Lektion kannst du einfache Risiko- und Entscheidungsmodelle in n8n umsetzen, die deine KI-Analysen mit quantitativen Parametern verknüpfen.
Du verstehst, wie Positionsgröße, Drawdown-Kontrolle und Trefferquote algorithmisch abgebildet werden können – und wie das aktuelle LLM dabei als Berater dient, nicht als Entscheider.

---

## 🧩 Abschnitt 1 – Warum Entscheidungslogik notwendig ist

Analyse ist nur die halbe Miete. Ein Agent, der nicht weiß, _wann nicht zu handeln ist_, ist wertlos.
Das Ziel ist daher nicht, dass das Modell „rät“, sondern dass du ihm **Regeln** gibst, nach denen es seine Einschätzung gewichtet.

Beispiel – Wenn du nach einem LLM-Analysesignal „long“ handelst:

> _„Ich gehe nur dann Long, wenn Risiko kleiner als 2 %, Trendstärke > 0.7 und Momentum positiv ist.“_

Diese Bedingung lässt sich formal ausdrücken, z. B. als JSON-Logik, die du später in einem n8n-Switch oder Function Node abprüfst:

```json
{
  "risk": 0.015,
  "trend_strength": 0.8,
  "momentum": 0.3,
  "decision": "long"
}
```

---

## ⚙️ Abschnitt 2 – Risikoparameter definieren

Bevor du eine Entscheidung triffst, musst du definieren, welche Kennzahlen dein Agent überhaupt kennt.

Grundgrößen:

- **Kontogröße (Equity):** z. B. 10 000 € im Demokonto
- **max. Risiko pro Trade:** 1–2 %
- **Risk per Point oder Pip:** Abhängig vom Instrument
- **Stop-Loss Entfernung:** technisch oder prozentual
- **Risk/Reward Ratio:** mindestens 1 : 1.5 empfohlen

Daraus ergibt sich eine formelbasierte Positionsgröße:

```
Positionsgröße = (Kontogröße × Risiko_pro_Trade) ÷ Stop-Loss_Abstand
```

Diese Berechnung kannst du in n8n mit einem **Function Node** abbilden:

```javascript
const equity = 10000;
const riskPercent = 0.02;
const stopLossPoints = 50;
const pipValue = 1; // Beispiel
const positionSize = (equity * riskPercent) / (stopLossPoints * pipValue);
return [{ positionSize }];
```

---

## 🧠 Abschnitt 3 – Das LLM als Risiko-Assistent

Das Modell kann Risiko nicht berechnen, aber **einschätzen** – etwa basierend auf Volatilität oder Datenverteilung.
Du kannst ihm dazu einen Prompt geben, der Zahlen in Sprache übersetzt:

```
Analysiere diese Daten: Volatilität=0.013, Drawdown=1.8 %, Trendstärke=0.76.
Wie hoch ist das Risiko eines Gegentrends auf einer Skala von 0 bis 1?
```

Das LLM liefert etwa:

```json
{ "reversal_risk": 0.28, "confidence": 0.71 }
```

Dieser Wert kann dann in deinem n8n-Flow über einen Switch-Node in eine Entscheidungslogik einfließen.

---

## 🧩 Abschnitt 4 – Entscheidungsbaum im Workflow

Ein einfacher Agent-Flow in n8n:

```
[Webhook Trigger]
   ↓
[Get Market Data]
   ↓
[HTTP Request → LLM]
   ↓
[Function Node → Risikoformel]
   ↓
[Switch Node → Entscheidung]
   ↓
[Telegram → Empfehlung]
```

Beispielhafte Switch-Bedingungen:

- Wenn `reversal_risk < 0.4 und trend_strength > 0.7` → Long
- Wenn `reversal_risk > 0.6` → No Trade
- Sonst → Neutraler Statusbericht

Dadurch lernt dein Agent, zwischen „sieht gut aus“ und „besser abwarten“ zu unterscheiden.

---

## 💡 Abschnitt 5 – Fehler und Versionierung

Bevor du mehrere Bedingungen baust:

- Führe jeden Flow einmal mit Testdaten aus.
- Prüfe im Execution-Log, ob alle JSON-Werte vorhanden sind.
- Dokumentiere deine Formeln in deiner Zed-Mappe („Versionskontrolle light“).

Wenn du den n8n-Flow später exportierst (`.json`), bleibt alles reproduzierbar.

---

## 🧭 Abschnitt 6 – Reflexion

- Wo liegt für dich die Grenze zwischen Unterstützung und Autonomie des Agenten?
- Wie würdest du einen Fehlalarm definieren – und wie würdest du ihn im Workflow erkennen?
- Welche Regel in deinem Risikosystem ist nicht verhandelbar?

Diese Fragen sind zentral, wenn du später ein Feedback-System hinzufügst.

---

## 🧩 Abschnitt 7 – Hausaufgabe / Experiment

1. Ergänze deinen Flow aus Kapitel 3 um eine **Function Node**, die eine Positionsgröße berechnet.
2. Füge einen **Switch Node** ein, der nur bei `risk <= 0.02` und `trend_strength > 0.7` eine positive Empfehlung weitergibt.
3. Logge jede Entscheidung mit Datum und Parameterwerten in eine lokale CSV-Datei (`Write Binary File`).

Optional: Baue einen zweiten Telegram-Alert für „No Trade“-Bedingungen.

---

## 🚨 Abschnitt 8 – Berechnung & Logik Debugging

### Division durch Null in Positionsgrößen-Berechnung

**Problem:** `positionSize = (equity * risk) / 0` → NaN/Infinity
**Sichere Berechnung:**

```javascript
const equity = 10000;
const risk = 0.02;
const stopLoss = $json.stop_loss || 50; // Fallback-Wert
const pipValue = 1;

if (stopLoss <= 0) {
  return [{ error: "Invalid stop loss", positionSize: 0 }];
}

const positionSize =
  Math.round(((equity * risk) / (stopLoss * pipValue)) * 100) / 100;
return [{ equity, risk, stopLoss, positionSize }];
```

### Switch-Node Bedingungen greifen nicht

**Problem:** `if trend_strength > 0.7` funktioniert nicht
**Debug-Checklist:**

1. Datentyp prüfen: Ist `trend_strength` string oder number?
2. Feld existiert? Execution-Log → JSON-Structure anschauen
3. Switch-Condition korrekt? `{{$json.trend_strength}} > 0.7`
4. Fallback-Route definieren? (Else-Branch)

**Debug-Function vorschalten:**

```javascript
console.log("Debug Switch Input:", $json);
console.log("trend_strength type:", typeof $json.trend_strength);
console.log("trend_strength value:", $json.trend_strength);
return [$json];
```

### Risiko-Validation Template

Für kritische Berechnungen:

```javascript
const equity = parseFloat($json.equity) || 10000;
const risk = parseFloat($json.risk_percent) || 0.02;
const stopLoss = parseFloat($json.stop_loss) || 0;

// Validierung
if (equity <= 0) return [{ error: "Invalid equity", equity }];
if (risk <= 0 || risk > 0.1) return [{ error: "Risk out of range", risk }];
if (stopLoss <= 0) return [{ error: "Invalid stop loss", stopLoss }];

const riskAmount = equity * risk;
const positionSize = riskAmount / stopLoss;

return [
  {
    equity,
    risk_percent: risk,
    risk_amount: riskAmount,
    stop_loss: stopLoss,
    position_size: Math.round(positionSize * 100) / 100,
    valid: true,
  },
];
```

---

## ✅ Zusammenfassung

Nach Kapitel 4 kannst du:

- quantitative Risikoparameter in n8n abbilden,
- das LLM für Risiko- und Trendbewertung nutzen,
- Entscheidungsbäume modellieren und automatisiert ausführen.

Im nächsten Kapitel geht es darum, wie dein Agent **Gedächtnis und Feedback** erhält – um aus Fehlern zu lernen und seine Entscheidungen über Zeit zu verbessern.
