+++
date = '2026-05-04T00:00:00-04:00'
draft = false
title = 'WiFi CSI Research: Building a Foundation Before the First Signal'
description = 'Initializing the WiFi CSI research environment, moving firmware into version control, and enabling CSI support in ESP32-C6 before collecting any signal data.'
author = 'Jesse Bentley'
tags = ['wifi-csi', 'esp32-c6', 'firmware', 'security-research', 'rf']
categories = ['wifi-csi-research']
series = ['wifi-csi-sensing']
showToc = true
tocopen = false
+++

# WiFi CSI Research: Building a Foundation Before the First Signal

## Background

The goal of this project is to use WiFi Channel State Information (CSI) to detect physical motion and environmental changes passively — no cameras, no dedicated sensors, only distortion introduced into an existing RF signal.

Before that is possible, the system needs a foundation that supports reproducible research. That means:

* firmware under version control
* a consistent build environment
* CSI explicitly enabled at the hardware level

This post covers the first two sessions. No CSI data was collected. The outcome here is structural — the system can now be built and configured without guessing.

---

## What I Expected

The expectations were straightforward:

* a repository that separates raw and processed data
* firmware inside the repository, version-controlled with everything else
* CSI exposed as a configurable option in ESP-IDF
* a successful build after enabling CSI

None of these are complex individually. Skipping them creates failure modes that are harder to debug later.

---

## What I Ran Into

The first issue showed up immediately: the firmware was not inside the repository.

The ESP32-C6 project existed at:

```text
~/esp/csi_capture
```

That breaks reproducibility. The repository can change without the firmware, and the firmware can change without the repository. There is no single source of truth for the system state.

The fix was straightforward — move the firmware into the repo — but the implication matters:

```text
firmware/esp32-c6/csi_capture/
```

Now the firmware, configuration (`sdkconfig`), and repository history move together.

The second issue was environmental.

Running:

```bash
idf.py menuconfig
```

from the wrong directory produced:

```text
CMakeLists.txt not found
```

This is a good failure. It makes the problem obvious. A misconfigured environment that still builds would be harder to detect.

After re-sourcing ESP-IDF from the correct directory, menuconfig opened as expected.

From there, CSI had to be explicitly enabled.

CSI is disabled by default. If left unchanged, the firmware builds and runs without error but produces no CSI output. That creates a silent failure condition — the system appears operational but yields no signal.

---

## What the Outcome Was

The repository was initialized and committed.

```text
wifi-csi-research/
├── firmware/
├── data/
│   ├── raw/
│   └── processed/
├── analysis/
├── experiments/
├── docs/
├── captures/
├── logs/
```

Commit:

```text
6d6f7bf — chore: initialize wifi csi research repository
```

The firmware was moved into the repository and CSI was enabled.

```text
firmware/esp32-c6/csi_capture/
```

Commit:

```text
0dbdb88 — feat: integrate ESP32-C6 firmware and enable CSI support
```

Verification steps:

* repository structure exists locally and matches the commit
* firmware now exists inside the repository path
* `.gitignore` excludes ESP-IDF build artifacts
* `idf.py set-target esp32c6` reflected in build output
* menuconfig opened successfully after correcting working directory
* CSI option located and enabled
* firmware rebuilt without errors

Evidence:

![Menuconfig UI showing WiFi configuration](/images/wifi-csi/2026-05-02/05-menuconfig.png)

*Menuconfig UI confirming correct configuration context.*

![ESP32-C6 build output confirming target](/images/wifi-csi/2026-05-02/06-build-target.png)

*Build output confirming ESP32-C6 target is applied.*

---

## Analysis

The outcome here is structural, not functional.

CSI being disabled by default introduces a silent failure mode. The firmware compiles and runs, but produces no usable signal. Without explicitly enabling CSI, the system appears correct while doing nothing.

The directory error reinforces the same idea. ESP-IDF depends on correct working directory and environment state. In this case the failure was explicit, which is preferable.

Moving the firmware into the repository closes a reproducibility gap. A clean clone of the repository can now recreate the firmware state, including configuration.

Defining structure before collecting data enforces separation between:

* experiment definition
* captured data
* analysis

Once data starts accumulating, imposing that separation becomes harder.

---

## Limitations

* No CSI data has been collected
* No callback exists to receive CSI events
* CSI signal flow is unvalidated
* ESP32-C6 CSI behavior is untested on this device
* Build environment is not pinned or containerized

The system builds. It does not yet produce or validate signal data.

---

## Security Implications

WiFi CSI enables passive sensing using existing RF signals.

A device does not need to transmit. It only observes changes in the channel.

If CSI can be collected reliably:

* motion inside an environment may be inferred without cameras
* detection becomes harder because no new RF source is introduced
* sensing capability can be embedded in otherwise legitimate devices

At this stage, none of that is demonstrated.

Everything depends on one condition:

→ reliable, repeatable CSI data collection

That has not yet been validated.

---

## GitHub

Repository: wifi-csi-research
Branch: main

Relevant commits:

* `6d6f7bf` — initialize repository
* `0dbdb88` — integrate firmware and enable CSI

---

## Future Work

Next step: establish the first data path.

* register a CSI callback in firmware
* output raw CSI data over serial
* flash to ESP32-C6
* capture serial output
* introduce controlled motion and compare signal changes

The loop closes when:

* CSI frames are observed in serial output
* those frames change in response to controlled physical movement
