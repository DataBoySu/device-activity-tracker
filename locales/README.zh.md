<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">设备活动跟踪器</h1>

<p align="center">通过RTT分析进行WhatsApp和Signal活动跟踪</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **免责声明**: 仅用于教育和安全研究的概念验证。演示WhatsApp和Signal中的隐私漏洞。

## 概述

此项目实现了 Gabriel K. Gegenhuber、Maximilian Günther、Markus Maier、Aljosha Judmayer、Florian Holzbauer、Philipp É. Frenzel 和 Johanna Ullrich（维也纳大学 & SBA 研究）撰写的论文 **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** 中的研究。

**功能：** 通过测量 WhatsApp 消息送达回执的往返时间（RTT），该工具可以检测：
- 用户何时正在积极使用设备（低 RTT）
- 设备何时处于待机/空闲模式（高 RTT）
- 潜在的位置变化（移动数据 vs. WiFi）
- 随时间推移的活动模式

**安全影响：** 这表明消息应用中存在重大的隐私漏洞，可用于监视。

## 示例

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## 安装

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**要求:** Node.js 20+，npm，WhatsApp 账户

## 使用方法

### Docker（推荐）

使用 Docker 是运行应用程序最简单的方式：

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

应用程序将在以下地址可用：
- 前端：[http://localhost:3000](http://localhost:3000)（或您配置的 `CLIENT_PORT`）
- 后端：[http://localhost:3001](http://localhost:3001)（或您配置的 `BACKEND_PORT`）

要停止容器：

```bash
docker compose down
```

### 手动设置

#### Web 界面

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

打开 `http://localhost:3000`，使用 WhatsApp 扫描二维码，然后输入电话号码进行跟踪（例如 `491701234567`）。

### CLI Interface (only WhatsApp)

```bash
npm start
```

按照提示进行身份验证并输入目标号码。

**示例输出:**

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

- **🟢 在线**: 设备正在被积极使用 (RTT 低于阈值)
- **🟡 待机**: 设备处于空闲/锁定状态 (RTT 高于阈值)
- **🔴 离线**: 设备离线或无法到达 (未收到 CLIENT ACK)

## 工作原理

跟踪器发送探测消息并测量往返时间（RTT）以检测设备活动。有两种探测方法可用：

### 探针方法

| 方法 | 描述 |
|--------|-|
| **删除** (默认) | 向不存在的消息 ID 发送一个 "delete" 请求。 |
| **反应** | 向不存在的消息 ID 发送一个反应表情符号。 |

### 检测逻辑

从发送探测消息到接收 CLIENT ACK（状态 3）之间的时间被测量为 RTT。设备状态通过动态阈值进行检测，该阈值计算为中位数 RTT 的 90%：低于阈值的值表示处于活动使用状态，高于阈值的值表示处于待机模式。测量值存储在历史记录中，中位数会持续更新以适应不同的网络条件。

### 切换探测方法

在网页界面中，可以使用控制面板中的下拉菜单切换探测方法。在 CLI 模式下，默认使用删除方法。

## 常见问题

- **无法连接到 WhatsApp**：删除 `auth_info_baileys/` 文件夹并重新扫描 QR 码。

## 项目结构

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

## Ethical & Legal Considerations

⚠️ 仅限研究和教育用途。未经明确同意，不得跟踪人员 - 这可能会违反隐私法律。认证数据 (`auth_info_baileys/`) 存储在本地，绝不能提交到版本控制。

## 引用

Based on research by Gegenhuber et al., University of Vienna & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## 许可证

MIT License - 请查看 LICENSE 文件。

使用 [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys) 构建

---

**负责任地使用。此工具演示了影响数百万用户的实际安全漏洞。**

## 星星历史

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

