+++
date = '2026-05-08T00:00:00-04:00'
draft = false
title = 'SentinelFlip: Day Zero'
description = 'Starting a modular physical reconnaissance platform the right way — structure before code, decisions before tools.'
author = 'Jesse Bentley'
tags = ['sentinelflip', 'esp32', 'hardware', 'reconnaissance', 'physical-security']
categories = ['sentinelflip']
series = ['sentinelflip']
showToc = true
tocopen = false
+++

## The Goal

Start building SentinelFlip which is a modular physical reconnaissance platform for real-world use.

Not a proof-of-concept that works once in a lab. Something that holds up in:

- Physical security assessments  
- Travel security checks in Airbnbs, unfamiliar environments, unknown infrastructure  
- Long-term environmental monitoring such as detecting unexpected devices or changes over time  

The system will combine embedded hardware (ESP32), wireless signal collection (WiFi and BLE), structured data capture, and tooling that can be reused without rebuilding from scratch.

---

## What I Expected

Day Zero should have taken twenty minutes.

Create a repo, add a README, push, move on.

The assumption was simple: real work starts when hardware arrives.

---

## What I Ran Into

That assumption broke as soon as I thought through what this project actually produces.

This isn’t just code. It generates:

- Hardware builds and wiring documentation  
- Firmware running on ESP32 devices  
- Captured signal data from real environments  
- Analysis tooling built on top of that data  
- Experiments that will fail repeatedly  

If all of that lives in a flat structure, the project becomes:

- Hard to debug  
- Hard to maintain  
- Hard to use in real scenarios  

The failure mode isn’t theoretical and it’s operational. You lose track of what’s stable, what’s experimental, and what data can be trusted.

So instead of pushing forward, I stopped and treated this as a system design problem.

---

## TELOS: How I Approached This Problem

The initial mistake was treating repository structure as something cosmetic and as something to fix later.

That doesn’t hold for a system like this.

SentinelFlip operates across three distinct contexts:

- Controlled lab testing  
- Real-world field use  
- Long-term passive monitoring  

Each has different failure tolerances.

- Firmware that crashes is acceptable in a lab  
- The same behavior is a failure in the field  
- Long-term monitoring requires consistency over time  

Mixing those contexts breaks reliability.

So the problem became separation of concerns:

- What is expected to break? → experiments  
- What must remain stable? → tools and workflows  
- What must persist and be trusted later? → data and documentation  

Those categories are incompatible if they share the same space.

The friction point was instinct. I kept wanting to start building immediately.

That works for small scripts. It fails for systems that accumulate:

- Hardware revisions  
- Firmware iterations  
- Captured datasets  
- Analysis tooling  

What changed was framing.

This is not a repository. It’s infrastructure.

Infrastructure decisions made early are cheap.  
The same decisions made after months of accumulated data are expensive and disruptive.

Security relevance is direct.

Field tooling doesn’t usually fail because the core capability is wrong. It fails because:

- Data isn’t structured  
- Experimental code contaminates stable workflows  
- Results aren’t reproducible  

If output can’t be reproduced, it can’t be trusted.

This structure is meant to prevent that from happening.

---

## What the Outcome Was

A repository structure that separates concerns from the start:

- `hardware/` — physical builds, wiring diagrams, enclosure design  
- `firmware/` — embedded code running on ESP32 devices  
- `tools/` — analysis scripts and utilities  
- `experiments/` — raw testing and iteration  
- `data/` — captured logs and datasets  
- `docs/` — long-form documentation  

![Local repository structure showing top-level directories](/images/sentinelflip/2026-05-01/01-repo-structure-terminal.png)  
*Day Zero directory structure. Separation of concerns before a single line of firmware exists.*

![GitHub repository showing initial commit](/images/sentinelflip/2026-05-01/02-github-repo-initial.png)  
*Initial commit live on GitHub. One branch, one commit, clean baseline.*

What confirms this worked:

- The repo exists with defined separation before any implementation  
- The structure is visible locally and in GitHub  
- There is a single clean baseline commit with no mixed concerns  

What this enables:

- Experiments remain isolated from stable tooling  
- Data remains structured and retrievable  
- Workflows become predictable instead of ad hoc  

This is still early-stage.

The structure exists. The capability does not.

That’s intentional.

---

## GitHub

[SentinelFlip](https://github.com/JDBentley/sentinelflip)

---

## What's Next

Structure is in place. Now it needs to produce data.

- Flash the ESP32-S3  
- Build initial firmware for WiFi and BLE scanning  
- Capture first real signal data  
- Begin building the logging pipeline  

This is where the project transitions from organized to operational.