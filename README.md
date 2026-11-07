# sdlc-cycle

One command starts a full SDLC loop for one ticket in your repo: **spec → build → review → deliver**,
run by role templates on [herdr-core](https://github.com/herdrdev/herdr-core) panes, with an
orchestrator agent that gates every handoff and only asks you the questions that matter.

```
sdlc ─▶ spec ─▶ build ─▶ [gates] ─▶ review ─▶ [re-check] ─▶ rebase ─▶ deliver ─▶ cleanup
          └ spec-fallback  └ build-fallback        └ review-fallback          └ deliver-fallback
```

You talk to the orchestrator. It writes no code: it cuts the worktree, dispatches `claude` and
`codex` workers on role templates, runs typecheck/lint/tests itself before and after review, relays
the workers' questions, fails over when an agent dies or runs out of credits, and cleans up.

## Install

```bash
npx skills add eunho/sdlc-cycle -g -a pi -a claude-code -a codex -y
```

The skill checks its own dependencies on first use and installs whatever is missing:

```bash
npx skills add mattpocock/skills    -g -a pi -a claude-code -a codex -y   # spec, build, review, deliver, retro
npx skills add backnotprop/pstack   -g -a pi -a claude-code -a codex -y   # principles, craft/verification
npx skills add herdrdev/herdr-core  -g -a pi -a claude-code -a codex -y   # hc dispatch syntax
```

Then, in the repo, in a herdr terminal:

```
/sdlc-cycle T-42 add --version to the CLI
```

or just `sdlc-cycle: T-42 …`, or "take T-42 through the SDLC". The skill installs the ten role
templates into `~/.config/herdr-core/templates/` on first use (never overwriting ones you edited),
runs preflight, and becomes the orchestrator.

## What the skill does, in order

0. **Dependencies** — checks for the eighteen role skills, installs `mattpocock/skills` or
   `backnotprop/pstack` only if something is missing, and stops if an install fails.
1. **Templates** — copies `templates/*.md` into `${HERDR_CORE_TEMPLATES:-~/.config/herdr-core/templates}`
   if missing, then checks `hc templates` reads them.
2. **Preflight** — git repo, clean tree, `origin` with `main`, hc reachable, `$HC_TERMINAL_ID` set.
3. **Orchestrator** — reads the installed `sdlc.md` and follows it. That file, not this README, is
   the contract.

Standing defaults it applies without asking: worker questions answered "as recommended" (only
irreversible, public-interface, data-model or scope-changing calls come to you), the worktree trust
dialog is always accepted, one line per gate, one ticket.

## The ten templates

| Template | Agent | Role |
| --- | --- | --- |
| `sdlc` | pi | orchestrator: worktree, dispatch, gates, relay, failover, cleanup |
| `spec` | claude opus | grill, seams, ADR/glossary, `spec.md` |
| `build` | claude sonnet | failing test → green → refactor at the spec's seams |
| `review` | codex | two axes (standards + spec), direct-fix mandate |
| `deliver` | claude sonnet | rebase, checks, PR body, retro |
| `retro` | pi | batch: turn repeated instructions into structure |
| `spec-fallback` | codex | same spec contract |
| `build-fallback` | codex | same build contract — forces `review-fallback` (author ≠ judge) |
| `review-fallback` | claude opus | Policy A when codex is rate-limited |
| `deliver-fallback` | codex | same delivery contract |

Fallbacks are shims: they invoke the skill in their own spelling (`/name` for claude, `$name` for
codex) and read the primary template's body, so there is one contract per role.

## Customize

Edit the installed templates in `~/.config/herdr-core/templates/` — different models, different
agents, your own check commands. The skill copies a template only when the target is missing, so
your edits survive updates. To pin a flow, edit `sdlc.md`; to change one role's standards, edit that
role's file.

## Requirements

- [herdr-core](https://github.com/herdrdev/herdr-core) (`hc`) with `claude`, `codex` and `pi` on PATH
- [mattpocock/skills](https://github.com/mattpocock/skills) — `grill-with-docs`, `implement`,
  `code-review`, `pr`, `to-spec`, `to-tickets`, `triage`, `wayfinder`, `domain-modeling`,
  `codebase-design`, `setup-matt-pocock-skills`, `tdd`, `diagnosing-bugs`
- [backnotprop/pstack](https://github.com/backnotprop/pstack) — `principle-encode-lessons-in-structure`
  and the craft/verification companions (`tdd` ships here too)
- `retro` and `writing-for-agents` from mattpocock, `herdr-core` from herdrdev
- optional: `resolving-merge-conflicts` — no longer in either tree; the deliver role inlines the rules
- A git repo with an `origin` that has `main`

## License

MIT
