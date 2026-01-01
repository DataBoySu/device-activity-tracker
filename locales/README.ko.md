<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">기기 활동 추적기</h1>

<p align="center">RTT 분석을 통한 WhatsApp 및 Signal 활동 추적기</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **경고**: 교육 및 보안 연구 목적의 개념 증명만으로, WhatsApp 및 Signal의 개인정보 취약점을 보여줍니다.

## 개요

이 프로젝트는 Gabriel K. Gegenhuber, Maximilian Günther, Markus Maier, Aljosha Judmayer, Florian Holzbauer, Philipp É. Frenzel, Johanna Ullrich(비엔나 대학교 & SBA 연구소)가 작성한 논문 **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"**의 연구를 구현합니다.

**기능:** WhatsApp 메시지 전달 수신 확인의 Round-Trip Time(RTT)을 측정함으로써 이 도구는 다음을 감지할 수 있습니다:
- 사용자가 기기 사용 중인 경우 (낮은 RTT)
- 기기가 대기/비활성 모드인 경우 (높은 RTT)
- 잠재적인 위치 변화 (모바일 데이터 vs. WiFi)
- 시간에 따른 활동 패턴

**보안 영향:** 이는 감시를 위해 악용될 수 있는 메시징 앱의 중요한 개인정보 유출 취약점을 보여줍니다.

## 예시

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## 설치

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**요구사항:** Node.js 20+, npm, WhatsApp 계정

## 사용법

### Docker (추천)

애플리케이션을 실행하는 가장 쉬운 방법은 Docker를 사용하는 것입니다:

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

애플리케이션은 다음 주소에서 사용할 수 있습니다:
- 프론트엔드: [http://localhost:3000](http://localhost:3000) (또는 설정한 `CLIENT_PORT`)
- 백엔드: [http://localhost:3001](http://localhost:3001) (또는 설정한 `BACKEND_PORT`)

컨테이너를 중지하려면:

```bash
docker compose down
```

### 수동 설정

#### 웹 인터페이스

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

`http://localhost:3000`을 열고, WhatsApp으로 QR 코드를 스캔한 후 추적할 전화번호를 입력하세요 (예: `491701234567`).

### CLI 인터페이스 (WhatsApp만 지원)

```bash
npm start
```

프롬프트에 따라 인증하고 대상 번호를 입력하십시오.

**예시 출력:**

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

- **🟢 Online**: 장치가 활발히 사용되고 있음 (RTT 임계값 이하)
- **🟡 Standby**: 장치가 대기/잠김 상태 (RTT 임계값 이상)
- **🔴 Offline**: 장치가 오프라인 또는 연결 불가능 (CLIENT ACK 수신 없음)

## 작동 방식

트래커는 프로브 메시지를 보내고, 장치 활동을 감지하기 위해 Round-Trip Time (RTT)을 측정합니다. 두 가지 프로브 방법이 사용 가능합니다:

### Probe Methods

| Method | Description |
|--------|-|
| **Delete** (Default) | 존재하지 않는 메시지 ID에 대해 "delete" 요청을 보냅니다. |
| **Reaction** | 존재하지 않는 메시지 ID에 반응 이모지(예: 🐤)를 보냅니다. |

### 감지 논리

프로브 메시지를 보내는 시간과 CLIENT ACK(상태 3)를 받는 시간 사이의 시간은 RTT로 측정됩니다. 장치 상태는 중앙값 RTT의 90%로 계산된 동적 임계값을 사용하여 감지됩니다: 임계값 이하의 값은 활성 사용을 나타내고, 임계값 이상의 값은 대기 모드를 나타냅니다. 측정값은 역사에 저장되며 중앙값은 다양한 네트워크 조건에 적응하기 위해 지속적으로 업데이트됩니다.

### 프로브 방법 전환

웹 인터페이스에서 제어판의 드롭다운을 사용하여 프로브 방법을 전환할 수 있습니다. CLI 모드에서는 기본적으로 delete 방법이 사용됩니다.

## 일반적인 문제

- **WhatsApp에 연결되지 않음**: `auth_info_baileys/` 폴더를 삭제하고 QR 코드를 다시 스캔하세요.

## 프로젝트 구조

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

## 자신을 보호하는 방법

가장 효과적인 완화 방법은 WhatsApp에서 설정 → 개인정보 보호 → 고급 메뉴에서 "알 수 없는 계정의 메시지 차단"을 활성화하는 것입니다.

이 설정은 공격자가 알 수 없는 번호에서 대량의 메시지를 보내는 스팸 탐지 반응을 보내는 능력을 줄일 수 있지만, WhatsApp은 "대량"이 무엇을 의미하는지 공개하지 않기 때문에, 속도 제한이 작동하기 전에 공격자가 많은 수의 탐지 반응을 보내는 것을 완전히 방지하지는 않습니다.

읽음 확인 기능을 비활성화하면 일반 메시지에 도움이 되지만, 이 특정 공격에는 보호 효과가 없습니다. 2025년 12월 현재, 이 취약점은 WhatsApp과 Signal에서 여전히 악용될 수 있습니다.

## 윤리적 및 법적 고려사항

⚠️ 연구 및 교육 목적만에 사용하십시오. 명시적인 동의 없이 사람을 추적하는 것은 개인정보 보호법을 위반할 수 있습니다. 인증 데이터(`auth_info_baileys/`)는 로컬에 저장되며, 절대 버전 관리에 커밋해서는 안 됩니다.

## 인용

Gegenhuber 등, 비엔나 대학교 & SBA 연구의 연구를 바탕으로 작성됨:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## 라이선스

MIT 라이선스 - LICENSE 파일을 참조하십시오.

[@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)로 구축됨

---

**책임감 있게 사용하십시오. 이 도구는 수백만 명의 사용자에게 영향을 미치는 실제 보안 취약점을 보여줍니다.**

## 별 역사

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

