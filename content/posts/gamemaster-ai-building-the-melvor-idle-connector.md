+++
date = '2026-05-15T08:00:00-04:00'
draft = false
title = 'GameMaster AI: Building the Melvor Idle Connector'
description = 'Week 4 of GameMaster AI — building the full Melvor Idle connector with dynamic skill and action discovery, observation vectors, and action masking.'
author = 'Jesse Bentley'
tags = ['python', 'reinforcement learning', 'gamemaster-ai', 'melvor idle', 'cdp', 'rainbow dqn', 'hierarchical rl']
categories = ['gamemaster-ai']
series = ['gamemaster-ai']
showToc = true
tocopen = false
+++

## The Goal

Build the Melvor Idle specific connector which is the Python class that reads live game state, builds a normalized observation vector the agent can train on, discovers every possible action in the game automatically, and masks out actions the agent isn't allowed to take yet based on its current skill levels.

---

## What I Expected

I expected this to be a straightforward translation week. We already knew where the game data lived from Week 3. I figured it was just a matter of reading it into Python and packaging it up cleanly.

It was more than that. A lot more.

---

## What I Ran Into

**Installing the expansions first**

Before writing a single line of the connector I installed both expansions - Throne of the Herald and Into the Abyss. This turned out to be the right call.

Without expansions: 24 skills, 895 recipe actions.
With both expansions: 28 skills, 1,520 recipe actions.

If we'd built the connector against 895 actions and added expansions mid-training, the observation vector size and action space size would have changed. That means the neural network architecture changes. That means throwing away all training progress and starting over.

Always install everything before you define your action space.

**The frame ID changes every restart**

In Week 3 we hardcoded the game iframe's frame ID to find the right JavaScript execution context. That ID changes every time the game restarts. Hardcoding it would have broken the connector every single session.

The fix: discover the context by origin instead of by ID. The game iframe always loads from `https://steam.melvoridle.com` and that never changes. We listen for context creation events and grab the first one matching that origin. Survives restarts, survives reconnects, survives everything.

**1,521 actions is a lot**

The full action space after expansion discovery:

- Magic: 26 spells
- Woodcutting: 34 trees
- Fishing: 59 spots
- Firemaking: 33 logs
- Cooking: 86 recipes
- Mining: 40 ores
- Smithing: 279 recipes
- Thieving: 50 targets
- Farming: 71 crops
- Fletching: 132 recipes
- Crafting: 187 recipes
- Runecrafting: 229 recipes
- Herblore: 72 potions
- Agility: 115 obstacles
- Summoning: 53 familiars
- Astrology: 29 constellations
- Archaeology: 18 sites
- Harvesting: 7 plants
- Plus 1 stop action

**1,521 total.** The agent starts with most of these locked. Action masking handles it and is a binary vector where `True` means the action is currently valid based on the agent's skill levels. As the agent levels up, more actions unmask automatically. The neural network never changes size.

**The websockets library changed**

Between when the original `APIConnector` was written and now, websockets updated to version 16.0 and moved types around. `WebSocketClientProtocol` became `ClientConnection`. The import path changed from `websockets.client` to `websockets.asyncio.client`. The test mocks were patching the wrong path.

Mypy caught it. Fixed in about ten minutes once we knew what changed. Good reminder to pin your dependencies or check changelogs before assuming old code still works.

**A lot to absorb**

Honest moment: this week had a steep learning curve. CDP, execution contexts, observation normalization, action masking, reward signals, the difference between `evaluate` in the base class and `_game_evaluate` in the Melvor connector and it's a lot of moving pieces to hold in your head at once.

I went slow. I read each block before typing it. I asked why before asking what. That's the right pace for this kind of project even when it feels slow.

---

## What the Outcome Was

42 tests passing. The connector is complete and working:

- 28 skills discovered dynamically at connect time
- 1,521 actions in the full action registry
- Observation vector: 60 floats — 28 skills × 2 (level + XP normalized) plus 4 globals (GP, slayer coins, bank items, combat flag)
- Action mask built from live skill levels every tick
- Reward signal: total XP gained since last observation
- Full game state read in a single JavaScript call — no timing issues

We also locked in some major architecture decisions this week:

**Algorithm roadmap:**
- Phase 1: Rainbow DQN — most sample efficient discrete action algorithm, critical for single instance training
- Phase 2: PPO — better for longer episodes in simulation games
- Phase 3: Rainbow DQN or PPO depending on game type
- Phase 4: SAC — continuous control for racing games
- Research tier: custom multi-agent for StarCraft and Dota 2

**Algorithm-agnostic training interface:** The training loop will use an abstract `BaseAgent` class. Swapping Rainbow for SAC means changing a config value, not rewriting the system. Designed for the game we'll train in year three, not just the game we're training today.

**Hierarchical RL planned for Week 9:** High level manager sets goals (max woodcutting, unlock smithing). Low level worker executes the steps. Natural fit for Melvor Idle's skill tree — the game's own prerequisite structure becomes the curriculum.

---

## GitHub

From-scratch reinforcement learning system that trains AI agents to play video games — no libraries, no wrappers.

[gamemaster-ai](https://github.com/JDBentley/gamemaster-ai)

Specific files referenced in this post:

- `src/gamemaster/connectors/melvor.py` — full Melvor Idle connector with skill discovery, action registry, observation vector, action masking, and reward signal
- `src/gamemaster/types.py` — added `metadata` field to `Observation` dataclass
- `tests/connectors/test_melvor.py` — 15 new tests covering state, masking, and observation vector
- `tests/connectors/test_api.py` — fixed websocket mock patch targets for websockets v16

---

## What's Next

Week 5 is item protection and shop integration. The agent needs to know which bank items are crafting ingredients and must never sell them. It also needs to understand the shop — what's available, what it can afford, what helps progression. Both affect the observation space and action space. After that, Week 6 is the training loop — Rainbow DQN, replay buffer, and the first actual training run on a live Melvor Idle character.