+++
date = '2026-05-02T00:00:00-04:00'
draft = false
title = 'WiFi CSI Research: Building a Foundation Before the First Signal'
description = 'Setting up the WiFi CSI research environment, integrating firmware into version control, and enabling CSI on ESP32-C6 before first signal capture.'
author = 'Jesse Bentley'
tags = ['wifi-csi', 'esp32-c6', 'firmware', 'security-research', 'rf']
categories = ['wifi-csi-research']
series = ['wifi-csi-sensing']
showToc = true
tocopen = false
+++

# WiFi CSI Research: Building a Foundation Before the First Signal

## Background

The goal of this project is to use WiFi Channel State Information (CSI) to detect physical motion and environmental changes passively with no cameras, no dedicated sensors, only distortion introduced into an existing RF signal.

Before that is possible, the system needs a foundation that supports reproducible research:

- firmware under version control  
- a consistent build environment  
- CSI explicitly enabled at the hardware level  

This post covers the first two sessions. No CSI data was collected. The outcome here is structural, the system can now be built and configured without guessing.

---

## What I Expected

The expectations were straightforward:

- a repository that separates raw and processed data  
- firmware inside the repository, version-controlled with everything else  
- CSI exposed as a configurable option in ESP-IDF  
- a successful build after enabling CSI  

None of these are complex individually. Skipping them creates failure modes that are harder to debug later.

---

## What I Ran Into

The first issue showed up immediately: the firmware was not inside the repository.

![Initial WiFi CSI repository structure](/images/wifi-csi/2026-05-02/01-initial-repo-structure.png)

*Initial repository structure before firmware was moved into the project.*

The ESP32-C6 project existed at:

```bash
~/esp/csi_capture
```

That breaks reproducibility. The repository can change without the firmware, and the firmware can change without the repository. There is no single source of truth.

The fix was simple which is move the firmware:

![Firmware moved into repository](/images/wifi-csi/2026-05-02/08-firmware-moved-into-repo.png)

*Firmware relocated into `firmware/esp32-c6/csi_capture/` so source and configuration are version-controlled together.*

---

The second issue was environmental.

Running:

```bash
idf.py menuconfig
```

from the wrong directory produced an error instead of opening configuration.

That failure matters. It confirms the environment is not aligned rather than silently misconfiguring the build.

After correcting the working directory and re-sourcing ESP-IDF:

![Menuconfig opened successfully](/images/wifi-csi/2026-05-02/04-menuconfig-success.png)

*ESP-IDF menuconfig opened successfully after running from the correct project context.*

---

From there, the next step was target configuration.

![ESP32-C6 target set](/images/wifi-csi/2026-05-02/05-esp32c6-target-set.png)

*ESP-IDF target configured for ESP32-C6.*

![Target confirmation in build output](/images/wifi-csi/2026-05-02/06-target-confirmation-build-output.png)

*Build output confirming ESP32-C6 target configuration is applied.*

---

The final critical step was enabling CSI.

![WiFi menuconfig options](/images/wifi-csi/2026-05-02/07-wifi-menuconfig-options.png)

*WiFi configuration menu showing CSI option before enabling.*

![WiFi CSI enabled in menuconfig](/images/wifi-csi/2026-05-02/10-csi-enabled-confirmed.png)

*CSI explicitly enabled in firmware configuration.*

CSI is disabled by default. If left unchanged, the firmware builds and runs but produces no CSI data and a silent failure condition.

---

## What the Outcome Was

The repository was initialized and committed.

![Initial files staged for commit](/images/wifi-csi/2026-05-02/02-git-status-initial-files.png)

*Git status showing baseline files staged before the first commit.*

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

The firmware was integrated and CSI enabled:

```text
0dbdb88 — feat: integrate ESP32-C6 firmware and enable CSI support
```

Build verification:

![Build completed with CSI enabled](/images/wifi-csi/2026-05-02/11-build-with-csi.png)

*Firmware build completed successfully after CSI was enabled.*

Verification confirms:

- repository structure exists and is committed  
- firmware now resides inside version control  
- `.gitignore` excludes build artifacts  
- ESP32-C6 target is applied in build output  
- CSI is explicitly enabled  
- firmware builds successfully  

---

## Analysis

The outcome here is structural, not functional.

CSI being disabled by default introduces a silent failure condition. The system appears operational while producing no signal.

The directory error reinforces the same idea. ESP-IDF depends on correct working directory and environment state. In this case, failure was explicit which is preferable.

Moving firmware into the repository closes a reproducibility gap. The firmware source, configuration, and commit history now move together.

Defining structure before data collection enforces separation between:

- experiment definition  
- captured data  
- analysis  

Once data starts accumulating, that separation becomes difficult to impose.

---

## Limitations

- No CSI data has been collected  
- No callback exists to receive CSI events  
- CSI signal flow is unvalidated  
- ESP32-C6 CSI behavior is untested on this hardware  
- Build environment is not pinned or containerized  

The system builds. It does not yet produce or validate signal data.

---

## Security Implications

WiFi CSI enables passive sensing using existing RF signals.

A device does not need to transmit but it only observes channel changes.

If CSI can be collected reliably:

- motion may be inferred without cameras  
- detection becomes harder (no new RF source)  
- sensing can be embedded into legitimate devices  

At this stage, none of that is demonstrated.

Everything depends on one condition:

→ reliable, repeatable CSI data collection  

That has not yet been validated.

---

## GitHub

Repository: wifi-csi-research  
Branch: main  

Relevant commits:

- `6d6f7bf` — initialize repository  
- `0dbdb88` — integrate firmware and enable CSI  

---

## Future Work

Next step: establish the first data path.

- register a CSI callback in firmware  
- output raw CSI data over serial  
- flash to ESP32-C6  
- capture serial output  
- introduce controlled motion and compare signal changes  

The loop closes when:

- CSI frames are observed in serial output  
- those frames change in response to controlled movement  