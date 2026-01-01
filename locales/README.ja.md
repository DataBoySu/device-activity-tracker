<!--START_SECTION:navbar-->
<div align="center">
  <a href="../README.md">🇺🇸 English</a> | <a href="README.de.md">🇩🇪 Deutsch</a> | <a href="README.fr.md">🇫🇷 Français</a> | <a href="README.hi.md">🇮🇳 हिंदी</a> | <a href="README.ja.md">🇯🇵 日本語</a> | <a href="README.ko.md">🇰🇷 한국어</a> | <a href="README.pt.md">🇵🇹 Português</a> | <a href="README.ru.md">🇷🇺 Русский</a> | <a href="README.zh.md">🇨🇳 中文</a>
</div>
<!--END_SECTION:navbar-->

<h1 align="center">デバイス活動トラッカー</h1>

<p align="center">RTT分析によるWhatsApp & Signal活動トラッカー</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat&logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/React-18+-61DAFB?style=flat&logo=react&logoColor=black" alt="React"/>
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License MIT"/>
</p>

> ⚠️ **免責事項**: 教育およびセキュリティ研究のためのプロトタイプのみを目的としています。WhatsAppおよびSignalにおけるプライバシーの脆弱性を示しています。

## 概要

このプロジェクトは、Gabriel K. Gegenhuber、Maximilian Günther、Markus Maier、Aljosha Judmayer、Florian Holzbauer、Philipp É. Frenzel、およびJohanna Ullrich（ウィーン大学およびSBAリサーチ）による論文 **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** における研究を実装しています。

**提供する機能:** WhatsAppのメッセージ配信受領証のラウンドトリップタイム（RTT）を測定することで、このツールは以下を検出できます：
- ユーザーがデバイスを積極的に使用しているとき（RTTが低い）
- デバイスがスタンバイ/アイドルモードにあるとき（RTTが高い）
- 位置変更の可能性（モバイルデータ vs. WiFi）
- 時間経過に伴う活動パターン

**セキュリティ上の影響:** これは、監視のための悪用が可能であることを示す、メッセージングアプリにおける重大なプライバシーの脆弱性を示しています。

## 例

![WhatsApp Activity Tracker Interface](../example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## インストール

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd device-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**要件:** Node.js 20+, npm, WhatsApp アカウント

## 使い方

### Docker (推奨)

アプリケーションを実行する最も簡単な方法は Docker を使用することです：

```bash
# Copy environment template
cp .env.example .env

# (Optional) Customize ports in .env file
# BACKEND_PORT=3001
# CLIENT_PORT=3000

# Build and start containers
docker compose up --build
```

アプリケーションは以下で利用可能です:
- フロントエンド: [http://localhost:3000](http://localhost:3000) (または設定された `CLIENT_PORT`)
- バックエンド: [http://localhost:3001](http://localhost:3001) (または設定された `BACKEND_PORT`)

コンテナを停止するには:

```bash
docker compose down
```

### マニュアル設定

#### ワークステーションインターフェース

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

`http://localhost:3000` を開き、WhatsApp で QR コードをスキャンした後、電話番号を入力してトラッキングを開始します（例: `491701234567`）。

### CLIインターフェース（WhatsAppのみ）

```bash
npm start
```

プロンプトに従って認証し、ターゲット番号を入力してください。

**例:**

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

- **🟢 Online**: デバイスが積極的に使用されています（RTTがしきい値以下）
- **🟡 Standby**: デバイスがアイドル/ロックされています（RTTがしきい値以上）
- **🔴 Offline**: デバイスがオフラインまたは到達不能です（CLIENT ACKが受信されていない）

## 仕組み

トラッカーはプローブメッセージを送信し、デバイスの活動を検出するために往復時間（RTT）を測定します。利用可能なプローブ方法は2つあります:

### Probe Methods

| Method | Description |
|--------|-|
| **Delete** (Default) | 不存在のメッセージIDに対して「削除」リクエストを送信します。 |
| **Reaction** | 不存在のメッセージIDに対してリアクションエモジを送信します。 |

### 検出ロジック

プローブメッセージを送信してからCLIENT ACK（ステータス3）を受信するまでの時間はRTTとして測定されます。デバイスの状態は、中央値RTTの90％として計算された動的しきい値を使用して検出されます。しきい値以下の値はアクティブ使用を示し、しきい値以上の値はスタンバイモードを示します。測定値は履歴に保存され、中央値は継続的に更新されて、さまざまなネットワーク条件に適応します。

### プローブ方法の切り替え

ウェブインターフェースでは、コントロールパネルのドロップダウンを使用してプローブ方法を切り替えることができます。CLIモードでは、削除方法がデフォルトで使用されます。

## 一般的な問題

- **WhatsAppに接続できない**: `auth_info_baileys/` フォルダを削除し、QRコードを再スキャンしてください。

## プロジェクト構成

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

## 自分を守る方法

最も効果的な緩和策は、WhatsAppの「未確認アカウントからのメッセージをブロック」を有効にする事です。
設定 → プライバシー → 高度な設定の下でこの設定を有効にしてください。

この設定は、攻撃者が未知の番号から高頻度のメッセージを送信してプローブ反応をスパムする能力を減らす可能性があります。WhatsAppは「高頻度」という用語の定義を明らかにしていないため、レート制限が発動するまでに攻撃者が多数のプローブ反応を送信する可能性は完全には防げません。

既読通知を無効にすることは通常のメッセージに対しては効果的ですが、この特定の攻撃には保護されません。2025年12月時点では、この脆弱性はWhatsAppおよびSignalにおいても依然として悪用可能です。

## Ethical & Legal Considerations

⚠️ 研究および教育目的でのみ使用してください。明示的な同意なしに人物を追跡することは禁止されており、これはプライバシー法に違反する可能性があります。認証データ（`auth_info_baileys/`）はローカルに保存され、バージョン管理にコミットしてはいけません。

## 出典

Gegenhuber 他、ウィーン大学 & SBA Research の研究に基づく

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
```

## ライセンス

MIT ライセンス - LICENSE ファイルをご覧ください。

[@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys) で構築されました。

---

**責任ある使用をお願いします。このツールは、何百万人ものユーザーに影響を与える実際のセキュリティ脆弱性を示しています。**

## スターハistory

[![Star History Chart](https://api.star-history.com/svg?repos=gommzystudio/device-activity-tracker&type=date&legend=top-left)](https://www.star-history.com/#gommzystudio/device-activity-tracker&type=date&legend=top-left)

