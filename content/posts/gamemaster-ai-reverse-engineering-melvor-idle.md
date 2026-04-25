+++
date = '2026-05-08T08:00:00-04:00'
draft = false
title = 'GameMaster AI: Reverse Engineering Melvor Idle'
description = 'Week 3 of GameMaster AI — attempting to unlock parallel training instances by reverse engineering the Melvor Idle NW.js app. Three layers deep and still not there.'
author = 'Jesse Bentley'
tags = ['python', 'reinforcement learning', 'gamemaster-ai', 'reverse engineering', 'nwjs', 'cdp', 'steam', 'security']
categories = ['gamemaster-ai']
series = ['gamemaster-ai']
showToc = true
tocopen = false
+++

## The Goal

Unlock parallel instances of Melvor Idle so the AI can collect training experience from multiple game windows simultaneously. Standard RL training runs 8 to 16 environments at once with more data, faster learning. The game only allows one window. The goal was to fix that.

We didn't.

---

## What I Expected

Melvor Idle runs on NW.js which is basically a browser wrapped in a desktop app. I expected to find a single instance lock in the JavaScript source, patch it out in ten minutes, and spend the rest of the week setting up the CDP connection.

The CDP part worked. The parallel instances part did not. Here's exactly what happened.

---

## What I Ran Into

**Layer 1 — Finding the app structure**

Melvor Idle doesn't use Electron like I originally assumed. It uses NW.js, which means the source files sit unpackaged in a folder called `package.nw` inside the game directory. No `.asar` to extract. Just open files.

`package.json` configures the whole app. Two things jumped out immediately:

```json
"chromium-args": "... --disable-devtools",
"id": "melvor_idle_steam"
```

`--disable-devtools` was blocking CDP access entirely. The window `id` was likely enforcing single instance behavior. Both were one-line fixes.

Removed `--disable-devtools`, removed the `id` field, launched the game and CDP came up immediately. That part worked perfectly.

**Layer 2 — Steam SDK blocking at the SDK level**

Removing the window ID wasn't enough for parallel instances. Steam saw the game already running and refused to launch a second copy. Launching the executable directly with a different debug port silently failed and the process started and died before a window opened.

The culprit was `greenworks.init()` in `steamInit.js`. Greenworks is the Steam SDK bridge for NW.js. When a second instance calls `initAPI()` the Steam SDK detects an existing session and kills the process. This happens in compiled C++ code inside `greenworks-win64.node` and before any JavaScript runs.

I patched `steamInit.js` to check for a lock file first:

```javascript
var FLAG_FILE = 'F:\\MelvorAI\\instance.lock';

function testSteamAPI() {
    if (fs.existsSync(FLAG_FILE)) {
        console.log('Secondary instance — skipping Steam init.');
        return false;
    }
    if (greenworks.init()) {
        fs.writeFileSync(FLAG_FILE, process.pid.toString());
        return true;
    }
    return false;
}
```

The lock file was never created. The JavaScript patch never ran. The binary killed the process first.

**Layer 3 — The compiled binary wall**

`greenworks-win64.node` is a compiled Node.js native addon. It calls the Steam SDK at the C++ level. There's no JavaScript to patch. To get past this layer the options are:

- Patch the compiled binary directly with a hex editor
- Hook `initAPI()` at runtime with Frida
- Strip the Steam SDK entirely and break achievements and cloud saves

None of those are a quick fix. All of them are a week of work on their own.

**What did work — CDP and first game state**

While hunting the parallel instance problem we ended up going further than planned on the CDP side. The game runs its actual logic inside an iframe pointed at `https://steam.melvoridle.com/index_game.php`. That iframe has its own JavaScript execution context — context ID 4 in our session.

Once we found that context we could execute JavaScript directly inside the game. The full `game` object is exposed on the window with skills, bank, combat, currencies, everything. The first real observation from a live character:

```json
{
  "gp": 0,
  "woodcutting_level": 1,
  "woodcutting_xp": 0,
  "bank_items": 0,
  "current_action": null,
  "is_active": false
}
```

That's what the AI sees when it opens its eyes for the first time. Zero gold, level 1, blank slate.

---

## What the Outcome Was

CDP access unlocked. First real game state read and done. Parallel instances however not achieved.

The goal was parallel instances. We didn't get there. Three layers deep into the app and hit a compiled binary we can't patch from JavaScript. That's the honest result.

The CDP work was a side win that wasn't even on the original plan for this week. It'll matter more next week when we build the Melvor-specific connector.

The parallel instance problem isn't dead but it's deferred. Frida is the right tool for layer 3 and it deserves its own dedicated week later in the project. For now Phase 1 trains on a single instance with maxed game speed and a high replay ratio.

---

## GitHub

From-scratch reinforcement learning system that trains AI agents to play video games — no libraries, no wrappers.

[gamemaster-ai](https://github.com/JDBentley/gamemaster-ai)

No new files were committed to the repository this week. The work was entirely on the Melvor Idle app itself — `package.json` and `steamInit.js` patches live in the game directory, not the project repo. `scratch.py` was used for exploration and discarded.

---

## What's Next

Week 4 is the Melvor Idle specific connector. We have CDP working and we know exactly where the game state lives — `game.gp`, `game.woodcutting`, `game.bank`, all of it accessible from Python via a websocket. Next week we build the subclass that turns that raw game data into a structured observation the agent can actually train on. First reward signal. First action sent to the game.

The parallel instance problem comes back later but with Frida.