+++
date = '2026-05-01T00:00:00-04:00'
draft = false
title = 'Building the First Agent-Environment Loop'
description = 'Building the first working GameMaster AI agent-environment loop, writing tests before the code was right, and letting pytest do the debugging for me.'
author = 'Jesse Bentley'
tags = ['python', 'reinforcement-learning', 'testing', 'pytest', 'ai']
categories = ['gamemaster-ai']
series = ['gamemaster-ai']
showToc = true
tocopen = false
+++

## The Goal

Get the first working agent-environment loop running in the GameMaster AI project. That means a toy environment the agent can interact with, a reward system that reinforces correct behavior, and tests that validate the whole loop behaves the way it should.

Success looks like: pytest passes, the agent moves, the environment responds correctly, and the reward math checks out.

---

## What I Expected

The `NumberLineEnv` was always going to be simple by design. A position on a number line, two actions (left and right), a reward for reaching the goal, a penalty for each wasted step. The `RandomAgent` just picks from the available actions randomly.

Nothing here is supposed to be clever. The point is establishing a working skeleton and an agent talks to environment, environment returns observation and reward, loop runs. Once that's solid, everything more complex gets built on top of it.

I expected the logic to be straightforward and the tests to pass on the first run.

---

## What I Ran Into

They didn't.

Pytest caught two separate bugs immediately and the failure output was specific enough to tell me exactly where to look.

![pytest failure output showing TypeError on observation and RandomAgent attribute error](/images/gamemaster-ai/2026-04-30/pytest-failure-number-line.png)
*3 failed, 3 passed. The failure messages pointed directly at both bugs.*

**Bug one: returning a method instead of calling it.**

Inside `NumberLineEnv`, the observation return looked like this:

```python
observation = self._get_observation
```

That passes back a bound method object, not the actual dictionary the observation is supposed to be. The error — `TypeError: 'method' object is not subscriptable` — shows up the moment anything tries to index into it. The fix is one character:

```python
observation = self._get_observation()
```

It's a small mistake that produces a confusing result if you're reading the code by eye. The test didn't give me a confusing result and it told me the type was wrong.

**Bug two: RandomAgent not storing its action space.**

The `RandomAgent` was initialized without saving the actions it received:

```python
def __init__(self, actions):
    pass
```

Any method that later tried to reference `self.actions` failed because the attribute never existed. The fix:

```python
def __init__(self, actions):
    self.actions = actions
```

Two bugs, both caught immediately by the test suite, both fixed in under five minutes once the output pointed at the right lines.

---

## What the Outcome Was

![pytest output showing 6 passed in 0.03s](/images/gamemaster-ai/2026-04-30/pytest-pass-number-line.png)
*6 passed. The full agent-environment loop validated.*

All tests passing after both fixes. The full loop works: the agent selects a valid action, the environment updates position, the reward system returns the correct value, and the observation comes back as a proper dictionary.

The framework now supports the complete agent → environment → reward cycle. That's the foundation everything else in this project gets built on.

The commit that closed this out: `fix: correct observation call and random agent state`

---

## GitHub

[GameMaster AI](https://github.com/JDBentley/gamemaster-ai)

---

## What's Next

The episode loop works but it's written inline right now. Next step is refactoring it into a reusable `Trainer` class so environments and agents can plug in without rewriting the loop each time. That's the scaffolding the rest of the framework needs before anything more complex is worth building.