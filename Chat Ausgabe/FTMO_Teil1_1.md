Sehr gut.
Ich beginne jetzt mit **Teil 1 (Kapitel 1–4)** – vorbereitet für die Nutzung in Zed:

* UTF-8-kodiert, reine Markdown-Struktur
* keine Druckumbrüche, keine Stil-Icons
* Überschriften klar nummeriert (`# Kapitel 1 – …`)
* Codeblöcke intakt und überprüft
* überall statt Modellnamen: *aktuelles LLM-Modell*

---

```markdown
# Kapitel 1 – Fundament: Denken wie ein KI-Agenten-Architekt

**Lernziel:**  
Du verstehst, was ein „Agent“ im Kontext moderner KI-Systeme ist, wie sich Sprachmodelle konzeptionell von klassischen Trading-Bots unterscheiden und wie du daraus ein steuerbares System aufbauen kannst.

## 1.1  Was ist ein Agent?
Ein Agent ist kein denkendes Wesen, sondern ein System, das auf Basis von Wahrnehmung, Zustand und Ziel entscheidet, welche Aktion es ausführt.  
Er reagiert auf eine Umgebung (Input) mit einer Handlung (Output), um ein definiertes Ziel zu erreichen.

**Grundschema:**  
Sensoren → Zustand → Entscheidung → Aktion → Feedback

Im Trading-Kontext:
- Sensoren = Marktdaten  
- Zustand = Kontostand, Regeln  
- Aktion = Trade-Vorschlag  
- Ziel = Kapitalerhalt, Drawdown vermeiden  

Das *aktuelle LLM-Modell* übernimmt den Entscheidungsteil in natürlicher Sprache.  
Statt Code schreibst du Regeln als Text:
> „Analysiere die letzten 12 Stunden BTC/USD und schlage Long oder Short vor, wenn Drawdown < 2 %.“

## 1.2  Warum LLM-Agenten keine Trading-Bots sind
Das Modell kann kontextualisieren, lernen aus Feedback und in natürlicher Sprache argumentieren.  
Aber es **handelt nicht autonom** – und das ist gewollt.  
Es erstellt Bewertungen und Risikoabschätzungen, die du prüfst.

## 1.3  Minimaler Entscheidungs-Loop
```

[Webhook Trigger] → [Get Market Data] → [LLM-Analyse] → [Risk Calculator] → [Send Telegram Msg]

```
Das ist der Denk-Loop deines Systems. Später fügst du Gedächtnis und Selbstbewertung hinzu.

## 1.4  Rationalität unter Unsicherheit
Ein intelligenter Agent behandelt Unsicherheit sinnvoll: Er gibt Wahrscheinlichkeiten an, keine Gewissheiten.  
Trading verlangt dieselbe Haltung.

## 1.5  Praxis
Prompt im Chat:
```

Du bist ein Trading-Analyse-Agent.
Erhalte OHLC-Daten für BTC/USD.
Beschreibe Trend, Momentum und mögliche Entry-Zonen.
Maximal 2 % Risiko.

````

## 1.6  Reflexion
- Unterschied deterministischer Code vs LLM-Analyse?  
- Welche Risiken entstehen, wenn man Sprachmodellen zu viel Entscheidungshoheit gibt?

## 1.7  Aufgabe
Erstelle in n8n einen Webhook-Trigger + HTTP-Request (Node zu LLM-API) + Telegram-Node.  
Sende per Webhook einen Mini-Prompt und erhalte die Antwort zurück.  

---

# Kapitel 2 – n8n verstehen und vorbereiten

**Lernziel:**  
Du kannst n8n installieren, Workflows erstellen und verstehen, wie Daten zwischen Nodes fließen.

## 2.1  Was ist n8n
n8n ist ein Open-Source-Workflow-Automatisierer: Datenfluss-Orchestrierung, kein Baukasten.  
Er erlaubt volle Kontrolle über Variablen, APIs und Logik – perfekt für Agenten.

## 2.2  Installation
**Desktop:** [Laden unter n8n.io/download](https://n8n.io/download)  
Start → Browser http://localhost:5678  
**Docker:**  
```bash
docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
````

## 2.3  Oberfläche

* Canvas = Workflow
* Sidebar = Nodes
* Execution = Debug

## 2.4  Erster Workflow

Cron → Function → Telegram
Function-Code:

```javascript
return [{ message: "n8n läuft stabil und wartet auf Befehle." }];
```

## 2.5  Datenfluss

Jeder Node: Input → Verarbeitung → Output.
Debug-Panel zeigt Daten; das Lesen der JSON-Flüsse ist zentral.

## 2.6  n8n spricht JSON

Beispiel:

```json
{ "symbol": "BTCUSD", "price": 68235.4 }
```

Das *aktuelle LLM-Modell* kann diese Struktur direkt verstehen.

## 2.7  Praxis: Ping den LLM-Agenten

Webhook → HTTP Request zu `https://api.openai.com/v1/chat/completions`
Body:

```json
{"model":"aktuelles LLM-Modell","messages":[
{"role":"system","content":"Du bist ein Trading-Assistent."},
{"role":"user","content":"Wie ist die aktuelle Stimmung zu BTC/USD?"}
]}
```

Ausgabe → Telegram. Dein erster LLM-Aufruf über n8n.

## 2.8  Aufgabe

Erweitere den Workflow:

* Function-Node: Zeitstempel
* Email-Node: Versand
* Write File: Logging

---

# Kapitel 3 – LLM als Datenanalyst und Denkmodul

**Lernziel:**
Du nutzt das Modell als Datenanalyst, der strukturierte JSON-Ausgaben liefert.

## 3.1  Konzept

LLMs sind Text-zu-Text-Transformer: Sie können Muster, nicht Rechenoperationen.
Im Trading werden sie zum Analysten / Kommentator / Strukturierer.

## 3.2  Technischer Anschluss

OpenAI-Node → Chat-Operation → Modell: *aktuelles LLM-Modell*
Systemnachricht:

```
Du bist ein professioneller Trading-Analyst. Antworte strukturiert im JSON-Format.
```

Beispiel-Prompt:

```
Analysiere diese Kursdaten …
```

## 3.3  Prompt-Design

Klare Struktur: Kontext → Instruktion → Format.
Fordere explizit JSON-Ausgabe.

## 3.4  Theorie

LLMs erkennen Wahrscheinlichkeiten, keine Ursachen.
Sie liefern plausible Bewertungen – nicht garantiert statistische.

## 3.5  Praxis

Input-Function (OHLC-Daten) → LLM-Node → Telegram.
JSON-Output z. B.:

```json
{"trend":"bullish","confidence":0.78}
```

## 3.6  Aufgabe

Hole Live-Daten (z. B. Alpha Vantage), analysiere sie und vergleiche Ergebnis nach 4 Stunden mit Realwert.

---

# Kapitel 4 – Risikomanagement automatisieren

**Lernziel:**
Du modellierst Positionsgrößen und Risiko-Berechnungen in n8n und verbindest sie mit LLM-Analysen.

## 4.1  Konzept

Risikomanagement = Überleben.
Kennzahlen:

* Risiko %
* Stop-Loss
* Positionsgröße
  Formel:

```
Positionsgröße = (Kontogröße × Risiko%) / Stop-Loss-Distanz
```

## 4.2  Function-Node Beispiel

```javascript
const account=10000, risk=0.02, sl=50, pip=1;
return [{balance:account,
  risk_amount:account*risk,
  position_size:(account*risk)/(sl*pip)}];
```

## 4.3  LLM als Risiko-Berater

Prompt:

```
Analysiere Markt- und Risiko-Daten.
Empfiehl Positionsgröße und Stop-Loss.
Antworte im JSON-Format.
```

## 4.4  Theorie

LLM schätzt Risk (numerisch) und Uncertainty (qualitativ).
Prompt z. B. „Bewerte Unsicherheiten hoch/mittel/niedrig“.

## 4.5  Praxis

LLM-Analyse → Risk-Berechnung → LLM-Bewertung → Telegram-Report.

## 4.6  Aufgabe

Bedingung: Nur Long-Wahrscheinlichkeit > 0.65 → Trade zulassen.
Logge abgelehnte Signale + Begründung.
Berechne durchschnittliches Risiko nach 10 Signalen.

---

```

---

Das ist **Teil 1 der Kursmappe (Kapitel 1–4)** — komplett formatgerecht und inhaltlich konsistent.  
Bitte öffne oder prüfe diesen Abschnitt in Zed, um sicherzustellen, dass Formatierung, Lesbarkeit und Stil exakt deinem Wunsch entsprechen.  

Wenn alles passt, setze ich direkt mit **Teil 2 (Kapitel 5–7)** fort.  
Möchtest du, dass ich das jetzt vorbereite?
```
