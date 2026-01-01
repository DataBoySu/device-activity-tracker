<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">Датчик активности устройства</h1>

<p align="center">Трекер активности WhatsApp и Signal через анализ RTT</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **ПРЕДУПРЕЖДЕНИЕ**: Пример для образовательных и исследовательских целей в области кибербезопасности. Показывает уязвимости в области конфиденциальности в WhatsApp и Signal.

## Общее описание

Этот проект реализует исследование из статьи **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** Габриэлем К. Гегенхубером, Максимилианом Гюнтером, Маркусом Маierом, Алььошой Юдмайером, Флорианом Хольцбауэром, Филиппом Э. Френцелем и Йоханной Ульрих (Университет Вены & SBA Research).

**Что он делает:** Измеряя время в пути (RTT) доставки сообщений WhatsApp, этот инструмент может обнаружить:
- Когда пользователь активно использует устройство (низкий RTT)
- Когда устройство находится в режиме ожидания/простоя (высокий RTT)
- Потенциальные изменения местоположения (мобильные данные vs. Wi-Fi)
- Паттерны активности со временем

**Последствия для безопасности:** Это демонстрирует значительную уязвимость приватности в приложениях для мессенджеров, которая может быть использована для слежки.

## Пример

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## Установка

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**Требования:** Node.js 20+, npm, аккаунт WhatsApp

## Использование

### Docker (Рекомендуется)

Самый простой способ запуска приложения — использование Docker:

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

Приложение будет доступно по адресу:
- Frontend: [http://localhost:3000](http://localhost:3000) (или ваш настроенный `CLIENT_PORT`)
- Backend: [http://localhost:3001](http://localhost:3001) (или ваш настроенный `BACKEND_PORT`)

Чтобы остановить контейнеры:

```bash
docker compose down
```

### Ручная настройка

#### Веб-интерфейс

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

Откройте `http://localhost:3000`, отсканируйте QR-код через WhatsApp, затем введите номер телефона для отслеживания (например, `491701234567`).

### CLI Interface (только WhatsApp)

```bash
npm start
```

Следуйте подсказкам, чтобы аутентифицироваться и ввести целевое число.

**Пример вывода:**

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

- **🟢 Online**: Device is actively being used (RTT below threshold)
- **🟡 Standby**: Device is idle/locked (RTT above threshold)
- **🔴 Offline**: Device is offline or unreachable (no CLIENT ACK received)

## Как это работает

Трекер отправляет зондирующие сообщения и измеряет время в пути (RTT), чтобы обнаружить активность устройства. Доступны два метода зондирования:

### Методы проверки

| Метод | Описание |
|--------|-|
| **Удалить** (По умолчанию) | Отправляет запрос "удалить" для несуществующего идентификатора сообщения. |
| **Реакция** | Отправляет эмодзи реакции к несуществующему идентификатору сообщения. |

### Detection Logic

The time between sending the probe message and receiving the CLIENT ACK (Status 3) is measured as RTT. Device state is detected using a dynamic threshold calculated as 90% of the median RTT: values below the threshold indicate active usage, values above indicate standby mode. Measurements are stored in a history and the median is continuously updated to adapt to different network conditions.

### Переключение методов пробы

В веб-интерфейсе вы можете переключаться между методами пробы с помощью выпадающего списка в панели управления. В режиме CLI по умолчанию используется метод удаления.

## Распространенные проблемы

- **Не подключается к WhatsApp**: Удалите папку `auth_info_baileys/` и отсканируйте QR-код заново.

## Структура проекта

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

## Этические и юридические соображения

⚠️ Только для исследовательских и образовательных целей. Никогда не отслеживайте людей без явного согласия — это может нарушать законы о защите приватности. Данные аутентификации (`auth_info_baileys/`) хранятся локально и никогда не должны добавляться в систему контроля версий.

## Цитирование

На основе исследований Gegenhuber и др., Университет Вены & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## Лицензия

MIT License - См. файл LICENSE.

Built with [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

---

**Используйте ответственно. Этот инструмент демонстрирует реальные уязвимости в безопасности, которые влияют на миллионы пользователей.**

## История звёзд

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

