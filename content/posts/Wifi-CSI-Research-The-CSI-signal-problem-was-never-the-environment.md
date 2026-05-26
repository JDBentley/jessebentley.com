+++
date = '2026-05-25T00:00:00-04:00'
draft = false
title = 'Wifi CSI Research: The CSI Signal Problem Was Never the Environment — It Was My Firmware'
description = 'Fixing a misleading CSI amplitude pattern on the ESP32-C6 by auditing frame acquisition, RX control metadata, and buffer filtering instead of blaming the RF environment.'
author = 'Jesse Bentley'
tags = ['csi', 'esp32-c6', 'wifi-sensing', 'firmware', '802.11', 'signal-analysis']
categories = ['wifi-csi-research']
series = ['wifi-csi-sensing']
showToc = true
tocopen = false
+++

## Background

Several sessions ago I started collecting baseline CSI captures from my desk setup:
- ESP32-C6
- home router
- controlled ping traffic
- no intentional movement

Boring by design.

A stable baseline matters because every future sensing claim depends on it. Before movement detection means anything, the “nothing is happening” state has to behave predictably.

Instead, the baseline looked wrong.

Across three separate runs, the amplitude distribution repeatedly split into two distinct clusters:
- a high cluster around ~45
- a low cluster around ~24

Same desk. Same router. Same environment.

The consistency made it look real.

That was the problem.

The question driving this session became:

> Is the bimodality an actual RF phenomenon, or is the measurement itself wrong?

That distinction matters more than the sensing result.

---

## Methodology

### Hardware

- ESP32-C6 development board
- Home 2.4GHz Wi-Fi network
- Ubuntu analysis workstation

### Traffic Generation

Controlled packet flow was generated using:

```bash
ping -i 1 -s 64 -c 300 <ESP_IP>
```

This produced:
- one ICMP packet per second
- 300 packets total
- predictable CSI callback timing

### v0.1.x Firmware Configuration

The original firmware configuration enabled all seven CSI acquisition flags simultaneously.

The logging pipeline only recorded:
- timestamp
- RSSI
- `data->len`

The firmware did **not** record:
- `rx_state`
- `n_csi_bytes`
- frame format metadata

This meant all CSI frames were averaged together regardless of buffer structure or frame type.

### v0.2.0 Firmware Configuration

The firmware was rebuilt after auditing:
- `esp_wifi_he_types.h`
- `esp_wifi_rxctrl_t`
- ESP32-C6 CSI acquisition behavior

Acquisition was constrained to:

```c
acquire_csi_legacy = true
acquire_csi_ht20 = true
```

Analysis filtering was also added:

```text
rx_state == 0
AND
n_csi_bytes == 128
```

### Dataset Comparison

#### v0.1.x datasets
- 3 desk baseline runs
- ~322–347 packets each
- 1013 packets total

#### v0.2.0 dataset
- `desk_run_04_esp32.csv`
- 322 packets
- 299 seconds
- 315 clean packets after filtering

Getting v0.2.0 to compile required three separate correction cycles due to ESP32-C6 struct inconsistencies.

That turned out to matter more than expected.

---

## What I Expected

Three possible explanations existed for the bimodal distribution.

### Hypothesis 1 — Antenna Diversity

The radio might be switching between antennas, producing two distinct amplitude characteristics.

This was testable through the `rxctrl` metadata.

### Hypothesis 2 — AGC / Receiver Gain Variation

Automatic gain control might be adjusting receiver sensitivity between packets, artificially creating amplitude jumps.

This was testable by correlating amplitude against `noise_floor`.

### Hypothesis 3 — Mixed Frame Types

Different 802.11 frame formats might produce different CSI buffer schemas, all being averaged together incorrectly.

This was testable by:
- constraining acquisition flags
- filtering by `n_csi_bytes`
- auditing frame metadata directly

---

## Results

### Antenna Diversity Eliminated

The ESP32-C6 has a single RF chain.

The `esp_wifi_rxctrl_t` struct contains no antenna field because there is no antenna switching behavior to report.

That hypothesis is closed.

---

### AGC / Noise Floor Correlation Eliminated

`noise_floor` remained effectively constant across the entire v0.2.0 run:

```text
-96 dBm → -97 dBm
```

No meaningful correlation existed between amplitude shifts and receiver noise floor behavior.

AGC variation does not explain the bimodality.

---

### Mixed Frame Types Confirmed as Root Cause

This turned out to be the actual problem.

The original v0.1.x firmware was collecting CSI from multiple frame schemas simultaneously:
- 128-byte CSI buffers
- 256-byte CSI buffers

The averaging logic treated both as equivalent.

That assumption was wrong.

The 256-byte schema appears to include:
- guard bands
- null subcarriers
- near-zero values that were not actually signal energy

Those values passed the naive filter:

```text
amplitude > 0
```

and artificially dragged the average downward.

The low cluster (~24) is most consistent with a firmware acquisition artifact rather than environmental signal behavior.

The high cluster (~45) was the actual signal.

No evidence currently suggests the bimodality originated from the RF environment.

---

## Results — v0.1.x vs v0.2.0

![v0.1.x bimodal vs v0.2.0 unimodal distribution](/images/wifi-csi/2026-05-26/01_vs_02_baseline_distribution_2026-05-26.png)

*Side-by-side histogram comparison: v0.1.x bimodal amplitude distribution (left) versus v0.2.0 unimodal distribution (right). Same desk, same router, same environment. The difference is entirely acquisition methodology and filtering.*

### Statistical Comparison

#### v0.1.x
- Mean amplitude: 42.6
- Standard deviation: 8.43

#### v0.2.0
- Mean amplitude: 43.8
- Standard deviation: 4.51

> Firmware correction alone reduced baseline standard deviation by roughly 46%.
>
> Same desk. Same router. Same environment.
>
> The difference was acquisition methodology.

---

### Multi-Run v0.1.x Bimodality

![Three separate v0.1.x baseline runs showing bimodal behavior](/images/wifi-csi/2026-05-26/desk_baseline_bimodality_2026-05-26.png)

*Three separate v0.1.x desk baseline runs showing the same bimodal amplitude pattern repeatedly. The artifact was consistent enough to initially appear like legitimate environmental signal behavior.*

This is the dangerous part.

The artifact was repeatable.

Repeatability alone is not proof of correctness.

---

### Link Negotiation Discovery

The ESP32-C6 associated as 802.11n rather than Wi-Fi 6.

The boot log confirmed:

```text
phytype CBW20-SGI
phymode 11bgn
he:0
vht:0
ht:1
```

![ESP32-C6 boot log showing 802.11n negotiation](/images/wifi-csi/2026-05-26/c6_link_negotiates_11n_not_he_2026-05-26.png)

*Boot log confirming the ESP32-C6 negotiated 802.11n (`ht:1`, `he:0`) rather than Wi-Fi 6 with the home router. Wi-Fi 6 capable hardware does not guarantee HE frame acquisition in practice.*

This also explained why earlier HE-only acquisition attempts produced zero captures.

The router simply was not negotiating HE frames.

---

### Firmware Instrumentation Audit

The original firmware instrumentation attempted to log fields that do not exist in the ESP32-C6 RX control layout.

![Firmware instrumentation changes between v0.1.x and v0.2.0](/images/wifi-csi/2026-05-26/firmware_diff_csi_acquisition_2026-05-26.png)

*Original firmware instrumentation attempting to log nonexistent C6 RX control fields (`sig_mode`, `mcs`, `cwb`, `ant`) before auditing the actual ESP32-C6 struct layout.*

After auditing the actual struct definitions, the logging pipeline was rebuilt around the correct metadata fields.

![Corrected ESP32-C6 RX control instrumentation for v0.2.0](/images/wifi-csi/2026-05-26/Screenshot%20from%202026-05-26%2000-58-57.png)

*Corrected v0.2.0 instrumentation using the actual ESP32-C6 RX control layout. Added `cur_bb_format`, `rx_state`, and `n_csi_bytes` validation to separate legitimate signal behavior from acquisition artifacts.*

The actual firmware changes were small.

The effect on data quality was not.

---

## Analysis

The most important finding here is methodological, not environmental.

The original v0.1.x captures looked internally consistent:
- repeated runs
- repeated clustering
- stable packet counts
- normal RSSI values

Nothing appeared obviously broken.

The problem only surfaced after auditing what the firmware was actually collecting versus what I assumed it was collecting.

That distinction matters because this type of error survives informal validation extremely well.

The data looked believable.

It was consistently wrong.

This also exposed an important ESP32-C6 implementation detail:

The C6's CSI acquisition structs and RX control structs do not follow a perfectly synchronized versioning model.

The acquisition config:
- lacks several fields documented elsewhere

The RX control layout:
- follows the newer v3-MAC structure

Assuming both followed the same revision model produced multiple compile failures during v0.2.0 development.

The lesson is simple:

Do not reason about firmware structures from memory.

Read the exact header implementation for the exact chip revision being used.

---

## Limitations

- v0.2.0 movement validation has not yet been completed
- Testing occurred in a single desk environment
- Only one router and one RF topology were evaluated
- Controlled ping traffic produces cleaner timing than real ambient traffic
- Residual degraded packets (`rx_state = 198`) still occur at low frequency
- v0.1.x datasets cannot be perfectly re-filtered retroactively because required metadata was never logged

This session solved a measurement problem.

It did not yet solve motion sensing.

---

## Security Implications

This finding matters beyond this project.

CSI-based sensing systems that compute variance thresholds from improperly filtered frame schemas may be calibrating against firmware artifacts instead of environmental signal behavior.

That creates a measurement-layer problem:
- the detector itself may not be wrong
- the input distribution may already be corrupted

An attacker who understands frame-format behavior could potentially manipulate traffic composition to alter observed CSI characteristics before detection logic even executes.

More broadly:

CSI research that does not publish:
- firmware versions
- acquisition flags
- filtering rules
- frame schema assumptions

cannot be meaningfully reproduced.

The acquisition methodology is part of the result.

Treating it as implementation detail is a validity problem.

---

## GitHub

Firmware:
- `firmware/esp32-c6/csi_capture/main/csi_capture.c`
- tagged `v0.2.0-csi-rxctrl`

Raw data:
- `data/raw/desk_active/desk_run_04_esp32.csv`

Historical comparison data:
- `data/raw/desk_baseline_v01/`

Analysis:
- `analysis/`

---

## Future Work

Immediate next step:

Capture a v0.2.0 movement dataset under the corrected acquisition pipeline and determine whether the previously observed movement separation still holds under the lower-variance baseline.

After that:
- kitchen placement testing
- office environment testing
- RF-dense environment validation
- ambient traffic acquisition validation
- normalization pipeline development

The important shift is this:

The project is no longer asking:
> “Can I collect CSI?”

It is now asking:
> “Can I trust the CSI I collected?”
