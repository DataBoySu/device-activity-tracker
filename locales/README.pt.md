<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">Device Activity Tracker</h1>

<p align="center">Rastreador de Atividade do WhatsApp & Signal via Análise de RTT</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **AVISO**: Prova de conceito apenas para fins educacionais e de pesquisa de segurança. Demonstra vulnerabilidades de privacidade no WhatsApp e no Signal.

## Overview

This project implements the research from the paper **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** by Gabriel K. Gegenhuber, Maximilian Günther, Markus Maier, Aljosha Judmayer, Florian Holzbauer, Philipp É. Frenzel, and Johanna Ullrich (University of Vienna & SBA Research).

**What it does:** By measuring Round-Trip Time (RTT) of WhatsApp message delivery receipts, this tool can detect:
- When a user is actively using their device (low RTT)
- When the device is in standby/idle mode (higher RTT)
- Potential location changes (mobile data vs. WiFi)
- Activity patterns over time

**Security implications:** This demonstrates a significant privacy vulnerability in messaging apps that can be exploited for surveillance.

## Exemplo

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## Instalação

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**Requisitos:** Node.js 20+, npm, conta do WhatsApp

## Uso

### Docker (Recomendado)

A forma mais fácil de executar a aplicação é usando Docker:

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

A aplicação estará disponível em:
- Frontend: [http://localhost:3000](http://localhost:3000) (ou a porta configurada `CLIENT_PORT`)
- Backend: [http://localhost:3001](http://localhost:3001) (ou a porta configurada `BACKEND_PORT`)

Para parar os containers:

```bash
docker compose down
```

### Configuração Manual

#### Interface Web

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

Abra `http://localhost:3000`, escaneie o código QR com o WhatsApp e, em seguida, insira o número de telefone para rastrear (ex.: `491701234567`).

### Interface CLI (somente WhatsApp)

```bash
npm start
```

Siga os prompts para autenticar e inserir o número de destino.

**Exemplo de Saída:**

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

- **🟢 Online**: Dispositivo está sendo usado ativamente (RTT abaixo do limiar)
- **🟡 Standby**: Dispositivo está ocioso/travado (RTT acima do limiar)
- **🔴 Offline**: Dispositivo está offline ou inacessível (nenhum CLIENT ACK recebido)

## Como Funciona

O rastreador envia mensagens de sonda e mede o Tempo de Idade e Volta (RTT) para detectar a atividade do dispositivo. Dois métodos de sonda estão disponíveis:

### Métodos de Probe

| Método | Descrição |
|--------|-|
| **Excluir** (Padrão) | Envia uma solicitação "delete" para um ID de mensagem inexistente. |
| **Reação** | Envia um emoji de reação para um ID de mensagem inexistente. |

### Lógica de Detecção

O tempo entre o envio da mensagem de probe e o recebimento do CLIENT ACK (Status 3) é medido como RTT. O estado do dispositivo é detectado usando um limiar dinâmico calculado como 90% da mediana do RTT: valores abaixo do limiar indicam uso ativo, valores acima indicam modo em standby. As medições são armazenadas em um histórico e a mediana é atualizada continuamente para se adaptar a diferentes condições de rede.

### Alternando Métodos de Sonda

Na interface web, você pode alternar entre métodos de sonda usando o menu suspenso no painel de controle. No modo CLI, o método delete é usado por padrão.

## Problemas Comuns

- **Não Conectando ao WhatsApp**: Exclua a pasta `auth_info_baileys/` e rescaneie o código QR.

## Estrutura do Projeto

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

## Considerações Éticas e Legais

⚠️ Apenas para fins de pesquisa e educação. Nunca rastreie pessoas sem consentimento explícito - isso pode violar leis de privacidade. Dados de autenticação (`auth_info_baileys/`) são armazenados localmente e nunca devem ser enviados para controle de versão.

## Citação

Baseado em pesquisa por Gegenhuber et al., Universidade de Viena & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## Licença

Licença MIT - Veja o arquivo LICENSE.

Construído com [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

---

**Use com responsabilidade. Esta ferramenta demonstra vulnerabilidades de segurança reais que afetam milhões de usuários.**

## Histórico de Estrelas 🌟

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

