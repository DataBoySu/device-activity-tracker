<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">Geräteaktivitätsverfolger</h1>

<p align="center">WhatsApp & Signal Aktivitätsverfolger über RTT-Analyse</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **HAFTUNGSAUSSCHLUSS**: Nur für Bildungszwecke und Sicherheitsforschung gedacht. Zeigt Privatsphäre-Schwachstellen in WhatsApp und Signal.

## Übersicht

Dieses Projekt implementiert die Forschung aus dem Paper **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** von Gabriel K. Gegenhuber, Maximilian Günther, Markus Maier, Aljosha Judmayer, Florian Holzbauer, Philipp É. Frenzel und Johanna Ullrich (Universität Wien & SBA Research).

**Was es tut:** Durch Messen der Round-Trip Time (RTT) von WhatsApp-Nachrichtenbestätigungen kann dieses Tool erkennen:
- Wenn ein Benutzer sein Gerät aktiv verwendet (niedrige RTT)
- Wenn das Gerät im Standby-/Leerlaufmodus ist (höhere RTT)
- Potenzielle Ortsänderungen (Mobile Daten vs. WiFi)
- Aktivitätsmuster im Laufe der Zeit

**Sicherheitsimplikationen:** Dies zeigt eine erhebliche Privatsphäre-Schwachstelle in Messaging-Apps, die für Überwachung ausgenutzt werden kann.

## Beispiel

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## Installation

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**Anforderungen:** Node.js 20+, npm, WhatsApp-Konto

## Verwendung

### Docker (Empfohlen)

Die einfachste Möglichkeit, die Anwendung zu starten, ist die Verwendung von Docker:

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

Die Anwendung wird unter folgenden Adressen verfügbar sein:
- Frontend: [http://localhost:3000](http://localhost:3000) (oder dein konfigurierter `CLIENT_PORT`)
- Backend: [http://localhost:3001](http://localhost:3001) (oder dein konfigurierter `BACKEND_PORT`)

Um die Container zu stoppen:

```bash
docker compose down
```

### Manuelle Einrichtung

#### Web-Oberfläche

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

Öffne `http://localhost:3000`, scanne den QR-Code mit WhatsApp und gib dann eine Telefonnummer ein, um den Verlauf zu verfolgen (z. B. `491701234567`).

### CLI-Schnittstelle (nur WhatsApp)

```bash
npm start
```

Folge den Prompts, um dich zu authentifizieren und die Zielnummer einzugeben.

**Beispiel-Ausgabe:**

```
╔════════════════════════════════════════════════════════════════╗
║ 🟡 Device Status Update - 09:41:51                             ║
╠════════════════════════════════════════════════════════════════╣
║ JID:        ***********@lid                                    ║
║ Status:     Standby                                            ║
║ RTT:        1104ms                                             ║
║ Avg (3):    1161ms                                             ║
║ Median:     1195ms                                             ║
║ Threshold:  1075ms                                             ║
╚════════════════════════════════════════════════════════════════╝
```

- **🟢 Online**: Gerät wird aktiv genutzt (RTT unter Schwellenwert)
- **🟡 Standby**: Gerät ist inaktiv/gesperrt (RTT über Schwellenwert)
- **🔴 Offline**: Gerät ist offline oder nicht erreichbar (kein CLIENT ACK empfangen)

## Wie es funktioniert

Der Tracker sendet Prüf-Nachrichten und misst die Round-Trip Time (RTT), um die Geräteaktivität zu erkennen. Zwei Prüf-Methoden sind verfügbar:

### Probe Methods

| Method | Description |
|--------|-|
| **Löschen** (Standard) | Sendet eine "delete"-Anfrage für eine nicht vorhandene Nachrichten-ID. |
| **Reaktion** | Sendet ein Reaktionsemoji an eine nicht vorhandene Nachrichten-ID. |

### Erkennungslogik

Die Zeit zwischen dem Senden der Prüfnachricht und dem Empfang des CLIENT ACK (Status 3) wird als RTT gemessen. Der Gerätestatus wird mithilfe eines dynamischen Schwellenwerts erkannt, der als 90 % des Median-RTT berechnet wird: Werte unterhalb des Schwellenwerts deuten auf aktiven Gebrauch hin, Werte oberhalb deuten auf Standby-Modus hin. Die Messungen werden in einer Historie gespeichert und der Median wird kontinuierlich aktualisiert, um sich an unterschiedliche Netzwerkbedingungen anzupassen.

### Wechseln der Prüfmethoden

Im Web-Interface kannst du zwischen Prüfmethoden wechseln, indem du den Dropdown-Menü im Steuerpanel verwendest. Im CLI-Modus wird die Löschen-Methode standardmäßig verwendet.

## Häufige Probleme

- **Keine Verbindung zu WhatsApp**: Lösche den Ordner `auth_info_baileys/` und scann den QR-Code erneut.

## Projektstruktur

```
device-activity-tracker/
├── src/
│   ├── tracker.ts         # WhatsApp RTT analysis logic
│   ├── signal-tracker.ts  # Signal RTT analysis logic
│   ├── server.ts          # Backend API server (both platforms)
│   └── index.ts           # CLI interface
├── client/                # React web interface
└── package.json
```

## How to Protect Yourself

The most effective mitigation is to enable “Block unknown account messages” in WhatsApp under
Settings → Privacy → Advanced.

This setting may reduce an attacker’s ability to spam probe reactions from unknown numbers, because WhatsApp blocks high-volume messages from unknown accounts.
However, WhatsApp does not disclose what “high volume” means, so this does not fully prevent an attacker from sending a significant number of probe reactions before rate-limiting kicks in.

Disabling read receipts helps with regular messages but does not protect against this specific attack. As of December 2025, this vulnerability remains exploitable in WhatsApp and Signal.

## Ethik und rechtliche Aspekte

⚠️ Nur für Forschungszwecke und Bildung. Niemals Personen ohne ausdrückliche Zustimmung verfolgen – dies kann gegen Datenschutzgesetze verstoßen. Authentifizierungsdaten (`auth_info_baileys/`) werden lokal gespeichert und dürfen niemals in die Versionskontrolle eingecheckt werden.

## Zitierung

Basierend auf Forschung von Gegenhuber et al., Universität Wien & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## Lizenz

MIT-Lizenz - Siehe LICENSE-Datei.

Built with [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

---

**Verwende dies verantwortungsbewusst. Dieses Tool demonstriert echte Sicherheitsanfälligkeiten, die Millionen von Nutzern betreffen.**

## Sternengeschichte

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

