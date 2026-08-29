---
name: axis-architecture
description: |
  Reviews a diff against the Architecture & API Semantics axis only (RUBRIC.md Tier 1): single source of truth, single responsibility, abstraction boundaries, earned abstractions, public-interface design, YAGNI, duplication/replacement, cross-layer sync, service boundaries. Dispatched in parallel by the deep-review skill.

  <example>
  Context: deep-review fanned out the axes for a PR
  user: "[deep-review orchestrator dispatch] architecture axis for PR 482"
  assistant: "Reviewing only the architecture/API tier and returning findings."
  <commentary>
  One axis per agent; this one owns the highest-cost-to-change-later concerns.
  </commentary>
  </example>
model: inherit
color: blue
tools: ["Read", "Grep", "Glob", "Bash"]
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/axis-architecture/SKILL.md` in full and follow it exactly. It is the canonical procedure shared with other plugin hosts. Remain read-only.
