+++
date = '2026-04-22T00:00:00-04:00'
draft = false
title = 'Portfolio'
description = 'Projects built from scratch and documented in real time.'
author = 'Jesse Bentley'
showToc = false
hideMeta = true
outputs = ['HTML']
+++

A collection of projects built from scratch and documented honestly on this blog. Every line of code written, every mistake made, and every problem solved is recorded in the corresponding blog series.

---

## Active Projects

---

### SentinelFlip

A modular physical security reconnaissance platform built from
scratch on the Flipper Zero. Not just a scanner — a system that
collects signals, detects changes over time, and produces
actionable intelligence for penetration testing engagements
and travel security assessments.

Every feature maps to a real world use case. Every line of code
is documented on the blog. Targeting a DEF CON 2027 presentation
and open source hardware release.

**Modules**

*Wireless Recon*
- BLE device scanning and fingerprinting
- WiFi device scanner with MAC classification
- Device tracking and delta analysis over time
- Deauth attack and evil twin detection
- Pre-loaded camera manufacturer OUI database
- Live editable whitelist/blacklist via Flipper,
  CrowPanel, or companion app

*Optical Recon*
- IR lens detector (940nm illumination + ESP32-CAM)
- Hidden camera detection from adjacent spaces
- NFC/RFID vulnerability assessment

*Environmental Sensing*
- WiFi CSI motion detection through walls
- Baseline and temporal analysis engine
- Risk scoring system (0-100 per scan)
- GPS stamped scan logging

*Intelligence Layer*
- Automated PDF report generation
- Shodan and CVE database integration
- Python companion app with map visualization
- Raspberry Pi base station with 5" display

**Hardware**

*v1.0 (current)*
- Flipper Zero (control and interface)
- ESP32-C6 (WiFi + BLE scanning, CSI sensing)
- ESP32-CAM with 940nm IR LED array
- NRF24L01+PA+LNA 2.4GHz module
- NEO-6M GPS module
- Elecrow CrowPanel 3.5" LVGL touchscreen
- SentinelBoard — custom open source expansion PCB
- Custom 3D printed PETG enclosure

*v2.0 (planned)*
- Integrated ESP32-S3 replacing WiFi Dev Board
- CC1352R dual band radio (Sub-GHz + 2.4GHz)
- Single unified board — JLCPCB manufactured
- CERN Open Hardware License

*Drop Box (POC)*
- LilyGO T-SIM7000G (ESP32 + cellular)
- 18650 battery powered (24-48hr runtime)
- 3D printed PETG enclosure with magnetic feet
- Cellular backhaul for remote monitoring
- "What do you think we can do for $35?"

**Conference Talk Series**

- Talk 1 → Hack Red Con Fall 2026
  "Seeing Through Walls for $10"
  WiFi CSI sensing as a pen testing attack vector
  CFP deadline: August 31 2026

- Talk 2 → BSides Nashville 2027
  Hidden camera detection at scale

- Talk 3 → BSides Knoxville 2027
  RFID enterprise vulnerabilities

**Timeline**

- Phase 1 complete — SDK and Hello World ✅
- Phase 2 in progress — WiFi/BLE scanner 🔄
- Phases 3-24 — Active development through 2027

**Blog series:** [SentinelFlip Build](/series/flipper-zero-build/)

**GitHub:** [SentinelFlip](https://github.com/JDBentley/sentinelflip)

---

### GameMaster AI

A reinforcement learning system that teaches AI agents to play Steam games from scratch. No Gymnasium, no Stable Baselines — everything built by hand in Python and PyTorch.

**Tech stack**

- Python 3.11
- PyTorch (custom neural networks)
- Tesseract OCR (screen state extraction)
- Anthropic API (LLM program synthesis)
- NVIDIA RTX 3070 Ti (training GPU)

**Current status**

- Phase 1 in progress — Melvor Idle
- Phase 2 pending — Military Incremental Complex

**Transfer learning experiment**

- Melvor Idle → Stardew Valley
- Measuring episodes to competence
- Testing negative transfer effects

**Blog series:** [GameMaster AI](/series/gamemaster-ai/)

**GitHub:** [gamemaster-ai](https://github.com/JDBentley/gamemaster-ai)

---

## Conference Talks

Target 2027:

- BSides Knoxville
- BSides Nashville
- BSides Charleston
- Hac kSpace Con
- Hack Red Con

*First talk proposal planned for BSides Knoxville 2027 based on the Flipper Zero build series.*

---

## Open Source

Everything built on this blog is documented publicly on GitHub. The goal is to contribute something back to the community that's been generous with knowledge and mentorship.

- [GitHub](https://github.com/JDBentley)