# Device Activity Tracker
Whatsapp Activity Tracker via RTT Analysis

> ⚠️ **DISCLAIMER**: Proof-of-concept for educational and security research purposes only. Demonstrates privacy vulnerabilities in WhatsApp and Signal.

## Overview

This project implements the research from the paper **"Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers"** by Gabriel K. Gegenhuber, Maximilian Günther, Markus Maier, Aljosha Judmayer, Florian Holzbauer, Philipp É. Frenzel, and Johanna Ullrich (University of Vienna & SBA Research).

**What it does:** By measuring Round-Trip Time (RTT) of WhatsApp message delivery receipts, this tool can detect:
- When a user is actively using their device (low RTT)
- When the device is in standby/idle mode (higher RTT)
- Potential location changes (mobile data vs. WiFi)
- Activity patterns over time

**Security implications:** This demonstrates a significant privacy vulnerability in messaging apps that can be exploited for surveillance.

## Example

![WhatsApp Activity Tracker Interface](example.png)

The web interface shows real-time RTT measurements, device state detection, and activity patterns.

## Installation

```bash
# Clone repository
git clone https://github.com/gommzystudio/device-activity-tracker.git
cd whatsapp-activity-tracker

# Install dependencies
npm install
cd client && npm install && cd ..
```

**Requirements:** Node.js 16+, npm, WhatsApp account

## Usage

### Web Interface (Recommended)

```bash
# Terminal 1: Start backend
npm run start:server

# Terminal 2: Start frontend
npm run start:client
```

Open `http://localhost:3000`, scan QR code with WhatsApp, then enter phone number to track (e.g., `491701234567`).

### CLI Interface

```bash
npm start
```

Follow prompts to authenticate and enter target number.

## How It Works

1. **Probe Messages**: Sends harmless reaction messages to non-existent message IDs
2. **RTT Measurement**: Measures time between send and delivery acknowledgment
3. **State Detection**: Analyzes RTT patterns to determine device state:
   - **Online**: Low RTT (~200-800ms) - actively using device
   - **Standby**: Higher RTT (~1000-3000ms) - idle/screen off
   - **Offline**: Very high RTT or timeout (>5000ms)

**The vulnerability:** Devices respond significantly faster when actively in use compared to standby mode. The attack is completely silent - no notifications, no visible messages.

## Understanding Output

- **RTT (ms)**: Current round-trip time
- **Avg(3)**: Moving average of last 3 measurements
- **State**: Online / Standby / Offline / Calibrating
- **Threshold**: Baseline for state detection

## Project Structure

```
device-activity-tracker/
├── src/
│   ├── tracker.ts      # Core RTT analysis logic
│   ├── server.ts       # Backend API server
│   └── index.ts        # CLI interface
├── client/             # React web interface
└── package.json
```

## How to Protect Yourself

This attack exploits fundamental design flaws in messaging apps. Here are ways to mitigate the risk:

### For Users

1. **Block Unknown Numbers**
   - In WhatsApp: Settings → Privacy → Messages → "My contacts"
   - This prevents unknown numbers from sending you any messages (including silent reactions)
   - **Most effective protection available to end users**

2. **Disable Read Receipts** (Limited effectiveness)
   - Helps with regular messages but NOT with this specific attack
   - Silent delivery receipts are still sent regardless of this setting

### For App Developers (WhatsApp, Signal, etc.)

The following mitigations should be implemented to fix this vulnerability:

1. **RTT Normalization**
   - Add artificial delays to delivery receipts
   - Normalize all RTT responses to a consistent timeframe (~2000ms ±200ms)
   - Randomize response timing to prevent pattern detection

2. **Rate Limiting**
   - Implement strict rate limits on probe-like messages
   - Detect and block accounts sending repeated reactions to non-existent messages
   - Exponential backoff for suspicious activity patterns

3. **Behavioral Analysis**
   - Flag accounts with abnormal messaging patterns
   - Temporary throttling for suspected tracking behavior
   - Require CAPTCHA verification for suspicious accounts

4. **OS-Level Protection** (iOS/Android)
   - Implement consistent network response times regardless of device state
   - Add noise to all network timing at the OS level
   - Queue low-priority messages during standby with artificial delays

**Status:** As of December 2025, these protections are NOT implemented in WhatsApp or Signal. This vulnerability remains exploitable.

## Ethical & Legal Considerations

**⚠️ Important:**
- For **research and education only**
- Do NOT track people without explicit consent
- May violate privacy laws in your jurisdiction
- Highlights need for better privacy protections in messaging apps

**Authentication data** (`auth_info_baileys/`) is stored locally - never commit to version control.

## Citation

Based on research by Gegenhuber et al., University of Vienna & SBA Research:

```bibtex
@inproceedings{gegenhuber2024careless,
  title={Careless Whisper: Exploiting Silent Delivery Receipts to Monitor Users on Mobile Instant Messengers},
  author={Gegenhuber, Gabriel K. and G{\"u}nther, Maximilian and Maier, Markus and Judmayer, Aljosha and Holzbauer, Florian and Frenzel, Philipp {\'E}. and Ullrich, Johanna},
  year={2024},
  organization={University of Vienna, SBA Research}
}
```

## License

MIT License - See LICENSE file.

Built with [@whiskeysockets/baileys](https://github.com/WhiskeySockets/Baileys)

---

**Use responsibly. This tool demonstrates real security vulnerabilities that affect millions of users.**

