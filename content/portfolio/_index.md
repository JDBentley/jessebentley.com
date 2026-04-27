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

### Flipper Zero Physical Security Suite

A long term open source physical security assessment toolkit built
from scratch on the Flipper Zero platform. No pre-built firmware —
every feature written, documented, and explained line by line.

**Hardware — v1.0 (current)**

- Flipper Zero (main device)
- ESP32 WiFi Dev Board
- ESP32-CAM with 940nm IR LED array
- NRF24L01+PA+LNA 2.4GHz module
- NEO-6M GPS module
- Elecrow CrowPanel 3.5" LVGL touchscreen
- SentinelBoard — custom open source expansion PCB
- Custom 3D printed PETG enclosure

**Hardware — v2.0 (planned)**

- Integrated ESP32-S3 (replaces WiFi Dev Board)
- Adds Bluetooth 5 LE scanning capability
- CC1352R dual band radio (Sub-GHz + 2.4GHz)
- Single unified board replaces all modules
- JLCPCB manufactured, CERN Open Hardware licensed

**Detection Capabilities**

- WiFi device scanner with MAC classification
- BLE device and tracker detection (AirTags, cameras)
- Sub-GHz RF spectrum analyzer
- IR lens detector (hidden camera illumination)
- NFC/RFID vulnerability assessment
- 2.4GHz device scanning (NRF24L01)
- Deauth attack detector
- Evil twin access point detector
- Probe request logger (passive)
- Captive portal detector

**Intelligence Layer**

- Three tier MAC classification (whitelist, blacklist, greylist)
- Pre-loaded camera manufacturer OUI database
- Live editable lists via Flipper, CrowPanel, or companion app
- Baseline and delta scanning (temporal analysis engine)
- Risk scoring system (0-100 per scan)
- Device fingerprinting beyond OUI lookup
- GPS stamped scan logging
- Automated PDF report generation

**Interface**

- Flipper Zero screen and buttons (field use)
- CrowPanel 3.5" LVGL touch dashboard (companion display)
- Raspberry Pi base station with 5" display (desk unit)
- Python companion app (map visualization, reporting)
- Shodan and CVE database integration (companion app)

**Timeline**

- Phase 1 complete — SDK and Hello World ✅
- Phases 2-10 — Core detection features (2026)
- Phases 11-18 — Intelligence and advanced features (2026)
- Phases 19-21 — Reporting, PCB, and enclosure (early 2027)
- Phases 22-24 — Research tier and ML layer (2027)

**Blog series:** [Flipper Zero Build](/series/flipper-zero-build/)

**GitHub:** [flipper-zero-security-suite](https://github.com/JDBentley/flipper-zero-security-suite)

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