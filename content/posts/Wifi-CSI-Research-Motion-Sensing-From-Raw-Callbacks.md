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

# From Raw Callbacks to Reproducible Datasets

## Background

The question driving this research block was straightforward:

> Can WiFi Channel State Information (CSI) produce measurable and repeatable differences between baseline and movement conditions?

Earlier sessions confirmed the ESP32-C6 could successfully capture CSI data under directed traffic conditions. The acquisition pipeline worked. Structured datasets existed.

What was still unknown was whether movement actually produced a measurable signal difference — and whether that difference remained consistent across repeated captures.

That distinction matters.

It is easy to generate noisy RF data. It is much harder to determine whether the observed variation is:
- movement-induced
- environmentally driven
- statistically repeatable
- or simply random RF instability

Most CSI sensing demonstrations stop at “the graph changed.”

This session focused on moving beyond that.

The goal was not proving detection. The goal was determining whether the signal behavior itself was stable enough to justify continued sensing research.

---

## Methodology

### Hardware

- ESP32-C6 (sensor node)
- Ubuntu laptop (`cerd-Latitude-7640`)
- MiFi hotspot for isolated directed traffic generation

### Firmware

The ESP32-C6 was running previously developed CSI capture firmware built on ESP-IDF v6.1-dev.

The callback exports:
- timestamp
- RSSI
- packet length
- first 16 CSI subcarrier values

### Traffic Generation

Reliable CSI acquisition required active packet flow directed at the ESP32-C6.

Directed ICMP traffic was generated using:

```bash
ping <ESP_IP>
```

This maintained stable CSI callback execution during all captures.

### Dataset Collection

Four datasets were captured across two separate runs:

- `baseline_01.csv`
- `movement_01.csv`
- `baseline_02.csv`
- `movement_02.csv`

Baseline captures:
- no intentional movement
- static environment
- directed traffic active

Movement captures:
- physical movement introduced during acquisition
- same traffic generation conditions maintained

### Analysis Pipeline

The analysis environment used:
- Python virtual environment
- pandas
- matplotlib

The processing pipeline was expanded to support:
- multi-run comparison
- variance calculations
- standard deviation comparison
- repeatability analysis

The project structure also evolved into a more formalized analysis workflow:

![Structured CSI dataset hierarchy and analysis workflow](/images/wifi-csi/2026-05-18/27-structured-dataset-hierarchy.png)
*Structured separation between baseline, movement, passive, assisted, and processed datasets. The project is transitioning from raw acquisition into repeatable analysis.*

---

## What I Expected

The initial expectation was straightforward:

Movement captures should produce:
- higher variance
- higher signal energy
- higher standard deviation

compared to baseline captures.

What I did not expect was how unstable the baseline itself would become between runs.

The assumption going into this session was that static-environment captures would reproduce relatively cleanly under similar conditions.

That assumption turned out to be wrong.

That failure became the most important finding from this testing block.

---

## Results

### Initial Statistical Comparison

The first analysis pass compared a single baseline capture against a movement capture.

The difference appeared immediately.

![Baseline versus movement statistical comparison output](/images/wifi-csi/2026-05-18/26-first-csi-statistics-output.png)
*Initial statistical comparison between baseline and movement captures. Movement immediately produces substantially higher variance and standard deviation.*

Initial statistical output:

```text
=== Baseline Statistics ===
Mean Energy: 625.68
Variance: 5082.34
Standard Deviation: 71.29
Minimum Energy: 514.00
Maximum Energy: 836.00

=== Movement Statistics ===
Mean Energy: 971.94
Variance: 253394.78
Standard Deviation: 503.38
Minimum Energy: 158.00
Maximum Energy: 1985.00
```

Movement increased:
- variance
- energy fluctuation
- standard deviation

substantially beyond the initial baseline capture.

At first glance, this looked promising.

The problem appeared during repeatability testing.

---

### Cross-Run Repeatability Analysis

A second set of baseline and movement captures was collected under nominally similar conditions.

The results introduced a new problem.

```text
=== Baseline 1 ===
Rows: 50
Mean Energy: 625.68
Variance: 5082.34
Standard Deviation: 71.29

=== Baseline 2 ===
Rows: 49
Mean Energy: 2060.16
Variance: 112590.43
Standard Deviation: 335.54

=== Movement 1 ===
Rows: 51
Mean Energy: 971.94
Variance: 253394.78
Standard Deviation: 503.38

=== Movement 2 ===
Rows: 49
Mean Energy: 1979.94
Variance: 435529.93
Standard Deviation: 659.95

[*] Repeatability Summary
Average Baseline Std Dev: 203.42
Average Movement Std Dev: 581.67
Movement/Baseline Std Dev Ratio: 2.86x
```

![Cross-session repeatability statistics across baseline and movement captures](/images/wifi-csi/2026-05-18/29-repeatability-statistics-output.png)
*Repeatability analysis across two baseline and two movement runs. Movement captures averaged roughly 2.86x higher standard deviation than baseline captures despite substantial environmental drift.*

Movement still produced higher instability overall.

But the second baseline capture drifted dramatically compared to the first.

Baseline 1 standard deviation:
```text
71.29
```

Baseline 2 standard deviation:
```text
335.54
```

That represents roughly a 4.7x increase in baseline instability between two supposedly static-environment captures.

No intentional movement occurred during either baseline run.

That became the real finding.

---

## Analysis

Movement consistently increased CSI instability across every capture set.

Movement datasets showed:
- higher variance
- larger energy fluctuation
- higher standard deviation

This aligns with the expected RF behavior:
human movement alters multipath propagation paths, and CSI reflects those changes.

That portion of the hypothesis held.

The baseline drift did not.

The second baseline capture approached the instability level of the first movement capture.

That matters because sufficiently noisy baselines can overlap with low-activity movement conditions.

If that overlap becomes large enough:
- movement detection becomes unreliable
- false positives increase
- environmental drift masks actual movement signatures

The likely causes are operational rather than exotic:
- neighboring RF traffic changes
- hotspot behavior variation
- antenna orientation changes
- body positioning differences
- multipath environmental fluctuation
- hotel RF congestion

Any of these factors can materially alter CSI output.

This introduces an important constraint:

Raw CSI instability alone is probably insufficient for reliable sensing.

Environmental normalization will likely be required before movement detection claims become defensible.

That finding is more important than simply producing “different graphs.”

---

## Limitations

- Testing occurred in an uncontrolled hotel RF environment
- Device orientation was not rigidly controlled
- Only two repeated runs per condition exist
- No normalization pipeline has been implemented
- No false positive filtering currently exists
- Passive sensing was not evaluated in this session
- Directed traffic remained required for stable acquisition

This is still early-stage characterization work, not a finalized sensing system.

---

## Security Implications

WiFi CSI sensing has direct physical security relevance.

Potential applications include:
- occupancy inference
- perimeter sensing
- covert presence detection
- RF-assisted environmental awareness

The baseline drift finding cuts both ways.

From a defensive perspective:
- environmental RF instability complicates reliable detection

From an offensive or evasion perspective:
- sufficiently noisy RF environments may naturally mask movement signatures

The repeatability findings also reinforce an important operational reality:

CSI sensing is not simply “movement changes the graph.”

The environment itself changes the graph continuously.

Understanding that distinction is critical before stronger sensing claims become credible.

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

The next phase is not collecting more random captures.

The next phase is solving the baseline drift problem.

The core question now becomes:

> How do I distinguish movement-induced CSI instability from environmental RF drift?

Immediate next steps:
- build normalization pipeline
- establish rolling baseline correction
- compare normalized movement signatures across sessions
- define provisional noise floor thresholds
- evaluate false positive reduction after normalization
- expand repeatability testing beyond two runs

Until environmental drift is understood and controlled, movement detection claims remain premature.