---
name: refactor
description: |
  Use this agent to improve code's structure and maintainability WITHOUT changing its behavior. It works in two complementary modes: subtractive (cut complexity, kill duplication, strip premature abstraction, minimize a changeset's footprint) and constructive (apply SOLID, introduce the right design pattern, extract methods/classes, replace conditionals with polymorphism), scoped either to a diff (uncommitted changes / recent commits) or to a codebase area. Reach for it before committing/PRing to tighten a changeset, when a module is accreting smells (duplication, long methods, large classes, SRP violations), or when a piece of code is either over-engineered or under-structured.

  Examples:
  <example>
  Context: The user just finished a feature and wants the diff tightened before committing.
  user: "I implemented the auth flow. Review my changes for unnecessary complexity before I commit."
  assistant: "I'll use the refactor agent in subtractive mode on your changeset to strip excess and catch any duplication of existing helpers."
  <commentary>Finished changeset + 'unnecessary complexity' → refactor agent, subtractive lens, diff-scoped.</commentary>
  </example>
  <example>
  Context: A service class has grown to handle several responsibilities.
  user: "UserService now does user management AND reporting AND notifications. Can we clean this up?"
  assistant: "I'll use the refactor agent in constructive mode to split UserService along its responsibilities (SRP) and introduce the right seams."
  <commentary>Large class / SRP violation across the codebase → refactor agent, constructive lens.</commentary>
  </example>
  <example>
  Context: The user over-abstracted a small utility.
  user: "This factory + strategy setup for two cases feels like overkill."
  assistant: "I'll use the refactor agent in subtractive mode to collapse the premature abstraction down to what the two cases actually need."
  <commentary>Over-engineering / premature abstraction → refactor agent, subtractive lens.</commentary>
  </example>
model: opus
color: purple
---

Read `${CLAUDE_PLUGIN_ROOT}/skills/refactor/SKILL.md` in full and follow it exactly. It is the canonical procedure shared with other plugin hosts.
