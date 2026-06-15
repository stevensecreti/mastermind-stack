---
name: crystallize
description: Turn the task you just did into a reusable, project-agnostic skill. Takes a concrete piece of work completed in this session, extracts its abstract recipe (intent, procedure, decision rules, done-condition), strips the incidental specifics (repo paths, file names, values, the particular framework/library/tool), and drafts a clean SKILL.md. Use when you've just done something by hand that you'll plausibly do again and want to "build the lever". Triggers: "make this a skill", "crystallize this", "turn what we just did into a skill", "encode this workflow".
argument-hint: [optional: what to crystallize and/or the new skill name]
---

# Crystallize

Turn the work we **just did** into a reusable skill. The principle is **Build the Lever**: you did the task by hand, learned the exact recipe, now encode it so it runs the same way every time. The hard, valuable part is **abstraction**: keep the essential procedure, drop the incidental specifics, so the skill works far beyond this one case.

## When to use, and when not to

**Use it** when you've just completed (or repeated) a multi-step task with a recipe you'll plausibly hit again, and the work involves real judgment a skill can guide.

**Don't** crystallize when:
- It's a genuine one-off. The *Laziness Protocol* applies: don't build a lever for something you won't repeat. Stay lazy here.
- It's purely deterministic with no AI judgment. That's a **script**, not a skill (a `cleanup-branch`-style chore belongs in code).
- An existing skill already covers it. Check first; extend the existing one rather than duplicating.

## Step 1: Pin the unit of work

State, in one sentence, exactly what we're crystallizing. If the session did several things, ask which one (or scope to the primary one). You're capturing *one clear job*.

## Step 2: Extract the abstract recipe

Reconstruct what was actually done as a repeatable procedure, the essential moves, not the false starts:

- **Trigger**: what situation calls for this?
- **Inputs**: what it needs to begin.
- **Procedure**: the ordered steps.
- **Decision points**: where judgment was applied, and the *rule* used to decide.
- **Verification / done-condition**: how you knew it was complete and correct.
- **Guardrails**: what not to do; the failure modes actually hit this time.

## Step 3: Separate essence from incidentals (the core move)

This is where the payoff comes from. Classify every concrete detail in what we did:

- **Essential (keep):** the procedure, the decision rules, the verification.
- **Incidental (strip or parameterize):** this repo's paths and file names, specific values, org names, and **the particular framework / library / language / tool** that happened to be involved.

Replace incidentals with inputs, placeholders, or "detect from the project." **Enforce the agnosticism bar:** the skill must not assume a specific library/framework/language/tool. If the work used one, generalize it, or make it a config key / a runtime detection step. A skill born here should be portable by construction.

## Step 4: Generalize and right-size

- **Will it fire again?** Name 1-2 *other* situations this skill would apply to. If you can't, it's a one-off (back to *Laziness Protocol*); stop.
- **Right-size it.** One clear job. If it's sprawling, it's probably several skills or none.
- **Check for overlap** with existing skills/agents. If it's a variant of one, extend that instead of shipping a near-twin.

## Step 5: Draft the SKILL.md

- **Frontmatter:** `name` (kebab-case; verify it collides with neither a built-in command nor an existing skill), `description` (what it does **and** the when-to-use triggers, since that drives invocation).
- **Body:** the recipe from Steps 2-3, written as instructions to a *future agent who wasn't here*: concrete, ordered, decision rules and guardrails inline.
- **Match the house format:** self-describing, in the voice of the surrounding skills.
- **Place it** in the right skill directory: the plugin's `skills/<name>/` if it's a portable stack skill, or the consuming repo's `.claude/skills/<name>/` if it's project-local. Ask if unclear.

## Step 6: Verify

- Re-read it cold: could someone follow this *without having been in this session*? Strip any "as we just saw" references.
- Confirm no incidental coupling leaked through (paths, values, tool/framework names, org).
- Confirm it still earns its place.

## Output

Report: the skill's name + path, a one-line statement of what it crystallizes, and (most important) the **essence/incidental split** you made, so the abstraction can be sanity-checked. Flag anything you parameterized and any scope you were unsure about.
