<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">Suivi de l'activité du dispositif</h1>

<p align="center">Suivi de l'activité de WhatsApp et Signal via l'analyse RTT</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **AVIS DE NON-RESPONSABILITÉ** : Concept de preuve uniquement à des fins pédagogiques et de recherche en sécurité. Démontre des vulnérabilités de confidentialité dans WhatsApp et Signal.

## Aperçu

Ce projet implémente la recherche de l'article **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** par Gabriel K. Gegenhuber, Maximilian Günther, Markus Maier, Aljosha Judmayer, Florian Holzbauer, Philipp É. Frenzel et Johanna Ullrich (Université de Vienne & SBA Research).

**Ce que cela fait :** En mesurant le temps de trajet aller-retour (RTT) des reçus de livraison de messages WhatsApp, cet outil peut détecter :
- Quand un utilisateur utilise activement son appareil (RTT faible)
- Quand l'appareil est en mode veille/inactif (RTT plus élevé)
- Des changements potentiels de localisation (données mobiles vs. WiFi)
- Des schémas d'activité au fil du temps

**Implications de sécurité :** Cela démontre une vulnérabilité importante à la vie privée dans les applications de messagerie qui peut être exploitée pour la surveillance.

## Exemple

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

**Exigences :** Node.js 20+, npm, compte WhatsApp

## Utilisation

### Docker (Recommandé)

La manière la plus simple d'exécuter l'application est d'utiliser Docker :

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

L'application sera disponible à l'adresse suivante :
- Frontend : [http://localhost:3000](http://localhost:3000) (ou votre port configuré `CLIENT_PORT`)
- Backend : [http://localhost:3001](http://localhost:3001) (ou votre port configuré `BACKEND_PORT`)

Pour arrêter les conteneurs :

```bash
docker compose down
```

### Configuration manuelle

#### Interface web

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

Ouvrez `http://localhost:3000`, scannez le code QR avec WhatsApp, puis entrez le numéro de téléphone pour le suivre (par exemple, `491701234567`).

### Interface CLI (uniquement WhatsApp)

```bash
npm start
```

Suivre les invites pour vous authentifier et entrer le numéro cible.

**Exemple de sortie :**

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

- **🟢 En ligne** : L'appareil est activement utilisé (RTT en dessous du seuil)
- **🟡 En veille** : L'appareil est inactif/verrouillé (RTT au-dessus du seuil)
- **🔴 Hors ligne** : L'appareil est hors ligne ou inatteignable (aucun CLIENT ACK reçu)

## Fonctionnement

Le tracker envoie des messages de sondage et mesure le Temps de trajet round-trip (RTT) pour détecter l'activité du dispositif. Deux méthodes de sondage sont disponibles :

### Méthodes de sondage

| Méthode | Description |
|--------|-|
| **Supprimer** (Par défaut) | Envoie une demande "supprimer" pour un ID de message inexistant. |
| **Réaction** | Envoie un emoji de réaction à un ID de message inexistant. |

### Logique de détection

Le temps entre l'envoi du message de sonde et la réception du CLIENT ACK (Statut 3) est mesuré comme étant le RTT. L'état du dispositif est détecté à l'aide d'un seuil dynamique calculé comme étant 90 % de la médiane du RTT : les valeurs en dessous du seuil indiquent une utilisation active, les valeurs au-dessus indiquent le mode veille. Les mesures sont stockées dans un historique et la médiane est continuellement mise à jour pour s'adapter aux différentes conditions réseau.

### Changer les méthodes de sonde

Dans l'interface web, vous pouvez changer entre les méthodes de sonde en utilisant le menu déroulant du panneau de contrôle. En mode CLI, la méthode delete est utilisée par défaut.

## Problèmes courants

- **Pas de connexion à WhatsApp** : Supprimez le dossier `auth_info_baileys/` et rescannez le code QR.

## Structure du projet

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

## Considérations éthiques et légales

⚠️ Uniquement pour des fins de recherche et éducatives. Ne jamais suivre des personnes sans consentement explicite - cela peut violer les lois sur la vie privée. Les données d'authentification (`auth_info_baileys/`) sont stockées localement et ne doivent jamais être commitées dans le contrôle de version.

## Citation

Basé sur des recherches menées par Gegenhuber et al., Université de Vienne & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## Licence

Licence MIT - Voir le fichier LICENSE.

Construit avec [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

---

**Utilisez-le avec responsabilité. Ce outil illustre des vulnérabilités de sécurité réelles qui affectent des millions d'utilisateurs.**

## Historique des étoiles

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

