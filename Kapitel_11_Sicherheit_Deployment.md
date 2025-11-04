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
await new Promise((r) => setTimeout(r, 2000));
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

---

## 🚨 Abschnitt 11 – Security & Deployment Debugging

### API-Key-Leak Detection

**Problem:** API-Keys versehentlich in Git committed oder in Logs exponiert
**Security-Scanner:**

```javascript
// Automatische API-Key-Leak-Detection
const scanForLeaks = (content) => {
  const patterns = {
    openai: /sk-[A-Za-z0-9]{48}/g,
    telegram: /\d{9,10}:[A-Za-z0-9_-]{35}/g,
    aws: /AKIA[0-9A-Z]{16}/g,
    generic_key: /api[_-]?key[\s]*[=:]\s*['"]?([a-zA-Z0-9_-]{20,})['"]?/gi,
  };

  const leaks = [];
  for (const [type, pattern] of Object.entries(patterns)) {
    const matches = content.match(pattern);
    if (matches) {
      leaks.push({
        type,
        count: matches.length,
        samples: matches.slice(0, 2).map((m) => m.substring(0, 10) + "..."),
      });
    }
  }

  return {
    safe: leaks.length === 0,
    leaks,
    recommendation:
      leaks.length > 0
        ? "Rotate all exposed keys immediately!"
        : "No leaks detected",
  };
};
```

### Environment Variable Validation

**Problem:** Fehlende oder falsch konfigurierte Umgebungsvariablen
**Validation-Script:**

```javascript
// n8n Environment Variables Validator
const validateEnvironment = () => {
  const required = ["OPENAI_API_KEY", "TELEGRAM_TOKEN", "N8N_ENCRYPTION_KEY"];

  const optional = [
    "N8N_BASIC_AUTH_ACTIVE",
    "N8N_BASIC_AUTH_USER",
    "WEBHOOK_URL",
  ];

  const validation = {
    missing: [],
    present: [],
    warnings: [],
  };

  // Check required variables
  for (const varName of required) {
    const value = process.env[varName];
    if (!value) {
      validation.missing.push(varName);
    } else {
      validation.present.push({
        name: varName,
        length: value.length,
        preview: value.substring(0, 5) + "...",
      });

      // Security checks
      if (value.length < 10) {
        validation.warnings.push(`${varName} seems too short`);
      }
      if (value === "your_key_here" || value === "placeholder") {
        validation.warnings.push(`${varName} contains placeholder value`);
      }
    }
  }

  return {
    valid: validation.missing.length === 0 && validation.warnings.length === 0,
    ...validation,
  };
};
```

### Docker Deployment Debugging

**Problem:** Container startet nicht oder verliert Daten
**Debug-Checklist:**

```bash
# 1. Container Status prüfen
docker ps -a | grep n8n

# 2. Logs analysieren
docker logs n8n --tail 100 --follow

# 3. Volume-Permissions prüfen
docker exec n8n ls -la /home/node/.n8n

# 4. Environment Variables verifizieren
docker exec n8n env | grep -E "(OPENAI|TELEGRAM|N8N)"

# 5. Netzwerk-Connectivity testen
docker exec n8n curl -I https://api.openai.com

# 6. Port-Binding verifizieren
netstat -tulpn | grep 5678
```

### HTTPS/SSL Certificate Debugging

**Problem:** SSL-Zertifikat-Fehler oder ungültige Certificates
**Validation-Script:**

```javascript
// SSL Certificate Validator
const validateSSL = async (domain) => {
  const https = require("https");

  return new Promise((resolve) => {
    const options = {
      hostname: domain,
      port: 443,
      path: "/",
      method: "GET",
      rejectUnauthorized: true,
    };

    const req = https.request(options, (res) => {
      const cert = res.socket.getPeerCertificate();

      resolve({
        valid: true,
        issuer: cert.issuer.O,
        subject: cert.subject.CN,
        validFrom: cert.valid_from,
        validTo: cert.valid_to,
        daysRemaining: Math.floor(
          (new Date(cert.valid_to) - new Date()) / (1000 * 60 * 60 * 24),
        ),
      });
    });

    req.on("error", (error) => {
      resolve({
        valid: false,
        error: error.message,
      });
    });

    req.end();
  });
};

// Usage
const certInfo = await validateSSL("your-domain.com");
if (certInfo.daysRemaining < 30) {
  console.warn("⚠️ Certificate expires in", certInfo.daysRemaining, "days");
}
```

### Backup & Recovery Testing

**Problem:** Backups sind korrupt oder unvollständig
**Backup-Validation:**

```bash
#!/bin/bash
# backup-validator.sh

BACKUP_DIR="./backups"
TEST_RESTORE_DIR="./test-restore"

echo "🔍 Validating backups..."

# Test workflow export
if [ -f "$BACKUP_DIR/workflows.json" ]; then
    echo "✓ Workflow backup exists"

    # Validate JSON structure
    if jq empty "$BACKUP_DIR/workflows.json" 2>/dev/null; then
        echo "✓ Workflow JSON valid"
    else
        echo "❌ Workflow JSON corrupted!"
        exit 1
    fi
else
    echo "❌ No workflow backup found!"
    exit 1
fi

# Test data store backup
if [ -f "$BACKUP_DIR/datastore.tar.gz" ]; then
    echo "✓ Data store backup exists"

    # Test extraction
    mkdir -p "$TEST_RESTORE_DIR"
    if tar -xzf "$BACKUP_DIR/datastore.tar.gz" -C "$TEST_RESTORE_DIR" 2>/dev/null; then
        echo "✓ Backup archive extractable"
        rm -rf "$TEST_RESTORE_DIR"
    else
        echo "❌ Backup archive corrupted!"
        exit 1
    fi
else
    echo "❌ No data store backup found!"
    exit 1
fi

echo "✅ All backups validated successfully"
```

### Rate Limiting & API Throttling

**Problem:** API-Calls werden geblockt wegen Rate-Limits
**Smart Rate Limiter:**

```javascript
// Adaptive Rate Limiter für n8n
class RateLimiter {
  constructor(maxRequestsPerMinute = 60) {
    this.maxRequests = maxRequestsPerMinute;
    this.requests = [];
    this.backoffMultiplier = 1;
  }

  async waitIfNeeded() {
    const now = Date.now();
    const oneMinuteAgo = now - 60000;

    // Cleanup old requests
    this.requests = this.requests.filter((time) => time > oneMinuteAgo);

    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = Math.max(0, 60000 - (now - oldestRequest));
      const adjustedWait = waitTime * this.backoffMultiplier;

      console.log(`⏳ Rate limit reached. Waiting ${adjustedWait}ms...`);
      await new Promise((resolve) => setTimeout(resolve, adjustedWait));

      // Exponential backoff
      this.backoffMultiplier = Math.min(this.backoffMultiplier * 1.5, 5);
    } else {
      // Reset backoff on success
      this.backoffMultiplier = 1;
    }

    this.requests.push(now);
  }

  getStats() {
    const now = Date.now();
    const oneMinuteAgo = now - 60000;
    const recentRequests = this.requests.filter((time) => time > oneMinuteAgo);

    return {
      requestsInLastMinute: recentRequests.length,
      availableSlots: this.maxRequests - recentRequests.length,
      backoffMultiplier: this.backoffMultiplier,
    };
  }
}

// Usage in n8n Function Node
const limiter =
  global.rateLimiter || (global.rateLimiter = new RateLimiter(60));
await limiter.waitIfNeeded();
// Proceed with API call...
```

---

## ✅ Zusammenfassung

Nach Kapitel 11 kannst du:

- API-Schlüssel und Umgebungen sicher verwalten,
- Benutzer- und Zugriffsrechte einrichten,
- deine Agentenumgebung produktionsreif deployen,
- Backups und Monitoring automatisieren,
- Security-Leaks systematisch erkennen und beheben,
- und Systemfehler sicher diagnostizieren.

Im nächsten Kapitel (12) wirst du lernen, **wie du das gesamte System evaluierst, skalierst und ggf. mit externen Datenfeeds (TradingView, News, Sentiment-APIs)** kombinierst – um deinen FTMO-Agenten wirklich marktfähig zu machen.
