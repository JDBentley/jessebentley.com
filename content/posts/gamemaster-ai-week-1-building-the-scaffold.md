+++
date = '2026-04-24T08:00:00-04:00'
draft = false
title = 'GameMaster AI: Project Scaffold and Core Types'
description = 'Week 1 of building GameMaster AI — a from-scratch reinforcement learning system. No libraries, no shortcuts. Starting with the project scaffold and core type system.'
author = 'Jesse Bentley'
tags = ['python', 'reinforcement learning', 'gamemaster-ai', 'project setup', 'pyproject.toml']
categories = ['gamemaster-ai']
series = ['gamemaster-ai']
showToc = true
tocopen = false
+++

## The Goal

Get a real, installable Python project off the ground from a completely empty folder. No training yet, no game connections, no AI. Just a clean scaffold that everything else will eventually sit on top of the build system, linter, type checker, test runner, and the core type definitions that every other file in this project will import from.

---

## What I Expected

Honestly I figured this would be the easy week. Create some folders, write a `pyproject.toml`, define a few data classes, run pytest, call it done. The kind of setup session you knock out in an afternoon and barely remember later.

That was mostly true. Mostly.

---

## What I Ran Into

Two things slowed me down.

**First: the `src/` layout.**

The project puts source code inside `src/gamemaster/` instead of just dropping a `gamemaster/` folder at the root. I'd seen this pattern before but hadn't thought hard about why it exists.

Without the `src/` layer, Python can accidentally import your local source folder instead of the properly installed package. That hides real bugs, code that appears to work on your machine breaks the moment someone else installs it. The `src/` layout forces Python to use the installed version every time, so you catch those problems early instead of in production.

Small detail. Matters a lot later.

**Second: a numpy bug that I already knew how to fix.**

I was writing tests for `types.py` and got this error:

```
TypeError: Cannot interpret '84' as a data type
```

The test was creating a fake screenshot as a numpy array — 84 pixels wide, 84 tall, 3 color channels. The correct call looks like this:

```python
np.zeros((84, 84, 3), dtype=np.uint8)
```

The shape has to be a tuple inside the call. What I'd actually typed in one of the tests was:

```python
np.zeros(84, 84, 3, dtype=np.uint8)
```

No tuple. Numpy reads that as three separate arguments, which makes no sense to it which is why we get a confusing error message that doesn't point at the real problem at all.

Here's the annoying part: I knew this. But I kept scanning the code and my brain kept reading the correct version even though I'd typed the wrong one.

What broke the loop was running:

```
pytest --tb=long
```

That gives you the full traceback with the exact line number. I stopped scanning and read just that one line in isolation. Spotted it immediately. Two missing parentheses. Five minutes of staring at my own code for a two-character fix.

---

## What the Outcome Was

11 tests passing. First commit pushed to GitHub. The scaffold is done and working:

```
gamemaster-ai/
├── pyproject.toml
├── CHANGELOG.md
├── README.md
├── .gitignore
├── src/
│   └── gamemaster/
│       ├── __init__.py
│       └── types.py
└── tests/
    ├── __init__.py
    └── test_types.py
```

`types.py` is the vocabulary file for the whole system. Everything else in this project will import from it. Here's what it defines:

- `ObservationSpace` — how an agent sees a game (structured data, raw pixels, or both)
- `ActionKind` — what kind of actions it can take (a fixed list of choices, a continuous range, or both)
- `ConnectorKind` — how it connects to the game (API, memory reading, screen capture, telemetry stream)
- `Observation`, `Action`, `Transition`, `Episode` — the data structures that flow through every part of training

Tooling is in place too. Ruff for linting, mypy in strict mode for type checking, pytest with asyncio support for tests. The package installs in editable mode so any change to source is live immediately without reinstalling.

---

## GitHub

The full project scaffold from this post is available in the repository:

[gamemaster-ai](https://github.com/JDBentley/gamemaster-ai)

Specific files referenced in this post:
- `src/gamemaster/types.py` — core type definitions
- `tests/test_types.py` — the 11 passing tests
- `pyproject.toml` — build system and tooling configuration
- `CHANGELOG.md` — session log

---

## What's Next

Week 2 is where the project starts actually talking to the outside world. We're building `connectors/base.py` — the abstract base that all four connector types will inherit from — and then the first real connector: `APIConnector`, which lets the agent communicate with Melvor Idle through its Chrome DevTools Protocol interface.
