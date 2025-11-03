# 📘 Kapitel 6 – Parameter-Tuning und A/B-Vergleiche

**Lernziel:**  
Nach dieser Lektion kannst du deinen Agenten so erweitern, dass er **verschiedene Strategiekonfigurationen testet**, Ergebnisse protokolliert und bewertet.  
Du lernst, wie man mit A/B-Vergleichen experimentiert, Parameter automatisch variiert und auf Basis der gespeicherten Daten Performance misst.

---

## 🧩 Abschnitt 1 – Warum Parameter-Tuning wichtig ist

Selbst gute Strategien verlieren, wenn sie starr bleiben.  
Die Märkte verändern sich, Liquidität, Volatilität und Trader-Verhalten ebenfalls.  
Dein Ziel ist es daher nicht, „die perfekte Einstellung“ zu finden, sondern ein **System**, das Anpassung ermöglicht.  

Das Prinzip lautet:  
> *Optimierung durch strukturierte Variation – nicht durch Intuition.*

Mit anderen Worten: du testest Varianten gezielt, beobachtest Ergebnisse, und wählst jene Parameter, die langfristig Stabilität bringen.

---

## ⚙️ Abschnitt 2 – Parameter als variable Inputs

In n8n kannst du Parameter dynamisch übergeben – ideal für systematisches Testen.  

Beispiel:  
Du möchtest herausfinden, ob ein höherer Stop-Loss-Abstand die Winrate verbessert.

**Function-Node (Parameter-Generator):**
```javascript
return [
  { variant: "A", stopLossPoints: 30, riskPercent: 0.02 },
  { variant: "B", stopLossPoints: 50, riskPercent: 0.02 }
];
```

Verbinde diesen Node mit deinem bestehenden **Analyse-Flow** über eine „Split In Batches“-Node, damit jede Variante separat getestet wird.

---

## 🧠 Abschnitt 3 – A/B-Vergleichsstruktur

Ein typischer Testablauf sieht so aus:

```
[Trigger / Backtest-Start]
   ↓
[Function Node → Parameter-Varianten]
   ↓
[Loop über Varianten]
   ↓
[Analyse & Entscheidung (LLM)]
   ↓
[Simulation oder Logging]
   ↓
[Data Store Insert (mit Variant-ID)]
```

In jeder Schleife wird eine Variante getestet.  
Speichere zusätzlich zur normalen Signal-Info ein Feld `variant: "A"` oder `"B"` im Data Store, um die Ergebnisse später trennen zu können.

---

## 🧩 Abschnitt 4 – Automatisierte Auswertung in n8n

Sobald du mehrere Varianten protokolliert hast, kannst du direkt in n8n die Performance vergleichen.

**Data Store Query + Function-Node:**
```javascript
const items = $input.all();
const grouped = {};
for (const i of items) {
  const v = i.json.variant || "unknown";
  grouped[v] = grouped[v] || { total: 0, wins: 0, losses: 0 };
  grouped[v].total++;
  if (i.json.outcome === "TP") grouped[v].wins++;
  if (i.json.outcome === "SL") grouped[v].losses++;
}
const report = Object.keys(grouped).map(v => ({
  variant: v,
  winrate: grouped[v].wins / grouped[v].total,
  total: grouped[v].total
}));
return report;
```

Damit erhältst du sofort die Erfolgsquote pro Variante.  
Das Ergebnis kannst du per Telegram, E-Mail oder Dashboard visualisieren lassen.

---

## 💡 Abschnitt 5 – Parameterwahl durch LLM-Auswertung

Das aktuelle LLM-Modell kann selbst einfache Mustererkennung übernehmen.  
Wenn du willst, dass dein Agent Verbesserungsvorschläge macht, gib ihm die aggregierten Ergebnisse als Input:

```
Analysiere folgende Testergebnisse:
Variante A: Winrate 0.52, Drawdown 3.1 %
Variante B: Winrate 0.64, Drawdown 5.5 %
Welche Variante bietet langfristig das bessere Chance/Risiko-Verhältnis?
```

Der Agent wird nicht „optimieren“, aber qualitativ einschätzen, welche Kombination robuster erscheint.  
Solche Rückmeldungen helfen dir, menschliche und maschinelle Perspektiven zu kombinieren.

---

## ⚠️ Abschnitt 6 – Achte auf statistische Fallstricke

- **Zu kleine Stichprobe:** Unter 30 Trades pro Variante ist fast jede Erkenntnis Zufall.  
- **Survivorship Bias:** Wenn du nur erfolgreiche Phasen analysierst, täuscht Stabilität.  
- **Overfitting:** Je enger du an vergangene Daten anpasst, desto schlechter performt die Strategie live.  
- **Paralleltests:** Führe nie zu viele Varianten gleichzeitig, sonst verwischst du die Ergebnisse.

Erfahrungsgemäß ist *langsames, kontrolliertes Testen* der Schlüssel.

---

## 🧩 Abschnitt 7 – Praxis: Dein erster A/B-Workflow

1. Kopiere deinen Flow aus Kapitel 5.  
2. Füge vor dem LLM-Node einen **Function-Node** hinzu, der zwei Parameter-Sets erzeugt (z. B. Stop-Loss 30 vs. 50).  
3. Füge einen **Split In Batches-Node** ein, damit jede Variante separat läuft.  
4. Ergänze im **Data Store Insert** ein Feld `variant`.  
5. Nach einigen Durchläufen: Query-Node + Function-Node → Aggregation der Winrates.  
6. Sende die Ergebnisse per Telegram.

Optional: Baue einen dritten Flow, der automatisch den besten Parameter anzeigt, sobald > 30 Trades pro Variante erreicht sind.

---

## 🧭 Abschnitt 8 – Reflexion

- Welche Parameter sind bei dir am einflussreichsten?  
- Ab wann würdest du sagen, dass eine Variante „statistisch signifikant besser“ ist?  
- Wie könntest du den Test planbar automatisieren, ohne die Kontrolle zu verlieren?

Solche Überlegungen machen den Unterschied zwischen Experiment und Optimierung.

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

1. Führe einen A/B-Test mit zwei Varianten deines bisherigen Systems durch (z. B. Stop-Loss 30 vs. 50).  
2. Dokumentiere mindestens 10 Signale je Variante.  
3. Erstelle eine Winrate-Statistik in n8n.  
4. Frage das LLM-Modell nach einer qualitativen Einschätzung.  
5. Trage Beobachtungen in deine Kursmappe ein.

---

## ✅ Zusammenfassung

Nach Kapitel 6 kannst du:
- Parameter dynamisch variieren und testen,  
- A/B-Vergleiche in n8n strukturieren,  
- Ergebnisse automatisch auswerten,  
- und dein LLM-System als beratende Instanz einbinden.  

Im nächsten Kapitel lernst du, wie du **mehrere spezialisierte Agentenrollen** kombinierst – Analyst, Risiko-Manager und Strategieberater – um deinen FTMO-Assistenten modular zu machen.