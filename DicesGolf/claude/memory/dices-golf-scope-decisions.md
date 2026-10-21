---
name: dices-golf-scope-decisions
description: "User decisions (2026-10-13) for the Dices Golf full development chantier — scope, courses, visitors, git"
metadata:
  node_type: memory
  type: project
  originSessionId: 84640d2f-4728-4b16-904d-02b49dbb5ac8
  modified: 2026-10-20T09:12:22.791Z
---

On 2026-10-13 the user asked me to develop Dices Golf (Godot project papergoldrip, rules in doc/claude.md + GDD v0.2) and answered:
- Scope: everything up to the infernal tier (all 87 skills, 18 campaign courses); endless rules stay open in the GDD.
- Campaign courses: procedural generator + solver that simulates attempts to select feasible seeds, stored with generator version, flagged provisional.
- Visitors: GDD profile v0.2 — deterministic economic cycles, visible golfers purely decorative.
- Git: work on branch `feat/v02-core`, commit per milestone (Conventional Commits, English, scopes godot/data/docs), no push.
- User allows breaking/replacing existing prototype code; keep very simple pixel-art style.

**Why:** these resolve the GDD "à approuver"/open choices for this chantier.
**How to apply:** don't re-ask these; record them in GDD §20 as user decisions.

Follow-up decisions (2026-10-14):
- hell_03 seed compromise accepted by user.
- Endless mode must be implemented: chain of generated holes of rising difficulty, each with its own shot/reroll budget; a failure ends the run, next run restarts at depth 1; record = max depth; ordinary manual gold capped at infernal scale, no first-clear bonus, no production.
- Endless skills: repeatable without cap, price rising per purchase, game stats only, active only in endless (campaign unchanged). Also serves as late gold sink.
- Red tier too easy → add WATER surface (ball stopping in water returns to shot start, shot lost), from red onward.
- Simulated player (solver) should be nerfed slightly to play like a mediocre human.
- Old saves can be trashed (no migration needed).

Roadmap decisions (2026-10-16, plan in ~/.claude/plans/peppy-seeking-stream.md):
- Dice redesign: "arrival zone per die" — DC/reach unchanged; ordinary success lands in a die-sized zone (D6 exact … D30 widest), crit lands exact, zone shown before shooting.
- Mini-golf: ~10 handcrafted dice-free puzzle levels (gates, pushers, bumpers, ice), optional activity.
- Club-house becomes functional + a few freely placed buildings (overrides GDD §01 exclusion, keep simple).
- Pace: chain all 7 plan steps without stopping, one commit per step, user tests at the end.
- Agents must use Read/Edit/Write rather than sed/cat (user's permission prompts were disruptive).

Playtest feedback (2026-10-17):
- Dice zones shifted one size up: D6 exact, D10 = old D20 (3×3), D20 = old D30 (11 cells), D30 ≈21 cells; ordinary success uniform over the zone (target not favoured), crit always exact; removed cells renormalised over the rest.
- Building prices OK, but clubhouse level 2 / buildings must be buyable from the blue tier (expensive, optional).

Decisions (2026-10-18, plan peppy-seeking-stream.md rewritten):
- Toolchain switches to Godot 4.7.2 (user's editor/export version) for tests, sims, captures.
- Newly unlocked course types go straight to the shop as a free offer (no more reserve); building click opens its own panel (mini-golf "Jouer"); quit button in options; dice wheel must not zoom + keyboard dice shortcuts; bonus tiles placed on near-optimal routes and picked up within 1 cell.
- Mini-golf: 10 more levels (20 total), same rewards (200 G first clear, 0.25 %/star, up to 15 %).
- Steam videos: trailer 60–90 s + gameplay 2–3 min, 1080p60 H.264, using the game's own music — user confirmed the music licence allows promotional use (still no credits file/source name).

Steam integration (2026-10-20), Steam app id 4165680:
- GodotSteam GDExtension plugin in addons/godotsteam (not the module folder ../GodotSteam-godot4, which needs an engine rebuild); service must no-op without Steam/plugin (tests, web).
- ~30 achievements, icons derived from skill-tree icons (pixel-sharp upscale + grey locked variant).
- Store texts EN + FR; store visuals composed in Godot from real game elements with a pixel-art "Dices Golf" logo.
