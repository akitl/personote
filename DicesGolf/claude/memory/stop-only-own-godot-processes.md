---
name: stop-only-own-godot-processes
description: "When pausing work, stop only Godot processes I launched (by PID/command), never the user's Godot 4.7.2 editor or GUT test runs"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 84640d2f-4728-4b16-904d-02b49dbb5ac8
  modified: 2026-10-17T22:33:43.129Z
---

When pausing a workflow, I killed every headless Godot process and took out two of the user's own GUT test runs (Godot 4.7.2, `addons/gut/gut_cmdln.gd`). The user keeps a Godot 4.7.2 editor open on papergoldrip and may run their own tests, while my tooling only uses Godot 4.5.1 with `tests/run_tests.gd`.

**Why:** the user's rule is to stop only the processes I launched, and killing their test runs throws away their work.

**How to apply:**
- Filter by my exact signature (Godot_v4.5.1 plus `tests/run_tests.gd`, `tools/select_seeds.gd`, `tools/simulate_*` and the other scripts my agents run), or by recorded PIDs.
- Never filter on `--headless` alone.
- Don't commit project.godot, .import or .uid changes that the 4.7.2 editor creates.
- Related: [[dices-golf-scope-decisions]].
