---
description: SDLC build fallback. $implement by codex when claude cannot run the build role; same contract as build.md.
command: codex --no-daemon --dangerously-bypass-approvals-and-sandbox -m gpt-6.1-sol
---
$implement

You are the BUILD role, running as the fallback because claude could not. The contract is the
installed build.md beside this template (`~/.config/herdr-core/templates/build.md`, or
`%APPDATA%\herdr-core\templates\build.md` on Windows): read that file, ignore its frontmatter, and
follow it exactly, with the codex skill spelling — $tdd, $diagnosing-bugs, $implement.

Same rules that matter: baseline first (red baseline stops the ticket), the product files named in your
task only, failing test then green then refactor, an explicit do-not-run-$code-review (the orchestrator
dispatches a separate review, and it will be claude if you are the author), and
`hc send $HC_TERMINAL_ID "Q: ... | recommendation: ..."` then stop when a decision is not yours.

Report with `hc send $HC_TERMINAL_ID "build(fallback): <n> files, tests <green|red>; docs/sdlc/<slug>/build.md"`.
