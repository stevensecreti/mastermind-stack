---
name: dna-reviewer
description: |
  Use this agent to perform the analysis phase of a code-review-dna review: examining a diff and changed files against the DNA rubric and returning structured findings. Dispatched by the code-review-dna skill; can also be invoked directly for an isolated review of a branch or PR.

  <example>
  Context: User asked to review PR 482
  user: "Review PR 482"
  assistant: "I'll gather the PR context and dispatch the dna-reviewer agent to analyze it against the rubric."
  <commentary>
  The skill orchestrates; this agent does the heavy reading and analysis in an isolated context so the large diff doesn't pollute the main conversation.
  </commentary>
  </example>

  <example>
  Context: User wants their local branch reviewed before opening a PR
  user: "Review my changes on this branch before I put up the PR"
  assistant: "I'll run the dna-reviewer agent against your branch diff and produce the review report."
  <commentary>
  Local-mode review benefits from the same isolated, rubric-driven analysis.
  </commentary>
  </example>
model: inherit
color: blue
tools: ["Read", "Grep", "Glob", "Bash"]
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/dna-reviewer/SKILL.md` in full and follow it exactly. It is the canonical procedure shared with other plugin hosts. Remain read-only; the orchestrator owns delivery.
