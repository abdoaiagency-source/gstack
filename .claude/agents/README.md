# gstack specialists as native Claude Code subagents

This directory turns gstack's specialist roster into native Claude Code
[subagents](https://docs.claude.com/en/docs/claude-code/sub-agents). Each
`*.md` file is a self-contained subagent: a role identity, a distilled
methodology lifted from the matching gstack skill, and a report-back contract.
Claude Code auto-delegates to them based on the `description` field, or you can
call one explicitly ("use the code-reviewer subagent on this diff").

These are **additive**. The full gstack skills (`/review`, `/cso`, `/ship`, ...)
still install via `./setup` and remain the richer, interactive surface. The
subagents are the lightweight, parallel-friendly, separate-context version of
the same roster, packaged the way Claude Code ships specialists natively.

## The roster

| Subagent | Source skill | What it does |
|----------|--------------|--------------|
| `ceo-reviewer` | `/plan-ceo-review` | CEO/founder-mode plan review: thinks bigger, challenges scope, ties to user value |
| `eng-reviewer` | `/plan-eng-review` | Eng-manager-mode architecture review: locks design, names risks, finds the simpler path |
| `design-plan-reviewer` | `/plan-design-review` | Designer's-eye plan review: UX, flows, and missing states before code |
| `design-reviewer` | `/design-review` | Visual design QA + fix: spacing, hierarchy, AI-slop patterns, slow interactions |
| `design-consultant` | `/design-consultation` | Builds a complete design system from scratch (type, color, layout, motion) |
| `code-reviewer` | `/review` | Pre-landing PR review: production bugs, confidence-scored, line-quoted |
| `security-auditor` | `/cso` | OWASP Top 10 + STRIDE audit, 8/10 gate, exploit scenario per finding |
| `qa-engineer` | `/qa` | Browser QA of a running app, then fixes the bugs it finds |
| `qa-reporter` | `/qa-only` | Browser QA, report only, no fixes |
| `debugger` | `/investigate` | Root-cause debugging under the Iron Law: no fix without a reproduced cause |
| `product-strategist` | `/office-hours` | YC Office Hours diagnostic + builder brainstorm, anti-sycophancy |
| `retro-lead` | `/retro` | Weekly engineering retrospective from real git history |
| `perf-benchmarker` | `/benchmark` | Performance regression detection via the browse daemon |
| `devex-auditor` | `/devex-review` | Live developer-experience audit: onboarding time, friction, fixes |
| `plan-pipeline` | `/autoplan` | Runs CEO -> design -> eng -> DX review sequentially, auto-decides, one plan |
| `release-engineer` | `/ship` | Ship prep: tests, version bump, CHANGELOG, staged commits (stops before push/PR) |
| `deployer` | `/land-and-deploy` | Land + deploy + canary plan (stops before merge/deploy) |
| `release-documenter` | `/document-release` | Post-ship doc updates + Diataxis coverage map |
| `doc-author` | `/document-generate` | Generates missing docs from scratch (Diataxis: tutorial/how-to/reference/explanation) |
| `spec-author` | `/spec` | Vague intent -> precise five-phase spec (stops before filing the issue) |
| `codex-consultant` | `/codex` | OpenAI Codex CLI second opinion / outside-voice challenge |

## Design conventions (why these differ from the skills)

A skill and a subagent are not the same execution model. The conversions follow
four rules so each subagent behaves correctly in its separate context:

1. **No `AskUserQuestion`, no `Agent` in the tool list.** A subagent runs to
   completion and returns a single result; it cannot hold an interactive
   dialogue with the user, and it cannot spawn further subagents. Where the
   source skill asks the user, the subagent instead surfaces the decision as a
   ranked recommendation in its report. Where the source skill dispatches a
   parallel "review army", the subagent runs those passes sequentially itself.

2. **Self-contained.** No gstack-internal binaries, no `~/.gstack/...` state, no
   telemetry, no upgrade/skill-routing scaffolding. Each subagent works even if
   gstack itself is not installed. Browser-driving subagents (`qa-engineer`,
   `qa-reporter`, `perf-benchmarker`, `design-reviewer`, `devex-auditor`) use
   the gstack `browse` binary if it is present and fall back to Playwright,
   reporting a blocker if neither exists.

3. **Handback before irreversible actions.** Subagents that touch outward-facing,
   hard-to-reverse surfaces do all the safe prep and then STOP, returning the
   exact commands for you to confirm and run. `release-engineer` never pushes or
   opens the PR. `deployer` never merges or deploys. `spec-author` never files
   the issue. `codex-consultant` flags that content leaves the machine and scans
   for secrets first.

4. **One voice.** Every subagent inherits the GStack voice: builder-to-builder,
   concrete, no em dashes, no AI vocabulary, decisions closed with user impact.

`model: inherit` on every subagent so they run on whatever model you have
selected for the main session.

## Regenerating / extending

These were distilled from the `SKILL.md` files at the repo root (one skill
directory per specialist). To add a specialist, copy the frontmatter shape from
any file here, keep the four conventions above, and distill only the skill's
unique methodology (the bottom of its `SKILL.md`), not the shared runtime
scaffolding at the top.
