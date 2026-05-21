---
title: "WiFi CSI Research: From Raw Callbacks to Reproducible Datasets"
date: 2026-05-18T00:00:00-04:00
draft: false
tags: ["wifi-csi", "esp32", "rf-sensing", "signal-analysis", "research"]
categories: ["wifi-csi-research"]
series: ["wifi-csi-research"]
showToc: true
tocopen: false
---

# WiFi CSI Motion Sensing: From Raw Callbacks to Reproducible Datasets

## Background

The question driving this research is straightforward:

> Can WiFi Channel State Information (CSI) reliably detect human movement in a real environment?

CSI describes how a wireless signal propagates between a transmitter and receiver across multiple subcarriers. When a person moves through that propagation path, the multipath reflections change. Signal energy shifts. Variance increases.

In theory, that change is measurable.

Most CSI sensing research happens under controlled conditions:
- fixed hardware
- stable RF environments
- repeatable physical layouts

This research does not.

Testing was conducted in a hotel environment using:
- consumer networking hardware
- uncontrolled neighboring RF activity
- an ESP32-C6
- improvised physical placement

The goal of this two-session block was narrower than “solve motion sensing.”

The actual goal was:

> Can I generate structured, reproducible CSI datasets that show measurable differences between baseline and movement conditions?

Before detection claims matter, the signal itself has to be proven.

---

## Methodology

### Hardware

- ESP32-C6 (sensor node)
- Ubuntu laptop (`cerd-Latitude-7640`)
- MiFi hotspot providing isolated directed traffic

### Firmware

The ESP32-C6 runs custom CSI capture firmware built on ESP-IDF v6.1-dev.

The callback exports:
- timestamp
- RSSI
- packet length
- first 16 CSI subcarrier values

### Traffic Generation

CSI acquisition required active packet flow directed at the ESP32-C6.

Directed ICMP traffic was generated using:

```bash
ping <ESP_IP>
```

This kept CSI callbacks firing consistently during capture.

### Dataset Structure

Four CSV datasets were collected across two sessions:

- `baseline_01.csv`
- `movement_01.csv`
- `baseline_02.csv`
- `movement_02.csv`

### Analysis Pipeline

The analysis environment used:
- Python virtual environment
- pandas
- matplotlib

The analysis script was expanded during session two to support:
- multi-run comparison
- repeatability analysis
- cross-session statistical output

---

## What I Expected

Movement captures should produce:
- higher variance
- higher signal energy
- larger standard deviation

compared to baseline captures.

That part was expected.

What I did not expect was baseline instability.

The assumption going into session two was that static-environment captures would reproduce relatively cleanly between runs.

That assumption failed.

That failure became the most important result from this testing block.

I also initially assumed ambient WiFi activity would sustain continuous CSI acquisition.

That assumption failed as well.

---

## Results

### Session 1 — Initial Acquisition

The first obstacle was a CSI API mismatch inside the ESP32-C6 firmware.

The firmware initially used an unsupported CSI configuration struct, causing the build to fail completely.

![Build failure caused by ESP32-C6 CSI configuration struct mismatch](/images/wifi-csi-research/2026-05-18/11-csi-config-struct-build-error.png)
*ESP32-C6 CSI configuration mismatch preventing firmware compilation.*

After correcting the struct mismatch, the firmware built and flashed successfully.

![Successful ESP32-C6 CSI firmware build after struct correction](/images/wifi-csi-research/2026-05-18/12-c6-csi-config-build-success.png)
*Successful firmware build after correcting ESP32-C6 CSI API differences.*

Initial callback output only confirmed execution:

```text
CSI len: 128
```

That proved the callback was firing, but not whether the captured signal was usable.

![Initial CSI callback output](/images/wifi-csi-research/2026-05-18/13-first-csi-output.png)
*Initial CSI callback execution confirmed.*

After expanding the callback to export subcarrier values, measurable variation appeared in the output:

```text
-27,10,-29,8,29,-2,32,0,-6,5,-1,-8
```

RSSI variation was also visible across packets:

```text
-89 → -48 → -60 → -80
```

![Expanded CSI output showing subcarrier variation](/images/wifi-csi-research/2026-05-18/16-expanded-csi-variation.png)
*Structured CSI output showing measurable subcarrier and RSSI variation.*

---

### Traffic Dependency Discovery

Initial testing used:
- passive idle observation
- phone hotspot testing

Neither produced reliable CSI acquisition.

The phone hotspot generated intermittent bursts. Passive network presence produced almost no sustained output.

CSI only stabilized once directed ICMP traffic targeted the ESP32-C6 directly.

![ESP32-C6 connected to MiFi network](/images/wifi-csi-research/2026-05-18/16-wifi-connected.png)
*ESP32-C6 associated to isolated MiFi network.*

![Directed ping traffic from laptop to ESP32-C6](/images/wifi-csi-research/2026-05-18/21-ping-to-esp-success.png)
*Directed ICMP traffic sustaining CSI acquisition.*

![Stable CSI generation during directed traffic](/images/wifi-csi-research/2026-05-18/22-csi-from-directed-traffic.png)
*Stable CSI acquisition under directed traffic conditions.*

A second build failure appeared mid-session after IP retrieval code was incorrectly placed at file scope.

![Build failure from misplaced runtime IP retrieval code](/images/wifi-csi-research/2026-05-18/18-ip-code-outside-function-build-error.png)
*Runtime function call incorrectly placed inside static initialization.*

After correcting the scope issue, structured dataset generation continued successfully.

Cleaned datasets from session one were verified before analysis.

![Session 1 cleaned baseline and movement datasets](/images/wifi-csi-research/2026-05-18/24-cleaned-csi-datasets.png)
*Structured baseline and movement datasets prepared for analysis.*

---

### Session 2 — Repeatability and Statistical Analysis

The analysis pipeline was expanded to compare all four datasets simultaneously.

The project structure evolved into a more formalized analysis workflow:

![Structured CSI dataset hierarchy and analysis workflow](/images/wifi-csi-research/2026-05-18/27-structured-dataset-hierarchy.png)
*Structured separation between baseline, movement, assisted, passive, and processed datasets.*

Initial statistical comparison between baseline and movement captures immediately showed movement increasing signal instability.

![Baseline versus movement statistical comparison output](/images/wifi-csi-research/2026-05-18/26-first-csi-statistics-output.png)
*Initial statistical comparison between baseline and movement captures. Movement immediately produces substantially higher variance and standard deviation.*

Cross-session analysis produced the following results:

```text
=== Baseline 1 ===
Mean Energy: 625.68
Variance: 5082.34
Standard Deviation: 71.29

=== Baseline 2 ===
Mean Energy: 2060.16
Variance: 112590.43
Standard Deviation: 335.54

=== Movement 1 ===
Mean Energy: 971.94
Variance: 253394.78
Standard Deviation: 503.38

=== Movement 2 ===
Mean Energy: 1979.94
Variance: 435529.93
Standard Deviation: 659.95
```

![Cross-session repeatability statistics across baseline and movement captures](/images/wifi-csi-research/2026-05-18/29-repeatability-statistics-output.png)
*Repeatability analysis across two baseline and two movement runs. Movement captures averaged roughly 2.86x higher standard deviation than baseline captures despite substantial environmental drift.*

---

## Analysis

Movement consistently increased CSI instability.

Across both sessions:
- movement datasets produced higher variance
- movement datasets produced higher standard deviation
- movement datasets produced larger energy fluctuations

That aligns with the underlying RF behavior. Human movement changes multipath propagation paths, and CSI captures those changes.

That part worked as expected.

The baseline drift did not.

Baseline 1 standard deviation:

```text
71.29
```

Baseline 2 standard deviation:

```text
335.54
```

That represents roughly a 4.7x increase in baseline noise between two static-environment captures taken in the same location.

No intentional movement occurred during either baseline run.

This matters because Baseline 2 begins approaching Movement 1 instability levels.

If baseline noise continues increasing, sufficiently noisy environments could overlap with low-activity movement captures, making reliable detection significantly harder.

The likely causes are operational rather than exotic:
- neighboring RF traffic changes
- hotspot behavior shifts
- body positioning differences
- antenna orientation
- hotel RF congestion
- multipath environmental changes

Any of these can materially affect CSI output.

The traffic dependency finding is also operationally important.

Reliable CSI acquisition required directed packet flow targeting the ESP32-C6.

Passive network presence alone was insufficient.

That creates practical constraints for:
- covert sensing
- passive monitoring
- low-noise deployments
- adversarial environments

---

## Limitations

- Testing occurred in an uncontrolled hotel RF environment
- Device orientation was not rigidly controlled
- Only two repeated runs per condition exist
- No normalization pipeline has been implemented
- No false positive filtering currently exists
- Passive sensing has not been validated
- CSI acquisition still depends on directed traffic
- Some early debugging stages were not visually documented

---

## Security Implications

WiFi CSI sensing has direct physical security relevance.

Potential applications include:
- covert presence detection
- perimeter movement sensing
- occupancy inference
- RF-assisted environmental awareness

The baseline drift finding cuts both ways.

From a defensive perspective:
- environmental RF instability complicates reliable detection

From an evasion perspective:
- noisy RF environments may naturally mask movement signatures

The traffic dependency finding is equally important.

A sensing system that requires active packet generation:
- has a detectable operational footprint
- depends on sustained traffic flow
- introduces identifiable dependencies an adversary could target

The broader takeaway is that CSI sensing is not as passive or straightforward as early assumptions suggested.

That finding is worth documenting before stronger detection claims are made.

---

## GitHub

Firmware:
- `firmware/esp32-c6/csi_capture/main/csi_capture.c`

Datasets:
- `baseline_01.csv`
- `movement_01.csv`
- `baseline_02.csv`
- `movement_02.csv`

Analysis:
- Python venv
- pandas
- matplotlib

```text
feat: implement reproducible CSI dataset acquisition workflow
```

---

## Future Work

The next phase is not simply collecting more data.

The next phase is answering the question baseline drift introduced:

> How do I distinguish movement-induced CSI instability from environmental RF drift?

Immediate next steps:
- Build normalization pipeline
- Establish rolling baseline correction
- Compare normalized movement signatures across sessions
- Define provisional noise floor thresholds
- Evaluate false positive reduction after normalization
- Expand repeatability testing beyond two runs

Until baseline instability is understood and controlled, movement detection claims remain premature.