---
description: SDLC deliver fallback. $pr by codex when claude cannot run the deliver role; same contract as deliver.md.
command: codex --no-daemon --dangerously-bypass-approvals-and-sandbox -m gpt-6.1-sol
---
$pr

You are the DELIVER role, running as the fallback because claude could not. The contract is the body of
~/.config/herdr-core/templates/deliver.md: read that file, ignore its frontmatter, and follow it
exactly, with the codex skill spelling — $resolving-merge-conflicts, $pr, $retro.

Same rules that matter: rebase onto origin/main first and never force-push, run the checks
yourself after the rebase, PR body with summary/evidence/merge danger, push the branch but never merge,
deploy or delete data, and write docs/sdlc/<slug>/pr.md with what you left for the human.

Report with `hc send $HC_TERMINAL_ID "deliver(fallback): <branch> <pr link|drafted>; docs/sdlc/<slug>/pr.md"`.
