# mastermind-stack

> The skills, agents, and house-style rules I use to get high-quality work out of Claude Code, pulled out of my own projects and made to run anywhere.

Coding agents are only as good as the engineer driving them. I've spent years building up a way of working: the standards I hold myself to, the patterns I reach for, the voice I review in. The more I leaned on Claude Code, the more I noticed I was re-teaching it the same things in every repo, so I pulled all of that accumulated craft out into one portable place. This is that.

I think the most valuable thing you can hand an agent is taste: a clear bar for what good looks like, and the discipline to subtract before you add. So the tools here lean on that. They reach for deletion before addition, they hold a real definition of done, and they review against standards I actually believe in rather than generic best-practice filler.

Everything is deliberately agnostic. No skill assumes a particular framework, library, language, test runner, or database, since I want the same bar whether I'm in a personal project, a side business, or my day job. When a tool genuinely needs to know something about your repo, like how you run tests, it reads it from a small config instead of hardcoding it.

And the whole thing is self-describing. A router skill maps whatever you're trying to do to the right tool, and every skill explains itself, so you (and the agent) can always find your way around.

## Install

```sh
# add this repo as a plugin marketplace
claude plugin marketplace add stevensecreti/mastermind-stack

# install the plugin
claude plugin install mastermind-stack@mastermind-stack
```

Then, once per repo, run the setup skill so the plugin knows your project's specifics and the house-style rules take effect:

```
/mastermind-setup
```

## What's inside

Each tool does one job. If you're not sure which one you want, the `mastermind` router will point you at it.

### Skills (14)

**UI design & analysis**


| Skill          | Does                                                                                                                                                                                   |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ui-breakdown` | Decompose a UI image into a framework-agnostic 5-tier component hierarchy + composition plan                                                                                           |
| `ui-explore`   | Explore the design space when you're not sure of the direction: generate several directionally different coded variations (Agent Teams) and pick one, or a hybrid.                     |
| `ui-refine`    | Refine an existing UI to excellence: point it at a component, page, section, or a chosen variation, and it runs an autonomous GAN generator/evaluator loop (Agent Teams + Playwright). |


**Code quality**


| Skill        | Does                                                                                                                                                                                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `naturalize` | Make AI/agent-written code read like a careful human wrote it *in this codebase*: strip the AI tells (over-comments, ceremonial guards, `any` escape hatches), conform to local idiom, naming, and density. Behavior-preserving; hands structural changes to `refactor`. |


**Code review**

The piece I'm most attached to. `code-review-dna` reviews in my own rubric and voice, distilled from thousands of my real PR comments, so the feedback reads like I wrote it.


| Skill             | Does                                                                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code-review-dna` | Fast single-pass review of a PR or local diff, in my rubric and review voice. Dispatches the `dna-reviewer` agent.                                             |
| `deep-review`     | Thorough multi-axis review: five parallel axis agents (architecture, implementation, types/naming, hygiene, tests/docs/observability), merged into one review. |


**Pull requests & git flow**


| Skill                 | Does                                                           |
| --------------------- | -------------------------------------------------------------- |
| `polish-prs`          | Finish a batch of PRs: address comments, fix CI, push together |
| `review-comments`     | Fetch, address, and reply to PR review comments                |
| `rebase-resolve-push` | Rebase onto latest main, resolve conflicts, force-push         |
| `fix-ci`              | Diagnose and fix failing CI checks on the current PR           |


**Requirements & meta**


| Skill              | Does                                                                                                                              |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| `interview-user`   | Structured interview to fill knowledge gaps before/while building                                                                 |
| `crystallize`      | Turn the task you just did into a reusable, project-agnostic skill: abstract the recipe, strip the incidentals, keep it agnostic. |
| `mastermind`       | The router: maps intent to the right tool in this stack                                                                           |
| `mastermind-setup` | One-time per-repo configuration (see below)                                                                                       |


### Agents (7)


| Agent                                                                                            | Does                                                                                                                                                                                                                             |
| ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `refactor`                                                                                       | Improve code without changing behavior: **subtractive** (cut complexity, kill duplication, remove premature abstraction) and **constructive** (SOLID, design patterns, extract/restructure), scoped to a diff or a codebase area |
| `dna-reviewer`                                                                                   | Single-pass review analyzer: reads the rubric + voice, returns structured findings. Dispatched by `code-review-dna`.                                                                                                             |
| `axis-architecture`, `axis-implementation`, `axis-types-naming`, `axis-hygiene`, `axis-coverage` | The five review axes, each scoped to one rubric tier (or the coverage doc). Dispatched in parallel by `deep-review`.                                                                                                             |


### House-style rules (4)

These aren't invoked. They're the standards I hold, loaded into every session once `mastermind-setup` installs them (Claude Code doesn't auto-load a plugin's `rules/`, so setup copies them into your repo or imports them from your `CLAUDE.md`).


| Rule                      | Does                                                                                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `architecture-principles` | Contract-first, modularity, converge-on-final-state, idempotency, reuse-first, exhaust-the-design-space, build-the-lever, fix-adjacent-problems                |
| `definition-of-done`      | Testing, validation, docs, stories, architectural quality: the bar every change clears                                                                         |
| `agent-teams-guidance`    | When to use a single session vs subagents vs agent teams; lead/worker roles; task decomposition; model routing                                                 |
| `critical-rules`          | The non-negotiables every session follows: autonomy default, reuse-first, the definition of done, fix-what-you-find, plus a canary (see [Canaries](#canaries)) |


### Process artifacts (`recommendations/`)

Optional, repo-level companions to the code-review skills, offered (not forced) by `mastermind-setup`: `CONVENTIONS.md` is the review rubric rewritten as forward-facing standards your team can learn before they trip over them, `LARGE_PR_WORKFLOW.md` is the circuit-breaker policy for when a change gets too big to review inline, and `ci/` holds a GitHub PR-size-guard Action plus a language-specific lint-offload example. The idea of moving mechanical nitpicks into tooling is agnostic; the ESLint config is just one example you adapt to your own linter.

## Canaries

One of the rules in `critical-rules` is a canary. It tells the agent to address me as "Mastermind" at the start of every response, and it's mixed in among the genuine rules with no hint of why it's there.

The greeting itself is beside the point. I think the hardest part of working with an agent over a long session is noticing when its context has quietly rotted: when it's still producing confident output but has started dropping the latent rules and constraints you set early on. A canary is an early warning for that case. The greeting costs nothing and depends on nothing, so the moment the agent stops saying "Mastermind," I know its attention to my instructions has slipped, usually before that slip shows up somewhere expensive, and I can re-ground it or start a fresh session. Right now that recovery is manual. Down the line I want to explore a canary detector that watches an agent's thread on its own, and the moment the canary goes silent, auto-heals the agent by reinjecting the dropped rules and summarizing the thread back down to what still matters.

It only works if the agent treats it as just another rule, so the reasoning lives here in the docs and never in the rule the agent actually loads. If the agent knew the greeting was a tripwire, I hypothesize it would over-index on remembering it and the signal would be worthless.

Use whatever word you like ("Mastermind" fits the stack; your own name works just as well), and add more canaries if you want. The mechanism is the point.

## Configuration

Most of the tools work with zero setup. The few that touch your project (the code reviewer, the definition-of-done rule) need to know a couple of things, like how you run checks and tests. Run `/mastermind-setup` once per repo and it detects all of that and writes a `mastermind.config.json` for you. You can also write it by hand:

```json
{
	"checkCommand": "make check",
	"testCommand": "make test",
	"storybook": false,
	"docsDir": "docs",
	"coverageMatrix": null,
	"reviewAutoPost": true
}
```

The surface is deliberately small, and it grows only as a new skill genuinely needs a key. Anything it doesn't know falls back to whatever your `CLAUDE.md` documents, then to sensible defaults.

## What's here, and what isn't

The agnosticism is a hard line, not something I'll relax later for convenience. If a skill only works with one component library, one database, or one test runner, it doesn't belong here, full stop. UI skills have to work for any framework. I'd rather ship a smaller set of tools that travel everywhere than a big pile that only fire in one stack.

More will land over time as I pull other pieces out of the projects they grew up in and make them general enough to earn a spot.

## License

MIT © Steven Secreti. It's my stack, but if any of it is useful to you, take it and make it your own.