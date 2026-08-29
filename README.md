# mastermind-stack

> The skills, review workers, and house-style rules I use to get high-quality work out of Codex and Claude Code, pulled out of my own projects and made to run anywhere.

Coding agents are only as good as the engineer driving them. I've spent years building up a way of working: the standards I hold myself to, the patterns I reach for, the voice I review in. The more I leaned on coding agents, the more I noticed I was re-teaching them the same things in every repo, so I pulled all of that accumulated craft out into one portable place. This is that.

I think the most valuable thing you can hand an agent is taste: a clear bar for what good looks like, and the discipline to subtract before you add. So the tools here lean on that. They reach for deletion before addition, they hold a real definition of done, and they review against standards I actually believe in rather than generic best-practice filler.

Everything is deliberately agnostic. No skill assumes a particular framework, library, language, test runner, or database, since I want the same bar whether I'm in a personal project, a side business, or my day job. When a tool genuinely needs to know something about your repo, like how you run tests, it reads it from a small config instead of hardcoding it.

And the whole thing is self-describing. A router skill maps whatever you're trying to do to the right tool, and every skill explains itself, so you (and the agent) can always find your way around.

## Install

### Codex

Add the repository marketplace and install the plugin:

```sh
codex plugin marketplace add stevensecreti/mastermind-stack
codex plugin add mastermind-stack@mastermind-stack
```

The repository also ships a native `.codex-plugin/plugin.json` manifest, so it can
be imported from GitHub in the Codex workspace marketplace. Once the plugin is
listed in the public Codex directory, the same package can be installed with the
directory's Install button.

Start a new task in the target repository and invoke `$mastermind-setup` once so
the plugin can detect the project's commands and install its standing rules into
`AGENTS.md`.

### Claude Code

```sh
claude plugin marketplace add stevensecreti/mastermind-stack
claude plugin install mastermind-stack@mastermind-stack
```

Then run `/mastermind-setup` once per repository. Claude Code installs the standing
rules under `.claude/rules/` and wires them into `CLAUDE.md`.

## What's inside

Each tool does one job. If you're not sure which one you want, the `mastermind` router will point you at it.

### Skills (22)

Six review-worker skills are internal building blocks used by the two review
orchestrators. The other 16 can be invoked directly in either host.

**UI design & analysis**


| Skill          | Does                                                                                                                                                                                   |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ui-breakdown` | Decompose a UI image into a framework-agnostic 5-tier component hierarchy + composition plan                                                                                           |
| `ui-explore`   | Explore the design space with several directionally different coded variations, using parallel workers when the host supports them, then pick one or a hybrid.                         |
| `ui-refine`    | Refine an existing UI with a generator/evaluator loop and real browser evidence when browser automation is available.                                                                   |


**Code quality**


| Skill        | Does                                                                                                                                                                                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `naturalize` | Make agent-written code read like a careful human wrote it in this codebase: remove AI tells and conform to local idiom, naming, and density.                                                                                                                        |
| `refactor`   | Improve code without changing behavior through subtractive simplification or constructive restructuring.                                                                                                                                                           |


**Code review**

The piece I'm most attached to. `code-review-dna` reviews in my own rubric and voice, distilled from thousands of my real PR comments, so the feedback reads like I wrote it.


| Skill             | Does                                                                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code-review-dna` | Fast single-pass review of a PR or local diff in my rubric and review voice. Uses the `dna-reviewer` worker skill.                                              |
| `deep-review`     | Thorough multi-axis review across architecture, implementation, types/naming, hygiene, and tests/docs/observability, merged into one review.                   |

Internal review workers: `dna-reviewer`, `axis-architecture`,
`axis-implementation`, `axis-types-naming`, `axis-hygiene`, and `axis-coverage`.


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


**Writing**


| Skill         | Does                                                                                                                                    |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `throughline` | Apply the Throughline meaning-first prose style to the current task.                                                                    |


### Claude Code agent adapters (7)

Claude Code can still dispatch the original seven agent definitions. Each is now
a thin adapter over the corresponding canonical skill, which prevents the Codex
and Claude implementations from drifting.


### Output styles (1)


| Style         | Does                                                                                                                                                                                                            |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Throughline` | Meaning-first prose: short subject-verb-object sentences, plain words, each idea chained to the next, assertions over hedges, every word earning its keep. Coding behavior is untouched; only the writing changes. |


**Using it in Claude Code**

1. Install the plugin (see [Install](#install)). The style ships in `output-styles/` and Claude Code discovers it automatically; no extra setup, and `mastermind-setup` is not involved.
2. Run `/config`, select **Output style**, and pick **Throughline**. Plugin styles appear unnamespaced in the picker, merged with the built-ins. (The old `/output-style` command was removed in v2.1.91; `/config` is the current path.)
3. To pin it instead of picking it per session, set `"outputStyle": "Throughline"` in `.claude/settings.json` for one project or `~/.claude/settings.json` for everywhere.

The style sets `keep-coding-instructions: true`, so Claude Code keeps its built-in software-engineering instructions and Throughline only governs how responses read.

**Using it in Codex or another harness**

Invoke `$throughline` in Codex. The canonical skill contains the same writing spec
as the Claude output style. You can also put the body of
[`output-styles/throughline.md`](output-styles/throughline.md) wherever another
tool accepts standing instructions.

### House-style rules (4)

These aren't invoked. They're the standards I hold, loaded into every session once
`mastermind-setup` installs them. Codex gets a managed block in the repository's
`AGENTS.md`; Claude Code gets `.claude/rules/` imports from `CLAUDE.md`.


| Rule                      | Does                                                                                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `architecture-principles` | Contract-first, modularity, converge-on-final-state, idempotency, reuse-first, exhaust-the-design-space, build-the-lever, fix-adjacent-problems                |
| `definition-of-done`      | Testing, validation, docs, stories, architectural quality: the bar every change clears                                                                         |
| `agent-teams-guidance`    | When to use one session versus parallel workers; lead/worker roles, task decomposition, and safe shared-workspace coordination                                |
| `critical-rules`          | The non-negotiables every session follows: autonomy default, reuse-first, the definition of done, fix-what-you-find, plus a canary (see [Canaries](#canaries)) |


### Process artifacts (`recommendations/`)

Optional, repo-level companions to the code-review skills, offered (not forced) by `mastermind-setup`: `CONVENTIONS.md` is the review rubric rewritten as forward-facing standards your team can learn before they trip over them, `LARGE_PR_WORKFLOW.md` is the circuit-breaker policy for when a change gets too big to review inline, and `ci/` holds a GitHub PR-size-guard Action plus a language-specific lint-offload example. The idea of moving mechanical nitpicks into tooling is agnostic; the ESLint config is just one example you adapt to your own linter.

## Canaries

One of the rules in `critical-rules` is a canary. It tells the agent to address me as "Mastermind" at the start of every response, and it's mixed in among the genuine rules with no hint of why it's there.

The greeting itself is beside the point. I think the hardest part of working with an agent over a long session is noticing when its context has quietly rotted: when it's still producing confident output but has started dropping the latent rules and constraints you set early on. A canary is an early warning for that case. The greeting costs nothing and depends on nothing, so the moment the agent stops saying "Mastermind," I know its attention to my instructions has slipped, usually before that slip shows up somewhere expensive, and I can re-ground it or start a fresh session. Right now that recovery is manual. Down the line I want to explore a canary detector that watches an agent's thread on its own, and the moment the canary goes silent, auto-heals the agent by reinjecting the dropped rules and summarizing the thread back down to what still matters.

It only works if the agent treats it as just another rule, so the reasoning lives here in the docs and never in the rule the agent actually loads. If the agent knew the greeting was a tripwire, I hypothesize it would over-index on remembering it and the signal would be worthless.

Use whatever word you like ("Mastermind" fits the stack; your own name works just as well), and add more canaries if you want. The mechanism is the point.

## Configuration

Most tools work with zero setup. The few that touch your project need to know how
you run checks and tests. Run `$mastermind-setup` in Codex or
`/mastermind-setup` in Claude Code once per repository; it detects those details
and writes `mastermind.config.json`. You can also write it by hand:

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

The surface is deliberately small, and it grows only as a new skill genuinely
needs a key. Anything it doesn't know falls back to the repository's `AGENTS.md`,
`CLAUDE.md`, or equivalent instructions, then to sensible defaults.

## What's here, and what isn't

The agnosticism is a hard line, not something I'll relax later for convenience. If a skill only works with one component library, one database, or one test runner, it doesn't belong here, full stop. UI skills have to work for any framework. I'd rather ship a smaller set of tools that travel everywhere than a big pile that only fire in one stack.

More will land over time as I pull other pieces out of the projects they grew up in and make them general enough to earn a spot.

## License

MIT © Steven Secreti. It's my stack, but if any of it is useful to you, take it and make it your own.
