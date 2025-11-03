# Kapitel 1 – Fundament: Denken wie ein KI‑Agenten‑Architekt

**Lernziel:** Du verstehst, was ein „Agent“ im Kontext moderner KI‑Systeme ist, wie sich Sprachmodelle konzeptionell von klassischen Trading‑Bots unterscheiden und wie du daraus ein steuerbares System aufbauen kannst.

## 1.1 Was ist ein Agent?
Ein Agent ist ein System, das auf Basis von Wahrnehmung, Zustand und Ziel entscheidet, welche Aktion es ausführt. Er reagiert auf eine Umgebung (Input) mit einer Handlung (Output), um ein definiertes Ziel zu erreichen.

Grundschema: Sensoren → Zustand → Entscheidung → Aktion → Feedback

Trading‑Bezug: Sensoren = Marktdaten; Zustand = Kontostand, Regeln; Aktion = Trade‑Vorschlag; Ziel = Kapitalerhalt/Drawdown vermeiden. Das aktuelle LLM‑Modell übernimmt den Entscheidungsteil in natürlicher Sprache.

## 1.2 Warum LLM‑Agenten keine Trading‑Bots sind
Das Modell kann kontextualisieren und in natürlicher Sprache argumentieren, handelt aber nicht autonom. Es erstellt Bewertungen und Risikoabschätzungen, die du prüfst. Entscheidungshoheit bleibt beim Menschen.

## 1.3 Minimaler Entscheidungs‑Loop
```
[Webhook Trigger] → [Get Market Data] → [LLM‑Analyse] → [Risk Calculator] → [Send Telegram Msg]
```

## 1.4 Rationalität unter Unsicherheit
Ein intelligenter Agent behandelt Unsicherheit sinnvoll: Wahrscheinlichkeiten statt Gewissheiten.

## 1.5 Praxis
Beispiel‑Prompt:
```
Du bist ein Trading‑Analyse‑Agent.
Du erhältst OHLC‑Daten für BTC/USD.
Beschreibe Trend, Momentum und mögliche Entry‑Zonen.
Maximal 2 % Risiko.
```

## 1.6 Reflexion
– Unterschied deterministischer Code vs. LLM‑Analyse?  
– Risiken bei zu großer Entscheidungshoheit für das Modell?

## 1.7 Aufgabe
Erstelle in n8n: Webhook‑Trigger → HTTP‑Request (LLM‑API) → Telegram‑Node. Sende per Webhook einen Mini‑Prompt und erhalte die Antwort zurück.

---

# Kapitel 2 – n8n verstehen und vorbereiten

**Lernziel:** Du kannst n8n installieren, Workflows erstellen und den Datenfluss zwischen Nodes verstehen.

## 2.1 Was ist n8n
Open‑Source‑Workflow‑Automatisierer mit voller Kontrolle über Variablen, APIs und Logik.

## 2.2 Installation
Desktop: https://n8n.io/download → Start → Browser unter http://localhost:5678  
Docker:
```bash
docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

## 2.3 Oberfläche
Canvas = Workflow, Sidebar = Nodes, Execution‑Panel = Debug. Datenflüsse sind JSON‑Objekte.

## 2.4 Erster Workflow
Cron → Function → Telegram. Function‑Code:
```javascript
return [{ message: "n8n läuft stabil und wartet auf Befehle." }];
```

## 2.5 Datenfluss
Jeder Node: Input → Verarbeitung → Output. Das Debug‑Panel zeigt die JSONs.

## 2.6 n8n spricht JSON
Beispiel:
```json
{ "symbol": "BTCUSD", "price": 68235.4 }
```

## 2.7 Praxis: Ping an das LLM
Webhook → HTTP‑Request POST https://api.openai.com/v1/chat/completions  
Beispiel‑Body (Schemavorlage – ersetze das Modell später durch einen realen Modellnamen in deiner Umgebung):
```json
{
  "model": "aktuelles LLM-Modell",
  "messages": [
    {"role": "system", "content": "Du bist ein Trading-Assistent."},
    {"role": "user", "content": "Wie ist die aktuelle Stimmung zu BTC/USD?"}
  ]
}
```

## 2.8 Aufgabe
Erweitere den Workflow: Function‑Node mit Zeitstempel, Email‑Node für Versand, Write‑File für Logging.

---

# Kapitel 3 – LLM als Datenanalyst und Denkmodul

**Lernziel:** Du nutzt das Modell als Datenanalyst, der strukturierte JSON‑Ausgaben liefert.

## 3.1 Konzept
LLMs sind Text‑zu‑Text‑Transformer: Sie erkennen Muster, rechnen aber nicht effizient. Im Trading dienen sie als Analyst/Kommentator/Strukturierer.

## 3.2 Technischer Anschluss
OpenAI‑Node → Operation „Chat“ → Modell: aktuelles LLM‑Modell. Systemnachricht:
```
Du bist ein professioneller Trading‑Analyst. Antworte strukturiert im JSON‑Format.
```

## 3.3 Prompt‑Design
Kontext → Instruktion → Format. Fordere JSON‑Ausgabe, z. B.:
```
Analysiere die Kursdaten in data.
Gib trend, momentum, long_probability, short_probability, comment als JSON zurück.
```

## 3.4 Theorie
LLMs liefern plausible Bewertungen (Wahrscheinlichkeiten), keine kausalen Garantien.

## 3.5 Praxis
Input‑Function (OHLC‑Daten) → LLM‑Node → Telegram. Beispiel‑JSON:
```json
{"trend":"bullish","momentum":"steigend","long_probability":0.78,"short_probability":0.22}
```

## 3.6 Aufgabe
Hole Live‑Daten (z. B. Alpha Vantage), lasse analysieren und vergleiche die Einschätzung nach 4 Stunden mit dem realen Verlauf. Protokolliere Treffergenauigkeit.

---

# Kapitel 4 – Risikomanagement automatisieren

**Lernziel:** Du modellierst Positionsgrößen und Risiko‑Berechnungen in n8n und verbindest sie mit LLM‑Analysen.

## 4.1 Konzept
Kennzahlen: Risiko‑Prozent, Stop‑Loss‑Distanz, Positionsgröße. Formel:
```
Positionsgröße = (Kontogröße × Risiko%) / Stop‑Loss‑Distanz
```

## 4.2 Function‑Node Beispiel
```javascript
const account = 10000;
const risk = 0.02;
const sl = 50;
const pip = 1;
return [{
  balance: account,
  risk_amount: account * risk,
  stop_loss_pips: sl,
  position_size: (account * risk) / (sl * pip),
  risk_percent: risk * 100
}];
```

## 4.3 LLM als Risiko‑Berater
Prompt:
```
Analysiere Markt- und Risiko-Daten.
Empfiehl Positionsgröße, Stop‑Loss und gib einen Risikokommentar.
Antwort im JSON-Format: recommended_size, stop_loss, risk_category, comment.
```

## 4.4 Theorie
Risk (quantifizierbar) vs. Uncertainty (qualitativ). Prompt z. B.: „Bewerte Unsicherheiten hoch/mittel/niedrig.“

## 4.5 Praxis
LLM‑Analyse → Risiko‑Berechnung → LLM‑Bewertung → Telegram‑Report.

## 4.6 Aufgabe
Regel: Nur wenn long_probability > 0.65 wird eine Position empfohlen. Logge abgelehnte Signale samt Begründung. Berechne Durchschnittsrisiko nach 10 Signalen.
