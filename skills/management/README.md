# Management

Skills for people, process, and operational queues — triage, hiring, reviews, retros.

- **[linear-triage-automation](./linear-triage-automation/SKILL.md)** — Event-driven Linear triage: duplicate detection, priority parsing from free text, owner assignment via Triage Intelligence with a routing-table fallback, and moving issues out of the Triage column. Expects a routing table and a fallback owner from the calling team.
- **[linear-handoff](./linear-handoff/SKILL.md)** — Hand off an engineer's open Linear tickets to another engineer with full context. Gathers Linear state plus git branch and PR status, composes a structured handoff comment per ticket, and reassigns only after explicit confirmation. Use for PTO coverage, role changes, team transfers, or sprint transitions. Requires a Linear MCP, git, and the gh CLI.
