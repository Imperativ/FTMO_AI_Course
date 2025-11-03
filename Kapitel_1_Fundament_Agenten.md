# 📘 Kapitel 1 – Fundament: Denken wie ein KI-Agenten-Architekt

**Lernziel:**
Nach dieser Lektion verstehst du, was ein „Agent“ im Kontext moderner KI-Systeme ist, wie sich ChatGPT-Agenten konzeptionell unterscheiden von klassischen Trading-Bots – und wie du dieses Wissen nutzt, um ein eigenes, steuerbares System aufzubauen.

---

## 🧩 Abschnitt 1 – Was ist ein Agent?

Ein **Agent** ist kein Wesen mit „Intelligenz“ im menschlichen Sinn, sondern ein **System, das auf Basis von Wahrnehmung, Zustand und Ziel entscheidet, welche Aktion es ausführt**.
Er reagiert also auf eine Umgebung (Umweltzustand) mit einer Handlung (Aktion), die ein Ziel verfolgt (Reward oder Erfolgskriterium).

Ein klassisches Agenten-Schema besteht aus:

- **Sensoren / Input:** Was der Agent wahrnimmt (z. B. Marktdaten, Zeit, Kontostand).
- **Zustand / Speicher:** Was er über die Welt weiß oder erinnert.
- **Aktoren / Output:** Was er tun kann (z. B. ein Signal senden, eine Order vorschlagen).
- **Ziel / Policy:** Wie er entscheidet, was „gut“ ist (z. B. Profit maximieren, Drawdown vermeiden).

In der Sprache von LLM-basierten Agenten ersetzt das Modell den **Entscheidungsteil** durch natürliche Sprache.
Statt Entscheidungsbäume in Code zu gießen, beschreibst du Regeln in Textform:

> „Analysiere die letzten 12 Stunden BTC/USD, erkenne Trendrichtung, berechne RSI und schlage Long oder Short vor – aber nur, wenn der Drawdown unter 2 % bleibt.“

Damit übernimmst du das Denken – die Logik bleibt aber algorithmisch steuerbar.

---

## 💬 Abschnitt 2 – Warum ChatGPT-Agenten keine „Trading-Bots“ sind

Ein ChatGPT-Agent kann **kontextualisieren**, **lernen aus Feedback** und **natürliche Sprache als Schnittstelle** nutzen.
Er kann aber **nicht autonom handeln** – und das ist auch gut so.
Für FTMO-Zwecke willst du einen _Assistenzagenten_:

- Er bewertet Marktsituationen.
- Erstellt Handlungsempfehlungen.
- Gibt Risikoabschätzungen.
- Und meldet sie über einen Workflow (z. B. Telegram) an dich.

Die Entscheidung bleibt bei dir – so umgehst du regulatorische Probleme und trainierst zugleich, die KI als Partner zu nutzen.

---

## ⚙️ Abschnitt 3 – Der minimale Entscheidungs-Loop

In n8n kannst du diesen Loop später als Workflow darstellen:

```
[Webhook Trigger]
   ↓
[Get Market Data]  ← Sensor
   ↓
[LLM Node]         ← Entscheidung
   ↓
[Risk Calculator]  ← Zustand / Policy
   ↓
[Send Telegram Msg]← Aktion
```

Dieser Ablauf ist die DNA deines künftigen Agenten.
Später fügen wir Persistenz (Speicher) und Selbstbewertung (Feedback-Loop) hinzu.

---

## 🧠 Abschnitt 4 – Theoretische Brücke: Rationalität und Unsicherheit

Künstliche Agenten operieren unter **Unsicherheit** – sie kennen die Zukunft nicht, sie schätzen sie.
Darum gilt: _Ein intelligenter Agent ist einer, der seine Unsicherheit sinnvoll behandelt._
Das ist im Trading entscheidend.
Wir werden LLMs so nutzen, dass sie Wahrscheinlichkeiten und Risikoqualitäten beschreiben, nicht absolute Wahrheiten.
Das Ergebnis ist kein „Signal“ im Sinne eines Dogmas, sondern eine _strukturierte Wahrscheinlichkeit_.

---

## 🧩 Abschnitt 5 – Mini-Praxis: Dein erster Agent in Gedanken

1. Starte das aktuelle LLM-Modell über die API oder die Oberfläche.
2. Gib folgenden Prompt ein:

```
Du bist ein Trading-Analyse-Agent.
Du erhältst Marktdaten (Open, High, Low, Close, Volume) für BTC/USD.
Analysiere sie, nenne Trendrichtung, Momentum und mögliche Entry-Zonen.
Verwende dabei ein Risikomanagement mit maximal 2 % Verlust pro Trade.
```

3. Lies die Antwort wie einen Bericht eines Assistenten – nicht als Order.
4. Beobachte: Erklärt der Agent seine Unsicherheit? Nutzt er Prozentangaben?
   Wenn nicht, ändere den Prompt und fordere explizit „Schätze Wahrscheinlichkeiten“.

---

## 🧭 Abschnitt 6 – Reflexion

- Wo liegt der Unterschied zwischen einem deterministischen Programm und einem LLM-Agenten?
- Welche Risiken entstehen, wenn man einem Sprachmodell zu viel Entscheidungshoheit gibt?
- Wie würdest du den Agenten „vorsichtiger“ machen, ohne seine Reaktionsfähigkeit zu zerstören?

---

## 🧩 Abschnitt 7 – Hausaufgabe / Experiment

Erstelle in n8n einen einfachen Workflow mit einem **Webhook Trigger** und einem **HTTP Request Node**.
Schick über den Webhook einen JSON-Payload mit einem Mini-Prompt (z. B. „Was denkst du über EUR/USD heute?").
Lass dir die Antwort per **Telegram Node** an dich senden.

Du hast damit deinen ersten funktionierenden Kommunikations-Loop gebaut:
Du (Mensch) → n8n (System) → LLM (Denker) → Telegram (Antwortkanal).

---

## 🛠️ Abschnitt 8 – Universelle Debug-Strategien für alle Kapitel

### JSON-Explorer Function

Wenn du nicht weißt, welche Daten in einem Node ankommen:

```javascript
// Zeige komplette Datenstruktur
console.log("=== DEBUG START ===");
console.log("Input Items Count:", $input.all().length);
console.log("First Item:", JSON.stringify($input.first(), null, 2));
console.log("All JSON keys:", Object.keys($json || {}));
console.log("=== DEBUG END ===");
return [$json];
```

### Error-Handling Template

Für kritische Nodes:

```javascript
try {
  // Deine Logik hier
  const result = doSomething($json);
  return [{ success: true, data: result }];
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

### Workflow-Monitoring Checkliste

- Execution-History regelmäßig prüfen (n8n → Executions)
- Failed Executions analysieren: Logs → Error Details
- Performance: Lange Laufzeiten identifizieren
- Telegram-Alerts bei kritischen Fehlern einbauen

---

## ✅ Zusammenfassung

Nach Kapitel 1 weißt du:

- was ein Agent konzeptionell ist,
- wo LLM-basierte Agenten sich unterscheiden,
- wie ein n8n-Workflow als Denk-Loop funktioniert,
- und welche Debug-Strategien dir bei Problemen helfen.

In Kapitel 2 werden wir diesen Loop konkretisieren und **n8n praktisch einrichten** – mit deinem ersten lauffähigen Flow, der Daten verarbeitet statt nur Text.
