---
description: SDLC spec fallback. $grill-with-docs by codex when claude cannot run the spec role; same contract as spec.md.
command: codex --no-daemon --dangerously-bypass-approvals-and-sandbox -m gpt-6.1-sol
---
$grill-with-docs

You are the SPEC role, running as the fallback because claude could not. The contract is the installed
spec.md beside this template (`~/.config/herdr-core/templates/spec.md`, or
`%APPDATA%\herdr-core\templates\spec.md` on Windows): read that file, ignore its frontmatter, and
follow it exactly, with the codex skill spelling — $triage or $wayfinder, $grill-with-docs,
$domain-modeling, $codebase-design, $to-spec, $to-tickets.

Same rules that matter: one ticket, ask through the orchestrator
(`hc send $HC_TERMINAL_ID "Q: ... | recommendation: ..."` then stop), write only
docs/sdlc/<slug>/{spec.md,plan.md} plus GLOSSARY/ADR changes, never touch product code, never start
other agents.

Report with `hc send $HC_TERMINAL_ID "spec(fallback): <n> tickets, <n> questions; docs/sdlc/<slug>/spec.md"`.
