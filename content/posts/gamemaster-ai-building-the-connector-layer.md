+++
date = '2026-05-01T08:00:00-04:00'
draft = false
title = 'GameMaster AI: Building the Connector Layer'
description = 'Week 2 of GameMaster AI — building the abstract connector base class and the first real connector that talks to Melvor Idle via Chrome DevTools Protocol.'
author = 'Jesse Bentley'
tags = ['python', 'reinforcement learning', 'gamemaster-ai', 'cdp', 'websockets', 'testing']
categories = ['gamemaster-ai']
series = ['gamemaster-ai']
showToc = true
tocopen = false
+++

## The Goal

Build the connector layer which is the code that sits between the AI agent and whatever game it's playing. This week that means two things: an abstract base class that defines what every connector must be able to do, and the first real connector that talks to Melvor Idle via Chrome DevTools Protocol.

---

## What I Expected

The base class felt straightforward going in. Four methods every connector needs is the connect, observe, act, and disconnect all wrapped in an abstract class so Python enforces the contract. I expected that to be quick.

The CDP connector was the part I wasn't sure about. I'd never talked to a game through a debug port before and I wasn't sure how the websocket communication would feel to write.

---

## What I Ran Into

Honestly? Not much broke this week. I went slow on purpose and especially so through the CDP connector. Before typing each section I stopped and mentally walked through what the code was actually doing. What is this websocket sending? What comes back? Why does the command ID need to increment?

That habit paid off. Nothing failed that shouldn't have failed.

The one thing worth explaining is why `_send` loops waiting for a matching ID instead of just reading the next message back. CDP is asynchronous and the game can send unsolicited events between your request and its response. Things like "the player just leveled up" or "a notification fired." If you just read the next message you might grab one of those instead of your actual response. The loop filters by command ID so you always get the reply you asked for.

The other thing that shaped this week's design was a hardware constraint I hadn't thought about up front: Melvor Idle is tied to a Steam account. One instance at a time. That kills the standard trick of running 8 or 16 copies of the game simultaneously to collect training experience faster. The connector is designed as single-instance by default because of it.

---

## What the Outcome Was

27 tests passing, up from 19 last week. The connector layer is in place and working:

- `base.py` defines the contract every connector must satisfy
- `api.py` implements CDP communication: connecting to the game, sending JavaScript, reading results back
- Both are fully tested without needing a real game running and the tests mock the websocket and HTTP calls

The CDP connector works like this in practice:

1. Hit `http://localhost:9222/json` to discover what's running
2. Grab the websocket URL for the page target
3. Connect and send JSON commands with incrementing IDs
4. Execute JavaScript directly inside the game's runtime via `Runtime.evaluate`
5. Read back whatever the game exposes

Melvor Idle runs in Electron, which is basically a browser in a desktop window. That browser exposes a CDP debug port the same way Chrome does when you open DevTools. Instead of scraping pixels off a screen we just ask the game directly what's happening.

The README also got its first real update this week — architecture overview, all four connector types documented, full game phase list, and setup instructions.

---

## GitHub

From-scratch reinforcement learning system that trains AI agents to play video games — no libraries, no wrappers.

[gamemaster-ai](https://github.com/JDBentley/gamemaster-ai)

Specific files referenced in this post:

- `src/gamemaster/connectors/__init__.py` — package marker for the connectors module
- `src/gamemaster/connectors/base.py` — abstract base class all connectors inherit from
- `src/gamemaster/connectors/api.py` — CDP connector for browser-based games
- `tests/connectors/test_base.py` — stub-based tests for the base class
- `tests/connectors/test_api.py` — mock-based tests for the API connector
- `README.md` — architecture overview, game phase list, setup instructions

---

## What's Next

Week 3 is where the agent actually touches Melvor Idle for the first time. We're building the game-specific subclass that overrides `_get_structured_obs`, `_execute_action`, and `_compute_reward` with real Melvor Idle logic by reading gold, skills, and action queues directly from the game via JavaScript. First real observation. First reward signal.

The plumbing is done. Time to plug something into it.