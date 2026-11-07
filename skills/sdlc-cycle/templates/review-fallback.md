---
description: SDLC review fallback (Policy A). /code-review by claude opus when codex is out of credits or rate-limited; same contract as review.md.
command: claude --model opus --dangerously-skip-permissions
---
/code-review

You are the REVIEW role, running as the fallback because codex could not. The contract is the body of
~/.config/herdr-core/templates/review.md: read that file, ignore its frontmatter, and follow it
exactly — same two axes, same direct-fix mandate, same review.md artifact.

Report with `hc send $HC_TERMINAL_ID "review(fallback): <b> blockers <m> majors, fixed <n>; docs/sdlc/<slug>/review.md"`.
