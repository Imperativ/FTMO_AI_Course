# 🤖 FTMO AI Trading Agent Course

> **Comprehensive course for building intelligent trading agents using n8n workflows and LLM integration**

[![GitHub Stars](https://img.shields.io/github/stars/Imperativ/FTMO_AI_Course?style=flat-square)](https://github.com/Imperativ/FTMO_AI_Course/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/Imperativ/FTMO_AI_Course?style=flat-square)](https://github.com/Imperativ/FTMO_AI_Course/network/members)
[![License](https://img.shields.io/github/license/Imperativ/FTMO_AI_Course?style=flat-square)](https://github.com/Imperativ/FTMO_AI_Course/blob/main/LICENSE)

---

## 🎯 **Über diesen Kurs**

Dieser umfassende Kurs lehrt dir, wie du **intelligente Trading-Agenten** entwickelst, die als Assistenten für FTMO-Challenges und Live-Trading fungieren. Du lernst, moderne KI-Technologien (LLMs) mit bewährten Workflow-Automatisierung (n8n) zu kombinieren, um robuste, nachvollziehbare und ethische Trading-Systeme zu schaffen.

### **Was du lernen wirst:**
- 🧠 **KI-Agenten-Architektur** für Trading-Anwendungen
- ⚙️ **n8n Workflow-Automatisierung** für komplexe Datenflüsse
- 🔗 **LLM-Integration** für intelligente Marktanalyse
- 📊 **Risikomanagement** und Positionsgrößenberechnung
- 💾 **Persistente Speicher** und Feedback-Schleifen
- 🔧 **Performance-Optimierung** und Qualitätsmetriken
- 🛡️ **Sicherheit** und ethische Überlegungen

---

## 📚 **Kurs-Struktur**

### 🟢 **Grundlagen-Block (Kapitel 1-5)**
*Fundamentales Verständnis und erste praktische Schritte*

| Kapitel | Titel | Schwierigkeit | Zeilen |
|---------|-------|---------------|--------|
| [**1**](Kapitel_1_Fundament_Agenten.md) | Fundament: Denken wie ein KI-Agenten-Architekt | 🟢 Einsteiger | 167 |
| [**2**](Kapitel_2_n8n_verstehen.md) | n8n verstehen und vorbereiten | 🟢 Einsteiger | 215 |
| [**3**](Kapitel_3_LLM_Integration.md) | ChatGPT-Integration vertiefen und Datenanalyse | 🟢 Einsteiger | 249 |
| [**4**](Kapitel_4_Risikomanagement.md) | Risikomanagement und Entscheidungslogik | 🟡 Fortgeschritten | 222 |
| [**5**](Kapitel_5_Speicher_und_Feedback.md) | Gedächtnis & Feedback: Deinen Agenten lernfähig machen | 🟡 Fortgeschritten | 262 |

### 🟡 **Optimierung-Block (Kapitel 6-9)**
*Verbesserung und Qualitätssteigerung deiner Agenten*

| Kapitel | Titel | Schwierigkeit | Zeilen |
|---------|-------|---------------|--------|
| [**6**](Kapitel_6_Parameter_Tuning.md) | Parameter-Tuning und A/B-Vergleiche | 🟡 Fortgeschritten | 160 |
| [**7**](Kapitel_7_Multi_Agent_System.md) | Multi-Agent-Koordination und Rollenaufteilung | 🟡 Fortgeschritten | 185 |
| [**8**](Kapitel_8_Performance_Optimierung.md) | Performance-Optimierung und Ressourcenmanagement | 🟡 Fortgeschritten | 196 |
| [**9**](Kapitel_9_Qualitaetsmetriken.md) | Qualitätsmetriken und kontinuierliche Verbesserung | 🟡 Fortgeschritten | 202 |

### 🔵 **Professional-Block (Kapitel 10-12)**
*Dokumentation, Sicherheit und professioneller Deployment*

| Kapitel | Titel | Schwierigkeit | Zeilen |
|---------|-------|---------------|--------|
| [**10**](Kapitel_10_Dokumentation_Versionierung.md) | Dokumentation, Versionierung und Auditierbarkeit | 🔵 Profi | 206 |
| [**11**](Kapitel_11_Sicherheit_Deployment.md) | Sicherheit, Compliance und Deployment | 🔵 Profi | 198 |
| [**12**](Kapitel_12_Evaluation_Skalierung.md) | Evaluation, Skalierung und Enterprise-Integration | 🔵 Profi | 196 |

### 🔴 **Advanced-Block (Kapitel 13-15)**
*Präsentation, Dokumentation und fortgeschrittene Kollaboration*

| Kapitel | Titel | Schwierigkeit | Zeilen |
|---------|-------|---------------|--------|
| [**13**](Kapitel_13_Systemdossier_Praesentation.md) | Systemdossier, Export und Präsentation | 🔴 Experte | 192 |
| [**14**](Kapitel_14_Adaptive_Strategien.md) | Adaptive Strategien und Machine Learning Integration | 🔴 Experte | 177 |
| [**15**](Kapitel_15_Multi_Agent_Kollaboration.md) | Multi-Agent-Kollaboration und hybride Entscheidungsnetze | 🔴 Experte | 188 |

### 🟣 **Specialized-Block (Kapitel 16-18)**
*Echtzeit-Integration und ethische Überlegungen*

| Kapitel | Titel | Schwierigkeit | Zeilen |
|---------|-------|---------------|--------|
| [**16**](Kapitel_16_Echtzeit_Integration.md) | Echtzeit-Integration und Live-Handel | 🟣 Spezialist | 138 |
| [**17**](Kapitel_17_Simulation_Testumgebung.md) | Simulation und Testumgebung | 🟣 Spezialist | 119 |
| [**18**](Kapitel_18_Ethik_Aufsicht.md) | Ethik, Haftung und menschliche Aufsicht | 🟣 Spezialist | 107 |

---

## 🚀 **Quick Start**

### **Voraussetzungen**
- 💻 Windows/macOS/Linux
- 🌐 Node.js 18+ (für n8n)
- 🤖 OpenAI API Key oder alternative LLM-Zugang
- 📱 Telegram Bot Token (für Benachrichtigungen)
- 📊 Trading-Demo-Konto (empfohlen)

### **Installation**
1. **Repository klonen:**
   ```bash
   git clone https://github.com/Imperativ/FTMO_AI_Course.git
   cd FTMO_AI_Course
   ```

2. **n8n installieren:**
   ```bash
   npm install -g n8n
   # oder als Docker
   docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
   ```

3. **Mit Kapitel 1 starten:**
   - [Kapitel 1: Fundament](Kapitel_1_Fundament_Agenten.md)
   - Ersten Workflow in n8n erstellen
   - LLM-API testen

---

## 🛠️ **Technologie-Stack**

### **Core Technologies:**
- **[n8n](https://n8n.io/)** - Workflow-Automatisierung
- **[OpenAI API](https://openai.com/api/)** - LLM-Integration
- **[Telegram Bot API](https://core.telegram.org/bots/api)** - Benachrichtigungen
- **JavaScript/Node.js** - Custom Functions

### **Optional Extensions:**
- **Docker** - Containerized Deployment
- **PostgreSQL** - Advanced Data Storage
- **Python** - Advanced Analytics
- **TradingView** - Market Data Integration

---

## 📖 **Zusätzliche Ressourcen**

### **Chat Ausgabe Ordner**
Ergänzende Materialien und alternative Erklärungen:
- [FTMO_Teil1.md](Chat%20Ausgabe/FTMO_Teil1.md) - Kompakte Grundlagen
- [FTMO_Teil2.md](Chat%20Ausgabe/FTMO_Teil2.md) - Erweiterte Konzepte
- [FTMO_Teil3.md](Chat%20Ausgabe/FTMO_Teil3.md) - Spezielle Anwendungen

### **VS Code Integration**
Das Projekt enthält optimierte VS Code Settings:
- Debugging-freundliche Konfiguration
- Konsistente Formatierung
- InlayHints deaktiviert für bessere Performance

---

## 🏆 **Lernpfad-Empfehlungen**

### **🟢 Beginner (2-3 Wochen):**
1. Kapitel 1-3 durcharbeiten
2. Ersten n8n Workflow erstellen
3. LLM-Integration testen
4. Einfache Marktdaten-Analyse

### **🟡 Intermediate (4-6 Wochen):**
1. Kapitel 4-9 absolvieren
2. Risikomanagement implementieren
3. Multi-Agent-System aufbauen
4. Performance-Monitoring einrichten

### **🔵 Advanced (8-12 Wochen):**
1. Kapitel 10-15 meistern
2. Vollständige Dokumentation
3. Sicherheits-Compliance
4. Enterprise-Integration

### **🟣 Expert (individuell):**
1. Kapitel 16-18 studieren
2. Echtzeit-Trading-Integration
3. Ethische Richtlinien implementieren
4. Eigene Erweiterungen entwickeln

---

## ⚠️ **Wichtige Hinweise**

### **🚨 Risiko-Disclaimer**
- Dieses System ist zu **Bildungszwecken** entwickelt
- **Niemals mit echtem Geld** ohne ausgiebige Tests
- **Immer Demokonto** für erste Versuche verwenden
- **Menschliche Aufsicht** bleibt obligatorisch

### **📋 Rechtliche Überlegungen**
- Lokale Finanzvorschriften beachten
- FTMO-Regelwerk einhalten
- Keine automatisierten Trades ohne Genehmigung
- Dokumentation für Audit-Zwecke

---

## 🤝 **Beitrag & Community**

### **Beitragen:**
1. Fork das Repository
2. Erstelle Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit deine Änderungen (`git commit -m 'Add AmazingFeature'`)
4. Push zum Branch (`git push origin feature/AmazingFeature`)
5. Öffne Pull Request

### **Issues & Feedback:**
- 🐛 [Bug Reports](https://github.com/Imperativ/FTMO_AI_Course/issues)
- 💡 [Feature Requests](https://github.com/Imperativ/FTMO_AI_Course/issues)
- 📝 [Diskussionen](https://github.com/Imperativ/FTMO_AI_Course/discussions)

---

## 📊 **Kurs-Statistiken**

- **📚 18 Kapitel** mit über 3.000 Zeilen Content
- **🚨 Strukturierte Debugging-Sektionen** in allen Kapiteln
- **💻 100+ Code-Beispiele** für direkte Anwendung
- **🧩 50+ Hausaufgaben** und praktische Übungen
- **⚙️ 20+ n8n Workflow-Templates**

---

## 🏷️ **Versioning & Updates**

**Aktuelle Version:** v1.0.0

### **Changelog:**
- **v1.0.0** (Nov 2024): Initial release mit vollständigem Kurs
- **v0.9.0** (Nov 2024): Debugging-Sektionen hinzugefügt
- **v0.8.0** (Nov 2024): Basis-Kapitel 1-5 finalisiert

---

## 📄 **Lizenz**

Dieses Projekt steht unter der [MIT Lizenz](LICENSE) - siehe LICENSE-Datei für Details.

---

## 🙏 **Danksagungen**

- **n8n Community** für die großartige Workflow-Platform
- **OpenAI** für zugängliche LLM-APIs
- **FTMO** für den Trading-Challenge-Kontext
- **Alle Contributors** die diesen Kurs möglich gemacht haben

---

**⭐ Gefällt dir dieses Projekt? Gib uns einen Stern!**

[🏠 Zurück zum Anfang](#-ftmo-ai-trading-agent-course)
