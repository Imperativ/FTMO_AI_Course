# 📘 Kapitel 2 – n8n verstehen und vorbereiten

**Lernziel:**
Nach dieser Lektion kannst du n8n selbstständig bedienen, einfache Workflows erstellen und verstehen, wie Daten durch Nodes fließen. Du lernst, n8n als die „Verkabelungsebene“ deines künftigen Trading-Agenten zu sehen.

---

## 🧩 Abschnitt 1 – Was n8n eigentlich ist

n8n ist ein **Workflow-Automatisierungssystem**, das Daten zwischen Diensten verschiebt und dabei eigene Logik ausführt.
Stell es dir wie ein **digitales Mischpult** vor: Jeder Node ist ein Regler oder Schalter, der Eingaben verändert, kombiniert oder weiterleitet.

Im Gegensatz zu „no-code“-Tools wie Zapier ist n8n **open source**, kann lokal laufen und erlaubt **volle Kontrolle über Logik, Variablen und API-Zugriffe**.
Für unsere Zwecke ist das entscheidend: Wir können damit nicht nur APIs ansprechen, sondern auch LLM-Prompts verarbeiten, Ergebnisse speichern und mit Risiko-Parametern anreichern.

---

## ⚙️ Abschnitt 2 – Installation und Start

**Option A: Desktop-App (empfohlen für Einstieg)**

1. Lade die aktuelle Version unter [https://n8n.io/download](https://n8n.io/download) herunter.
2. Starte die Anwendung, wähle einen Arbeitsordner (z. B. `FTMO-Agent/n8n-workspace/`).
3. Nach dem Start erreichst du die Oberfläche im Browser unter
   ```
   http://localhost:5678
   ```

**Option B: Docker (für fortgeschrittene Nutzung)**
Wenn du n8n dauerhaft auf einem Server laufen lassen willst:

```bash
docker run -it --rm   -p 5678:5678   -v ~/.n8n:/home/node/.n8n   n8nio/n8n
```

Beide Varianten sind funktional identisch; Docker eignet sich später, wenn du den Agenten dauerhaft laufen lassen willst.

---

## 💡 Abschnitt 3 – Der Aufbau der Oberfläche

Sobald n8n läuft, siehst du drei Hauptbereiche:

1. **Node-Canvas:** Hier verbindest du einzelne Funktionsblöcke (Nodes) miteinander.
2. **Sidebar:** Liste aller verfügbaren Nodes – sortiert nach Kategorie (Trigger, Webhook, HTTP Request, Function usw.).
3. **Execution-Panel:** Zeigt, welche Daten ein Node empfangen und ausgegeben hat – dein wichtigstes Debug-Werkzeug.

Ein Workflow ist immer eine **gerichtete Kette von Nodes**, in der jede Stufe Daten weiterreicht.

---

## 🧩 Abschnitt 4 – Dein erster Workflow

Wir beginnen mit einem kleinen Test:
Ein Workflow, der jede Stunde eine kurze Nachricht an dich sendet.

1. Erstelle einen neuen Workflow.
2. Ziehe einen **Cron-Node** auf die Fläche.
   - Einstellung: „Every hour“.
3. Füge einen **Function-Node** hinzu.
   - Code:
     ```javascript
     return [{ message: "n8n läuft stabil und wartet auf Befehle." }];
     ```
4. Füge einen **Telegram Node** hinzu (unter „Messaging“).
   - Trage deinen Bot-Token und deine Chat-ID ein.
   - Verwende `{{$json["message"]}}` als Nachrichtentext.
5. Verbinde die Nodes: **Cron → Function → Telegram**.
6. Speichere und aktiviere den Workflow.

Ergebnis: Dein Telegram-Bot meldet sich stündlich mit einem Statussignal.
Das ist dein „Herzschlag“ – ein kleiner, aber sehr nützlicher Testlauf, bevor du komplexe Flows baust.

---

## 🔧 Abschnitt 5 – Datenfluss und Debugging

Klicke nach einer Ausführung auf den **Function-Node** und schau dir im rechten Panel die JSON-Daten an.
Das Prinzip ist immer gleich:

- Eingabe (Input) → Verarbeitung → Ausgabe (Output).
- Jede Ausgabe wird automatisch an den nächsten Node übergeben.

Wenn du den Output eines Nodes inspizieren kannst, verstehst du praktisch jeden n8n-Workflow.
Die Fähigkeit, Flows zu _lesen_, ist fast wichtiger als sie zu _bauen_.

---

## 🧠 Abschnitt 6 – Theoretischer Unterbau: Daten als Sprache

n8n spricht JSON.
Jeder Node erzeugt, verändert oder liest **Datenobjekte**, die so aussehen:

```json
{ "symbol": "BTCUSD", "price": 68235.4, "volume": 1943 }
```

LLMs verstehen ebenfalls JSON, wenn du es korrekt formst.
Damit bilden n8n (Logik-Layer) und LLM (Analyse-Layer) eine **gemeinsame Sprache**.
Später wirst du z. B. Marktdaten als JSON einspeisen, das LLM analysiert sie, und das Ergebnis läuft wieder als JSON zurück – etwa:

```json
{ "trend": "bullish", "confidence": 0.73, "recommendation": "long" }
```

---

## 🧩 Abschnitt 7 – Mini-Praxis: „Ping den GPT-Agenten“

1. Erstelle einen neuen Workflow.
2. Node 1: **Webhook** – Methode `POST`.
3. Node 2: **HTTP Request** – Ziel `https://api.openai.com/v1/chat/completions`.
4. Authentifiziere dich mit deinem OpenAI-Key.
5. Sende folgenden Body:
   ```json
   {
     "model": "aktuelles LLM Modell",
     "messages": [
       { "role": "system", "content": "Du bist ein Trading-Assistent." },
       {
         "role": "user",
         "content": "Wie ist die aktuelle Stimmung zu BTC/USD?"
       }
     ]
   }
   ```
6. Ausgabe prüfen → JSON-Pfad `choices[0].message.content` → Telegram-Node → dir selbst senden.

Damit hast du **deinen ersten LLM-Aufruf über n8n** realisiert.
Es ist der Grundbaustein aller kommenden Agenten.

---

## 🧭 Abschnitt 8 – Reflexion

- Wie unterscheidet sich n8n von einer klassischen Programmiersprache?
- Welche Vorteile bringt der visuelle Aufbau gegenüber Code?
- Wie würdest du Fehlermeldungen oder API-Timeouts im Workflow abfangen?

Notiere Antworten in deiner Zed-Mappe, sie helfen später bei der Fehlerbehandlung.

---

## 🧩 Abschnitt 9 – Hausaufgabe / Experiment

Erweitere deinen Ping-Workflow:

1. Ergänze eine **Function-Node**, die dem LLM-Output einen Zeitstempel anhängt.
2. Sende diesen Output zusätzlich per **Email-Node** an dich.
3. Logge die Ausgabe in eine Datei auf deinem System (`Write Binary File`-Node).

Ziel: Du dokumentierst jede Kommunikation deines Agenten automatisch – ein Schritt Richtung Nachvollziehbarkeit und Evaluierung.

---

## 🚨 Abschnitt 9 – Debugging-Hinweise für n8n

### Node-Verbindungen funktionieren nicht

**Problem:** Nodes sind verbunden, aber Daten kommen nicht an
**Lösung:**

- Klicke auf "Test Step" bei jedem Node einzeln
- Prüfe im Debug-Panel: Sind die JSON-Felder korrekt benannt?
- Tipp: `{{$json["feldname"]}}` statt `{{$json.feldname}}` bei Sonderzeichen

### Webhook gibt 404 zurück

**Problem:** `POST http://localhost:5678/webhook/test` → 404
**Lösung:**

- n8n Workflow muss **aktiviert** sein (Toggle oben rechts)
- Webhook-URL kopieren (Button im Webhook-Node)
- Test mit curl:
  ```bash
  curl -X POST -H "Content-Type: application/json" -d '{"test":"data"}' [WEBHOOK-URL]
  ```

### "Variable not found" Fehler

**Problem:** `{{$json.symbol}}` führt zu Fehler
**Debug-Strategie:**

1. Execution-Log öffnen → Node anklicken → JSON-Output prüfen
2. Function-Node vorschalten:
   ```javascript
   return [$input.all()];
   ```
   Zeigt komplette Datenstruktur
3. Korrekte Referenz verwenden: `{{$json["symbol"]}}` oder `{{$items("NodeName")[0].json.symbol}}`

### JSON-Explorer für unbekannte Datenstrukturen

Wenn du nicht weißt, welche Daten ankommen:

```javascript
console.log("=== DEBUG START ===");
console.log("Input Items Count:", $input.all().length);
console.log("First Item:", JSON.stringify($input.first(), null, 2));
console.log("All JSON keys:", Object.keys($json));
console.log("=== DEBUG END ===");
return [$json];
```

---

## ✅ Zusammenfassung

Nach Kapitel 2 kannst du:

- n8n starten, bedienen und Workflows debuggen,
- Telegram- und OpenAI-Nodes nutzen,
- und hast deinen ersten LLM-Agenten über einen Webhook getriggert.

Im nächsten Kapitel wirst du lernen, **LLMs gezielt als Datenanalysten** einzusetzen: Marktdaten abrufen, strukturieren und bewerten lassen.
