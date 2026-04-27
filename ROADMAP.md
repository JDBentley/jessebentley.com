# SentinelFlip Roadmap

Complete development roadmap for the Flipper Zero Physical Security
Suite. Each phase builds on the previous. Every phase is documented
in real time at [jessebentley.com](https://jessebentley.com/series/flipper-zero-build/).

---

## Current Status

| Phase | Description | Status |
|---|---|---|
| Phase 1 | SDK setup and Hello World | ✅ Complete |
| Phase 2 | WiFi device scanner | 🔄 In progress |

---

## Foundation (Phases 1-5)

### Phase 1 — SDK Setup and Hello World ✅
**Hardware:** Flipper Zero, USB-C cable
**Languages:** C
**Goal:** Development environment working end to end

- Install ufbt on Ubuntu 24.04
- Understand Flipper app structure
- Write first C app displaying text on screen
- App running on real hardware

**Blog post:** [Phase 1 — SDK Setup and Hello World](https://jessebentley.com/posts/flipper-zero-phase-1-sdk-setup-and-hello-world/)

---

### Phase 2 — WiFi Device Scanner 🔄
**Hardware:** Flipper Zero, ESP32 WiFi Dev Board
**Languages:** C (Flipper), Arduino C++ (ESP32)
**Goal:** Detect WiFi devices including hidden cameras

- Flash WiFi Marauder firmware to ESP32
- UART communication between Flipper and ESP32
- Scan nearby WiFi devices
- Display device list on Flipper screen
- MAC address lookup against camera OUI database
- Flag suspicious devices

---

### Phase 3 — Sub-GHz RF Analyzer ⬜
**Hardware:** Flipper Zero (CC1101 built in)
**Languages:** C
**Goal:** Analyze Sub-GHz RF spectrum

- Scan 300-928MHz frequency range
- Visualize signal strength
- Detect transmissions in common camera frequencies
- Log detected signals with timestamps

---

### Phase 4 — IR Scanner ⬜
**Hardware:** Flipper Zero, 940nm IR LEDs, ESP32-CAM
**Languages:** C, Arduino C++
**Goal:** Detect hidden camera lenses via IR illumination

- Wire IR LED array to ESP32-CAM GPIO
- Calculate correct resistor values
- ESP32-CAM illuminates room with IR
- Camera feed analyzed for lens reflections
- Alert on potential camera detection

---

### Phase 5 — NFC/RFID Analyzer ⬜
**Hardware:** Flipper Zero (NFC built in)
**Languages:** C
**Goal:** Assess NFC/RFID vulnerabilities

- Read NFC and RFID cards
- Identify card type and encryption
- Flag vulnerable card types (Mifare Classic)
- Log card data for assessment reporting

---

## Intelligence Layer (Phases 6-12)

### Phase 6 — CrowPanel LVGL UI ⬜
**Hardware:** Elecrow CrowPanel 3.5"
**Languages:** C++, LVGL
**Goal:** Rich touch interface for scan results

- Set up LVGL on CrowPanel ESP32
- UART communication with Flipper
- Real time scan results display
- Color coded threat indicators
- Touch navigation between scan modes

---

### Phase 7 — MAC Whitelist/Blacklist System ⬜
**Hardware:** Flipper Zero, SD card
**Languages:** C, Python
**Goal:** Persistent device classification

- SD card file storage (whitelist.txt, blacklist.txt)
- Pre-loaded camera manufacturer OUI database
- Three tier classification (whitelist, blacklist, greylist)
- Live editing via Flipper UI
- Live editing via CrowPanel touch interface
- Live editing via Python companion app
- Sync across all interfaces

---

### Phase 8 — Baseline and Delta Scanning ⬜
**Hardware:** All previous
**Languages:** C, Python
**Goal:** Temporal analysis — detect what changed

- Save baseline scan on arrival
- Compare subsequent scans to baseline
- Alert on new devices appearing
- Alert on devices disappearing
- Time stamped change log
- This is the feature that catches
  cameras that only activate on motion

---

### Phase 9 — Risk Scoring System ⬜
**Hardware:** All previous
**Languages:** C, Python
**Goal:** Single number assessment of scan results

- 0-100 risk score per scan
- Factors: unknown devices, blacklisted MACs,
  deauth attacks, evil twins, open networks
- Color coded: green/yellow/red
- Displayed on Flipper and CrowPanel
- Included in PDF reports

---

### Phase 10 — Deauth and Evil Twin Detection ⬜
**Hardware:** Flipper Zero, ESP32 WiFi Dev Board
**Languages:** C, Arduino C++
**Goal:** Detect active wireless attacks

- Monitor for deauthentication frames
- Identify duplicate SSIDs (evil twin)
- Alert immediately on detection
- Log attacking MAC address
- Flag in risk score

---

### Phase 11 — Device Fingerprinting ⬜
**Hardware:** All previous
**Languages:** C, Python
**Goal:** Identify device type beyond OUI lookup

- Analyze beacon frame characteristics
- Distinguish phones from cameras from IoT
- Build device type signatures
- More accurate than OUI alone
- Reduces false positives

---

### Phase 12 — Probe Request Logger ⬜
**Hardware:** Flipper Zero, ESP32 WiFi Dev Board
**Languages:** C, Arduino C++
**Goal:** Passive device history tracking

- Capture probe requests from all nearby devices
- Log SSIDs each device has connected to previously
- Passive — no interaction with targets
- Reveals device history without active scanning

---

## Interface Layer (Phases 13-15)

### Phase 13 — Python Companion App ⬜
**Hardware:** Raspberry Pi 3B, 5" HDMI display
**Languages:** Python
**Goal:** Desktop base station for scan management

- USB serial connection to Flipper
- Map visualization of scan history
- GPS plotted detections
- Whitelist/blacklist management
- Shodan API integration
- CVE database lookup
- PDF report generation

---

### Phase 14 — GPS Location Logging ⬜
**Hardware:** NEO-M8N GPS module
**Languages:** C, Python
**Goal:** Stamp every detection with coordinates

- GPS fix acquisition
- Coordinate stamped scan logs
- Map visualization in companion app
- Location history across multiple scans
- Useful for tracking repeated detections

---

### Phase 15 — Stealth Mode and Auto Scan ⬜
**Hardware:** All previous
**Languages:** C
**Goal:** Silent operation for field use

- Minimize screen brightness
- Disable audio alerts
- Silent haptic alerts only
- Scheduled auto scan every X minutes
- Alert on any changes while running
- Leave overnight in hotel room

---

## Advanced Features (Phases 16-20)

### Phase 16 — RF Spectrum Analyzer ⬜
**Hardware:** Flipper Zero, NRF24L01
**Languages:** C
**Goal:** Continuous RF monitoring with baseline

- Build RF baseline on arrival
- Alert on new transmissions
- Catches motion-activated cameras
  that only transmit when triggered
- Time based analysis not point in time

---

### Phase 17 — Captive Portal and Network Auditor ⬜
**Hardware:** ESP32 WiFi Dev Board
**Languages:** Arduino C++
**Goal:** Identify network security risks

- Detect captive portals
- List open unencrypted networks
- WPA version checker
- Flag WEP and WPA networks
- Document risk for reporting

---

### Phase 18 — PDF Report Generator ⬜
**Hardware:** Raspberry Pi 3B
**Languages:** Python
**Goal:** Professional client-ready output

- Automated report from scan session
- All devices found and classified
- Risk score with breakdown
- GPS locations on map
- Suspicious devices highlighted
- Recommended actions per finding
- Professional enough to hand to a client

---

### Phase 19 — Shodan and CVE Integration ⬜
**Hardware:** Raspberry Pi 3B
**Languages:** Python
**Goal:** Automated vulnerability context

- Shodan API lookup for identified devices
- NVD CVE database lookup by device type
- Known vulnerabilities surfaced automatically
- Included in PDF reports
- Transforms scanner into vulnerability assessor

---

### Phase 20 — Wigle Integration ⬜
**Hardware:** Raspberry Pi 3B
**Languages:** Python
**Goal:** Network history and location intelligence

- Look up detected networks in Wigle database
- See if networks have been observed elsewhere
- Identify mobile surveillance rigs
- Historical context for detected devices

---

## Hardware Design (Phases 21-22)

### Phase 21 — SentinelBoard v1.0 ⬜
**Goal:** Custom open source expansion PCB

- KiCad schematic and PCB layout
- All modules on single board
- Built in power management and protection
- ESD protection on all GPIO lines
- UART multiplexer for multiple serial devices
- MicroSD, RTC, buzzer, vibration motor
- Onboard LiPo charging (TP4056)
- JLCPCB manufactured
- CERN Open Hardware License
- Design files open sourced on GitHub

---

### Phase 22 — SentinelBoard v2.0 ⬜
**Goal:** Integrated ESP32-S3 replaces WiFi Dev Board

- ESP32-S3-WROOM-1U integrated
- WiFi + Bluetooth 5 LE on board
- Adds BLE camera and tracker detection
- CC1352R dual band radio
- Single unified board
- JLCPCB PCBA assembly service
- DEF CON 2027 hardware

---

## Research Tier (Phases 23-24)

### Phase 23 — Temporal Analysis Engine ⬜
**Goal:** Pattern recognition beyond delta scanning

- Behavioral anomaly detection
- Device appearance time patterns
- Signal strength change analysis
- Motion triggered transmission detection
- Machine learning anomaly scoring
- The feature that catches threats
  point-in-time scanning misses entirely

---

### Phase 24 — Machine Learning Layer ⬜
**Goal:** Reduce false positives, improve detection

- On-device or companion app ML inference
- Train on known camera device signatures
- Anomaly scoring for unknown devices
- False positive reduction over time
- Model improves with each scan session

---

## Contributing

This project is open source and welcomes contributions:

- Additional camera manufacturer OUI entries
- Regional device signatures
- Bug fixes and improvements
- Hardware design review
- Documentation improvements

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines (coming soon).

---

## License

Software: [GPL-3.0](LICENSE)
Hardware (when released): CERN Open Hardware License v2

---

*Built by Jesse Bentley (Cerd) — [jessebentley.com](https://jessebentley.com)*