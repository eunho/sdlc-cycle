---
name: sdlc-cycle
description: "Run a full SDLC cycle for one ticket in this repo: spec, build, two-axis review, delivery, cleanup, by dispatching the herdr-core role templates from the sdlc orchestrator. Use when the user says sdlc-cycle, 'sdlc cycle', 'run the loop', 'take this ticket through the SDLC', 'implement, review and ship T-42', or gives a ticket and wants it specced, built, reviewed and delivered without further prompting."
---

# SDLC cycle

One ticket, end to end: `spec → build → review → deliver`, gated by the orchestrator, with the
human only answering the questions that matter. The contract lives in the templates, not here:
this skill gathers the inputs, installs the templates if needed, checks the repo, and starts the loop.

## 0. Dependencies — check first, install only what is missing

`hc` (herdr-core CLI), and a terminal started inside herdr so `$HC_TERMINAL_ID` is set.

The role templates invoke skills by name, so these must exist as skill directories in
`~/.agents/skills` (or `%USERPROFILE%\.agents\skills` on Windows, or `~/.claude/skills`):

```
mattpocock/skills    grill-with-docs implement code-review pr to-spec to-tickets
                     triage wayfinder domain-modeling codebase-design setup-matt-pocock-skills
backnotprop/pstack   tdd diagnosing-bugs resolving-merge-conflicts
retro role           retro writing-for-agents principle-encode-lessons-in-structure
herdr-core           herdr-core  (hc dispatch syntax)
```

```sh
missing=$(for s in grill-with-docs implement code-review pr to-spec to-tickets triage wayfinder \
  domain-modeling codebase-design setup-matt-pocock-skills tdd diagnosing-bugs \
  resolving-merge-conflicts retro writing-for-agents principle-encode-lessons-in-structure \
  herdr-core; do
  [ -e "$HOME/.agents/skills/$s" ] || [ -e "$HOME/.claude/skills/$s" ] || echo "$s"
done)
echo "$missing"
```

Empty → go to templates. Anything missing → say which in one line, then install the pack that
provides it, globally, so pi, claude and codex all see it:

```bash
npx skills add mattpocock/skills    -g -a pi -a claude-code -a codex -y
npx skills add backnotprop/pstack   -g -a pi -a claude-code -a codex -y   # tdd, diagnosing-bugs, conflicts
npx skills add herdrdev/herdr-core  -g -a pi -a claude-code -a codex -y   # hc dispatch syntax
```

One skill missing from a pack you already have → `--skill <name>` for just that one. An install that
fails stops the cycle: a role without its skill runs the task without its method, which is worse than
not starting.

## Inputs

Take from the user's message, in this order: the **ticket** (id, link, or one-line description) and
an optional **slug**. Derive the slug from the ticket id when it is missing: lowercase, dashes, at
most 12 characters (`T-42` → `t42`). Worker names are `<slug>-<role>`, and hc caps names at 32
characters, so a long slug breaks the cycle.

No ticket given: read `docs/agents/issue-tracker.md`, pick the top `ready-for-agent` ticket, and say
which one you picked in one line. Ask only if two tickets are equally ready.

## 1. Templates

Target directory, in this order:

1. `$HERDR_CORE_TEMPLATES` when set — honour it, wherever you are.
2. Windows: `%APPDATA%\herdr-core\templates` (verified: hc on Windows reads roaming `APPDATA`, not
   `%USERPROFILE%\.config` and not `%LOCALAPPDATA%`).
3. macOS/Linux: `~/.config/herdr-core/templates`.

The `templates/` folder beside this SKILL.md holds all ten role templates. For each file there, if
the target does not have it, copy it in. Never overwrite a template the user already has — their
edits win — and never edit the installed copies from this skill.

```sh
# macOS / Linux
mkdir -p "${HERDR_CORE_TEMPLATES:-$HOME/.config/herdr-core/templates}"
for f in <this skill dir>/templates/*.md; do
  cp -n "$f" "${HERDR_CORE_TEMPLATES:-$HOME/.config/herdr-core/templates}/"
done
hc templates --json   # must list sdlc, spec, build, review, deliver
```

```powershell
# Windows
$target = if ($env:HERDR_CORE_TEMPLATES) { $env:HERDR_CORE_TEMPLATES } else { "$env:APPDATA\herdr-core\templates" }
New-Item -ItemType Directory -Force -Path $target | Out-Null
Copy-Item "<this skill dir>\templates\*.md" $target -ErrorAction SilentlyContinue
hc templates --json
```

## 2. Preflight — run it, report only what fails

```sh
git rev-parse --show-toplevel          # must be a git repo
git status --short                     # must be empty
git remote get-url origin              # must exist
git rev-parse --verify origin/main     # base for the worktree
```

- `docs/agents/issue-tracker.md` missing → say so and continue; `spec` runs
  `/setup-matt-pocock-skills` and will ask the user about the tracker and labels.
- `$HC_TERMINAL_ID` unset → stop and tell the user to run this from a herdr terminal or a pane the
  Herdr app started; the loop dispatches workers through hc.
- Dirty tree, or no `origin/main` → stop, name the problem, let the user fix it. Never invent a base
  branch.

## 3. Become the orchestrator

Read the installed `sdlc.md` in full, ignore its frontmatter, and follow it exactly for this ticket
and slug. That file is the whole loop: worktree, dispatch shapes, deterministic gates, panel labels,
trust rule, question relay, stall and failover rules, cleanup.

Start at preflight inside it. Do not restate the plan to the user first and do not ask them to
confirm it — the plan is the template.

## Standing decisions for this cycle

Apply these for the whole cycle without asking again:

- **Default answer to worker questions: "as recommended".** Answer those yourself with
  `hc send <worker> "decision: as recommended"`. Escalate only when the recommendation is wrong, or
  the call is irreversible, changes a public interface or the data model, or exceeds the ticket's
  scope. When you escalate: two sentences, the recommendation, your objection, two options, then stop.
- **Always trust the worktree folder.** Never ask the user about that dialog.
- **You write no code, no spec, no review.** Dispatch the role; never do its work.
- **One line per gate.** `phase pass|fail — artifact path`. No prose between gates unless something
  needs the user.
- **One ticket.** When it ships, report branch, PR, and what is left, then run cleanup and stop.

## Ending and repeating

End of cycle: report the branch, the PR (or `pr.md` when origin is bare), the retro notes, what the
human must do (merge, deploy), and whether the branch was kept or deleted.

The user says "next" or "keep going" → start a new cycle for the next `ready-for-agent` ticket: new
slug, new worktree, new artifacts. Never reuse a worktree across tickets.
