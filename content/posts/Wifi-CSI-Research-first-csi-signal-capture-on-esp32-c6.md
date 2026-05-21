+++
date = '2026-05-11T00:00:00-04:00'
draft = false
title = 'Wifi CSI Research: First CSI Signal Capture on ESP32-C6'
description = 'From firmware errors to structured datasets — getting the first real CSI signal out of an ESP32-C6 and learning what reliable acquisition actually requires.'
author = 'Jesse Bentley'
tags = ['wifi-csi', 'esp32', 'firmware', 'esp-idf', 'signal-processing', 'research']
categories = ['wifi-csi-research']
series = ['wifi-csi-research']
showToc = true
tocopen = false
+++

## Background

WiFi Channel State Information (CSI) captures how a wireless signal changes as it travels between a transmitter and receiver. Every object in that path be it walls, furniture, or people moving through a room they alter the signal in ways that appear inside the CSI data.

The goal of this research track is determining whether those changes are detectable, repeatable, and usable in a physical security context.

Before any sensing or analysis becomes possible, the ESP32-C6 first needs to reliably produce CSI output at all.

These two sessions on May 2nd and May 8th, cover that entire path: from the first failed firmware build through structured, reproducible dataset generation.

---

## Methodology

**Hardware and environment:**
- Ubuntu laptop (`cerd-Latitude-7640`)
- ESP32-C6 connected via USB
- ESP-IDF v6.1-dev loaded through a custom `get_idf` alias
- Firmware project at `wifi-csi-research/firmware/esp32-c6/csi_capture`
- MiFi hotspot used for stable directed traffic testing in later sessions

**Experiment structure across both sessions:**

Session one (May 2nd) focused on getting any CSI output at all:
- Resolving firmware build errors
- Correcting HAL struct mismatches
- Verifying the CSI callback executes

Session two (May 8th) focused on stabilizing acquisition and transitioning from raw callback validation into structured, timestamped data suitable for future analysis.

The callback was expanded to capture:
- Timestamps
- RSSI
- Packet length
- First 16 CSI subcarrier values per sample

Directed traffic was generated using:

```bash
ping
```

Two datasets were captured:
- Baseline dataset — directed traffic only
- Movement dataset — physical movement during acquisition

---

## What I Expected

- ESP32-C6 connects to WiFi and the CSI callback fires once traffic exists
- Ambient network activity would be enough to generate continuous CSI
- Output would be immediately readable and consistent
- Baseline and movement captures would show visible differences early

---

## Results

### Session one — getting the callback to fire

The first build failed immediately.

The firmware used the older `wifi_csi_config_t` struct, which the ESP32-C6 HAL does not support. The correct type for this chip is `wifi_csi_acquire_config_t`.

Every field in the configuration struct was rejected by the compiler.

![CSI config struct mismatch — multiple unrecognized fields flagged by the compiler](/images/wifi-csi/2026-05-08/11-csi-config-struct-build-error.png)
*Every CSI config field rejected. Wrong struct type for the ESP32-C6 HAL.*

After correcting the struct type, the firmware built successfully.

![Successful build output for ESP32-C6 CSI firmware](/images/wifi-csi/2026-05-08/12-c6-csi-config-build-success.png)
*Clean build after struct correction. Binary generated and ready to flash.*

First CSI output after flashing confirmed the callback was executing:

```text
CSI len: 128
CSI len: 128
CSI len: 128
```

![Serial monitor showing repeated CSI len: 128 output](/images/wifi-csi/2026-05-08/13-first-csi-output.png)
*Callback firing. Length confirmed. No values yet — but the acquisition pipeline is alive.*

---

### Session two — expanding the callback and generating datasets

With the callback confirmed, the firmware was expanded to produce structured output.

The ESP-IDF alias workflow was validated before building.

![idf.py flash monitor running, alias workflow confirmed](/images/wifi-csi/2026-05-08/15-idf-alias-working.png)
*Environment loading cleanly via alias. No path issues.*

A second build failure appeared when IP retrieval code was placed at file scope instead of inside a function.

C does not allow runtime function calls during static initialization.

![Build error: initializer element is not constant](/images/wifi-csi/2026-05-08/18-ip-code-outside-function-build-error.png)
*`esp_netif_get_handle_from_ifkey` called outside a function. Runtime call placed in static initialization.*

After moving the call into the correct function scope, the build succeeded.

Expanded CSI output confirmed the callback was now producing structured signal data with measurable variation.

![Expanded CSI output showing timestamp, RSSI, and first 16 subcarrier values](/images/wifi-csi/2026-05-08/16-expanded-csi-variation.png)
*Timestamps, RSSI, and subcarrier values captured successfully.*

WiFi association completed successfully on the device.

![WiFi association log showing full connection lifecycle](/images/wifi-csi/2026-05-08/16-wifi-connected.png)
*Full WiFi association lifecycle visible in logs — auth, assoc, run.*

Directed ping traffic to the ESP32-C6 produced stable, continuous CSI generation.

![Ping output showing consistent ICMP replies from ESP32-C6](/images/wifi-csi/2026-05-08/21-ping-to-esp-success.png)
*Consistent ICMP replies from the ESP32-C6 during acquisition.*

![CSI output generated from directed ping traffic](/images/wifi-csi/2026-05-08/22-csi-from-directed-traffic.png)
*Continuous CSI generation during directed traffic.*

Cleaned baseline and movement datasets were then captured.

![Cleaned CSI datasets — baseline and movement captures](/images/wifi-csi/2026-05-08/24-cleaned-csi-datasets.png)
*First reproducible CSI datasets captured under baseline and movement conditions.*

---

## Analysis

The struct mismatch matters beyond simply fixing a compiler error.

The ESP32-C6 uses a different CSI API than older ESP32 variants. Documentation and examples written for the ESP32 or ESP32-S2 do not transfer cleanly. Firmware adapted from older examples needs to be validated directly against the ESP32-C6 HAL headers before implementation.

The traffic dependency finding is more significant.

Initial testing assumed ambient network activity would generate continuous CSI output. It did not.

Passive presence on the network produced intermittent and unreliable captures. Stable CSI acquisition only appeared once the ESP32-C6 became an active participant in packet exchange.

Directed ping traffic changed acquisition behavior immediately.

Once packets were consistently addressed to the ESP32-C6:
- CSI output stabilized
- Capture frequency increased
- Structured output became reproducible

This behavior reflects how CSI extraction works at the PHY layer. The device needs frames addressed to it in order to extract usable channel state information.

RSSI variation across sessions (`-89 → -48 → -60 → -80`) reflects real environmental changes:
- Device orientation
- Distance
- Hotspot topology
- Multipath effects

That variability is expected and is part of what makes CSI potentially useful for environmental sensing.

The movement dataset also shows measurable differences in subcarrier values compared to baseline.

What remains unknown is whether those differences are:
- Consistent
- Repeatable
- Distinguishable from environmental noise

That analysis has not been completed yet.

---

## Limitations

- CSI acquisition currently requires directed traffic
- Passive monitoring was unreliable in this setup
- Dataset cleaning is manual
- ESP monitor logs and CSI output are intermixed in raw captures
- No visualization pipeline exists yet
- No feature extraction has been implemented
- MiFi topology produced more stable traffic behavior than phone hotspot testing
- Physical movement events were not timestamp-correlated during acquisition

---

## Security Implications

The traffic dependency finding cuts both ways.

For sensing applications:
- CSI systems require active packet flow
- Passive listening alone was insufficient in this configuration
- A sensing device must remain an active network participant

That has operational implications for:
- Covertness
- Deployment design
- Reliability

For evasion:
- Interrupting packet flow interrupts CSI acquisition
- No packets to the sensor means no environmental visibility

For pentesting relevance:
- CSI-based sensing systems have identifiable operational dependencies
- Understanding those dependencies is the first step toward understanding failure modes

A sensing system dependent on active ping traffic from a known host is more fragile and more targetable than one capable of passive acquisition.

The broader finding is that CSI sensing is less passive than it initially appears.

That is worth documenting early.

---

## GitHub

[wifi-csi-research](https://github.com/JDBentley/wifi-csi-research)

Relevant files:
- `firmware/esp32-c6/csi_capture/main/csi_capture.c`
- `data/baseline_01.csv`
- `data/movement_01.csv`

---

## Future Work

The next phase moves into the temporal analysis layer.

Immediate priorities:
- Automated dataset parsing
- Separation of CSI data from ESP monitor logs
- Visualization of baseline vs movement captures across time
- Variance and signal energy calculations
- Amplitude delta analysis
- Establishing a measurable noise floor
- Determining where false positives begin appearing

The core question now is whether the differences between baseline and movement captures represent actual signal or environmental noise.

That answer requires statistical analysis and visualization which are both planned for the next session.
