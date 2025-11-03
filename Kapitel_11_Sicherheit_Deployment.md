# 📘 Kapitel 11 – Sicherheit, Zugriffsrechte und Deployment

**Lernziel:**  
Nach dieser Lektion kannst du deinen Agenten sicher betreiben, sensible Daten schützen und Deployment-Strategien für lokale oder Cloud-Umgebungen anwenden.  
Du lernst, wie du API-Keys, Umgebungsvariablen und Zugriffsrechte strukturierst – und wie du Systemausfälle und Leaks vermeidest.

---

## 🧩 Abschnitt 1 – Sicherheitsgrundlagen im Agentenbetrieb

Jede Automatisierung, die mit realen APIs arbeitet, trägt Verantwortung.  
Schon ein falsch platzierter Key oder offener Port kann Daten exponieren oder Handelsbefehle auslösen.  

Merke:  
> „Ein sicherer Agent ist kein schneller Agent – aber ein langlebiger.“

**Ziele der Betriebssicherheit:**
- Minimale Rechte (Prinzip der geringsten Privilegien)  
- Klare Trennung von Entwicklungs- und Produktionsumgebung  
- Keine sensiblen Daten im Code oder in Git  
- Regelmäßige Schlüsselrotation  

---

## ⚙️ Abschnitt 2 – Umgebungsvariablen statt Klartext-Keys

In n8n, Zed oder Docker werden API-Schlüssel als Umgebungsvariablen verwaltet.  

### Beispiel `.env`-Datei:
```
OPENAI_API_KEY=sk-xxxx
TELEGRAM_TOKEN=123456:abcd
N8N_ENCRYPTION_KEY=my_very_secret_key
```

**In n8n verwenden:**  
`${{ $env.OPENAI_API_KEY }}`  

**Debugging-Hinweis:**  
Wenn eine Node meldet „401 Unauthorized“, prüfe:
- ob die Variable korrekt benannt ist,  
- ob n8n nach Änderung neu gestartet wurde,  
- ob das `.env`-File im richtigen Verzeichnis liegt (`/home/node/.n8n` bei Docker).  

---

## 🧠 Abschnitt 3 – Zugriffsrechte und Benutzerrollen

Falls du n8n auf einem Server oder im Team nutzt:

**Empfohlene Rollenstruktur:**
- **Admin:** darf Flows erstellen, API-Keys verwalten  
- **Operator:** darf Flows ausführen, aber nicht verändern  
- **Viewer:** darf nur Logs und Reports sehen  

In n8n Pro oder mit Reverse-Proxy (SaaS-Betrieb) kannst du Benutzerrechte direkt im Dashboard definieren.  

**Debugging-Hinweis:**  
Probiere jeden neuen Benutzer mit Test-Account aus.  
Fehler: „Node not found“ oder „Access Denied“ → Rechte für Ressource (z. B. Data Store) prüfen.

---

## 💡 Abschnitt 4 – Sichere API-Kommunikation

### HTTPS statt HTTP  
Wenn du Webhooks oder externe Calls nutzt, immer über TLS (https://) mit gültigem Zertifikat.  
Kostenlose Zertifikate via Let’s Encrypt:  
```bash
sudo certbot --nginx -d deine-domain.de
```

### Rate Limiting  
Für externe APIs (z. B. OpenAI) setze Pausen zwischen Requests:  
```javascript
await new Promise(r => setTimeout(r, 2000));
```
So vermeidest du Sperrungen bei zu vielen Aufrufen.

### Input-Sanitization  
Vor jedem Prompt- oder JSON-Eintrag prüfen:
```javascript
if ($json.symbol && !$json.symbol.match(/^[A-Z]{3,5}\/[A-Z]{3,5}$/)) {
  throw new Error("Ungültiges Symbolformat");
}
```
Das schützt dich vor ungewollten Eingaben (Prompt-Injection, Code-Injection).

---

## ⚙️ Abschnitt 5 – Deployment-Optionen

### Option A: Lokale Installation  
Ideal für Einzelbetrieb und Tests.  
- Vorteil: volle Kontrolle, keine Daten wandern extern.  
- Nachteil: kein 24/7-Betrieb.

### Option B: Docker Server  
Empfohlen für Dauerbetrieb:  
```bash
docker run -d   --name n8n   -p 5678:5678   -v ~/.n8n:/home/node/.n8n   --env-file .env   n8nio/n8n
```
**Debugging-Hinweis:**  
Fehler „EACCES“ beim Schreiben in `/data` → Rechte auf Volume prüfen (`chmod 777 ~/.n8n` nur Test, produktiv restriktiver).

### Option C: Cloud-VPS oder Docker-Compose  
Für Teams oder dauerhaften Betrieb.  
In `docker-compose.yml` lassen sich Flows, Backups und Logs gemeinsam definieren.  
Empfohlen: tägliche Volume-Backups per cron oder rclone.

---

## 🧩 Abschnitt 6 – Monitoring und Alerts

### Überwachung via n8n-Events  
Jeder Flow-Run kann ein Event auslösen:  
```
[Error Workflow] → [Telegram Alert]
```

### Externes Monitoring  
Tools: Grafana + Prometheus, Healthchecks.io oder Uptime Kuma.  
Beispiel-Check:
```bash
curl -fsS https://mein-agent.de/health || echo "Agent offline"
```

### Debugging-Tipp:  
Wenn du n8n unter Docker betreibst:  
```bash
docker logs n8n --tail 50
```
zeigt die letzten Fehlerausgaben direkt.

---

## 💡 Abschnitt 7 – Backup und Wiederherstellung

### Workflow-Backups  
```bash
n8n export:workflow --backup --output=./backups/
```

### Data-Store-Backups  
Einfacher Ansatz: täglicher Cronjob:  
```bash
tar -czf backups/datastore_$(date +%F).tar.gz ~/.n8n/datastores/
```

**Debugging-Hinweis:**  
Verifiziere Backups regelmäßig!  
Prüfe per `tar -tzf dateiname.tar.gz` ob Archive lesbar sind.  
Viele Systeme scheitern, weil nie ein Restore-Test gemacht wurde.

---

## 🧩 Abschnitt 8 – Sicherheits-Audit-Checkliste

1. [ ] Keine Klartext-API-Keys in Flows  
2. [ ] HTTPS aktiviert und getestet  
3. [ ] n8n auf aktuelle Version aktualisiert  
4. [ ] Regelmäßige Log- und Token-Rotation  
5. [ ] Zugriff auf `.env` und `~/.n8n` auf 600 Rechte gesetzt  
6. [ ] Backups extern gespiegelt  
7. [ ] Error-Workflow aktiv  
8. [ ] Telegram- oder Mail-Alert bei Fehlern  

---

## 🧭 Abschnitt 9 – Reflexion

- Welche Sicherheitslücke wäre bei deinem System am kritischsten?  
- Wie würdest du den Verlust deiner `.env`-Datei kompensieren?  
- Was ist deine persönliche „Downtime-Toleranz“ für diesen Agenten?  

Diese Fragen helfen dir, ein Gefühl für betriebliche Relevanz zu entwickeln – nicht nur technische Sicherheit.

---

## 🧩 Abschnitt 10 – Hausaufgabe / Experiment

1. Richte eine `.env`-Datei ein und ersetze alle Keys im Flow.  
2. Starte deinen Agenten in Docker und teste Verbindung zu Telegram und OpenAI.  
3. Simuliere einen Systemausfall (n8n stop → start) und prüfe, ob alles korrekt wiederhergestellt wird.  
4. Führe deine eigene Sicherheits-Checkliste schriftlich durch.  
5. Optional: richte Healthchecks.io oder Uptime Kuma zur Statusüberwachung ein.

---

## ✅ Zusammenfassung

Nach Kapitel 11 kannst du:
- API-Schlüssel und Umgebungen sicher verwalten,  
- Benutzer- und Zugriffsrechte einrichten,  
- deine Agentenumgebung produktionsreif deployen,  
- Backups und Monitoring automatisieren,  
- und Systemfehler sicher diagnostizieren.  

Im nächsten Kapitel (12) wirst du lernen, **wie du das gesamte System evaluierst, skalierst und ggf. mit externen Datenfeeds (TradingView, News, Sentiment-APIs)** kombinierst – um deinen FTMO-Agenten wirklich marktfähig zu machen.