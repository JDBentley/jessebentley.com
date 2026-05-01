+++
date = '2026-05-01T00:00:00-04:00'
draft = false
title = 'SentinelFlip — Building the Foundation'
description = 'Day zero on SentinelFlip, a modular physical reconnaissance platform. Before writing any code, I spent time thinking about structure — and why that matters for a project that will generate hardware, firmware, data, and experiments all at once.'
author = 'Jesse Bentley'
tags = ['sentinelflip', 'esp32', 'physical-security', 'reconnaissance', 'hardware']
categories = ['sentinelflip']
series = ['sentinelflip']
showToc = true
tocopen = false
+++

## The Goal

Start building SentinelFlip which is a modular physical reconnaissance platform built around ESP32 hardware, wireless signal analysis, and structured data collection.

The use cases I have in mind are practical ones: physical security assessments, checking unknown environments like Airbnb rentals, and long-term monitoring for unexpected devices or changes in a space. Not a CTF tool. Not a demo. Something that could actually be useful in the field.

This is day zero. No firmware yet, no hardware, no captured data. Just laying the foundation correctly before any of that starts.

---

## What I Expected

I expected this step to take twenty minutes. Create a repo, add a README, push it, move on.

---

## What I Ran Into

Once I started thinking through what this project actually involves, the "quick repo" idea fell apart.

SentinelFlip isn't just code. It's going to include hardware builds, firmware running on embedded devices, captured signal data, analysis scripts, and experiments that will break frequently. If all of that lands in one flat directory without any structure, it becomes hard to debug, hard to explain, and hard to reuse when it actually matters.

So instead of rushing forward, I treated it like a system design problem. What does this project generate? Where does each thing live? What's going to be messy by nature, and what needs to stay clean?

---

## What the Outcome Was

A repository structure that separates concerns from the start:
hardware/     → physical builds, wiring diagrams, enclosure design
firmware/     → embedded code running on ESP32 devices
tools/        → analysis scripts and utilities
experiments/  → raw testing and iteration
data/         → captured logs and datasets
docs/         → long-form documentation

![Local repository structure showing top-level directories](/images/sentinelflip/2026-05-01/01-repo-structure-terminal.png)
*The local structure before anything gets pushed.*

![GitHub repository on day zero with initial commit](/images/sentinelflip/2026-05-01/02-github-repo-initial.png)
*Day zero baseline. One commit, clean history from the start.*

Three decisions drove this layout.

**Separation between experimentation and tooling.** Experiments are supposed to be messy and that's the point. But I don't want unstable test code mixed in with tools I might rely on during an actual engagement. Keeping `experiments/` and `tools/` separate means I can iterate aggressively in one place without touching the other.

**Data needs a home from day one.** This project will generate a lot of captured data. If that data doesn't have a predictable place to live, it becomes noise. Structured storage makes it usable for analysis, debugging, and demonstrating results.

**Real-world usability.** In a real assessment I don't want to dig through random scripts trying to remember what does what. I want predictable structure, clean tooling, and workflows I've already tested. Building that in from the beginning costs almost nothing. Retrofitting it later costs a lot.

---

## GitHub

[SentinelFlip](https://github.com/JDBentley/sentinelflip)

---

## What's Next

The structure is in place. Now the project needs to become real.

Next steps are flashing an ESP32-S3, building initial firmware for WiFi and BLE scanning, capturing the first actual signal data, and starting to sketch out a logging pipeline. That's where this moves from organized folders to actual capability.