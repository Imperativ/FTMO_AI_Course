# FTMO-AI-Course Roadmap

## Übersicht
Ein modular aufgebauter Lernkurs, um mit ChatGPT-Agenten, n8n und TradingView einen halbautomatischen FTMO-Trading-Assistenten zu entwickeln.  
Dauer: ca. 6 Kapitel à 20–30 Minuten (Gesamt ca. 5 Stunden).

---

## Kapitelübersicht

### Kapitel 1 – Denken wie ein Agenten-Architekt
- Konzept: Agent, Ziel, Zustand, Aktion
- Unterschied GPT-Agent vs. Trading-Bot
- Mini-Praxis: erster ChatGPT-Analyse-Prompt
- **Ergebnis:** Verständnis der Entscheidungslogik

### Kapitel 2 – n8n verstehen und vorbereiten
- Oberfläche, Nodes, Debugging
- Cron-Trigger, Telegram, Webhook
- GPT-API über HTTP Node
- **Ergebnis:** lauffähiger GPT-Webhook

### Kapitel 3 – GPT als Datenanalyst mit TradingView
- TradingView Webhooks an n8n
- GPT analysiert Echtzeitdaten
- Telegram-Ausgabe der Analyse
- **Ergebnis:** automatischer Marktkommentar

### Kapitel 4 – Risikomanagement und Entscheidungslogik
- FTMO-Parameter und EV-Berechnung
- Risk/Reward-Kalkulation
- GPT-Risikobewertung
- **Ergebnis:** quantifizierte Entscheidungsgrundlage

### Kapitel 5 – Agenten mit Gedächtnis
- Datastore oder SQLite
- Rückkopplung und tägliches Feedback
- GPT als Selbstbewerter
- **Ergebnis:** lernfähiger Agent mit Verlaufsspeicher

### Kapitel 6 – Multi-Agent-System & Abschlussprojekt
- Analyst-, Risiko- und Filter-Agent
- Telegram-Reporting
- **Ergebnis:** vollständiger halbautomatischer FTMO-Assistent

---

## Lernphasen
1. **Verstehen (Kap. 1–2)** – Architektur und Tools
2. **Bauen (Kap. 3–4)** – Datenfluss und Risiko
3. **Reflektieren (Kap. 5)** – Gedächtnis und Feedback
4. **Orchestrieren (Kap. 6)** – Multi-Agent-System

---

## Praktisches Ziel
Ein n8n-Workflow, der:
1. TradingView-Signale empfängt  
2. GPT-Analysen generiert  
3. Risiko kalkuliert  
4. Entscheidungen filtert  
5. Telegram-Berichte versendet

---

## Systemanforderungen
- n8n v1.51+ (Desktop oder Docker)
- OpenAI API-Key (gpt-4o oder Nachfolger)
- TradingView-Konto mit Webhook-Funktion
- Telegram-Bot-Token
- Optional: SQLite oder Supabase für Memory

---

## Lizenz & Autor
Erstellt für Lernzwecke, nicht als Finanzberatung.  
© 2025 Bernhard Reichert, mit Unterstützung von ChatGPT-5.
