# mgrabovskyi-skills

[Claude Code](https://docs.claude.com/en/docs/claude-code) skills for engineering leaders. The agent is the lever; the work is split across writing code, running operational queues, and not missing things in chat and email. These skills shape Claude for that work.

## Quickstart

```bash
/plugin marketplace add mgrabovskyi/skills
/plugin install mgrabovskyi-skills@mgrabovskyi-skills
```

That installs the plugin. Skills are loaded on demand by Claude based on their `description` frontmatter, so you don't run them by name — you describe what you want and the right one triggers.

## What these skills exist to fix

### Problem 1 — Coding agents overbuild and "improve" code you didn't ask them to touch

The most common failure mode for a coding agent: you ask for a small change, you get a sprawling diff. Adjacent code gets "improved." Variables get renamed. Error handling appears for impossible cases. Abstractions get introduced "for flexibility." Reviewing the PR takes longer than writing it would have.

**Fix:** [`karpathy-coding-guidelines`](./skills/engineering/karpathy-coding-guidelines/SKILL.md) — discipline rules the agent loads before writing code. Forces surfacing assumptions, minimal diffs, and verifiable success criteria. Pushes back against speculative abstraction and unrequested cleanup.

### Problem 2 — Triage is half rule-based work and half judgment, and the rule-based half eats your week

Most issue triage is mechanical: parse priority hints, route by domain, dedupe against existing issues. Native Linear features cover the deterministic half. The judgment half — "this `_Priority:_ p1` in the description means Urgent", "this is the same root cause as INF-2104" — is what burns calendar time when a human does it.

**Fix:** [`linear-triage-automation`](./skills/management/linear-triage-automation/SKILL.md) — sits **on top of** Linear's Triage Rules and Triage Intelligence (it does not replace them). Handles only the fuzzy parts: free-text priority parsing, duplicate confirmation, domain inference, and the decision to move an issue out of Triage or leave it for a human. Org-agnostic — takes a routing table and a fallback owner from the calling team.

### Problem 3 — Signal is scattered across Slack, Teams, email, and threads, and the morning catch-up is unbounded

A director's morning starts with hundreds of unread items across multiple tools. Most are noise. The useful items — explicit asks, places you were mentioned, things that shipped, things that are blocked — are buried.

**Fix:** [`morning-briefing`](./skills/productivity/morning-briefing/SKILL.md) — configures a recurring scheduled task that, each morning, pulls the last 24h from Slack/Teams + email and posts a four-section summary (needs from me, mentions, updates, blockers) to a destination you pick. The skill builds and installs the task; the task does the work daily.

## Reference

### Engineering

Skills for daily code work — writing, reviewing, debugging, refactoring.

- **[karpathy-coding-guidelines](./skills/engineering/karpathy-coding-guidelines/SKILL.md)** — Karpathy-style coding discipline: surface assumptions, favor minimal diffs, define verifiable success criteria, and avoid speculative abstraction. Use before letting Claude write or edit application code.

### Management

Skills for people, process, and operational queues — triage, hiring, reviews, retros.

- **[linear-triage-automation](./skills/management/linear-triage-automation/SKILL.md)** — Event-driven Linear triage: duplicate detection, priority parsing, owner assignment via Triage Intelligence with a routing-table fallback, and moving issues out of Triage.

### Productivity

Cross-cutting workflow tools — daily rituals, scheduled rollups, communication automation.

- **[morning-briefing](./skills/productivity/morning-briefing/SKILL.md)** — Recurring daily rollup of Slack/Teams + email activity, delivered as a four-section summary (asks, mentions, updates, blockers).

## Layout

```
.
├── .claude-plugin/
│   ├── plugin.json             # plugin manifest
│   └── marketplace.json        # one-entry marketplace, points at this plugin
├── skills/
│   ├── engineering/            # daily code work
│   ├── management/             # people, process, queues
│   ├── productivity/           # cross-cutting daily workflow
│   ├── in-progress/            # drafts not yet ready to ship
│   └── deprecated/             # retired skills, kept for history
├── docs/                       # notes and conventions
├── CLAUDE.md                   # contributor rules for this repo
└── README.md
```

A skill is a directory containing `SKILL.md`. The directory name matches the `name` in the frontmatter. Supporting files (templates, longer references) live alongside `SKILL.md` in the same directory and are loaded only when the skill needs them.

## Adding a new skill

1. Pick a bucket. The rule:
   - **Code-writing or code-reviewing work** → `skills/engineering/`
   - **People, process, or operational queues** → `skills/management/`
   - **Cross-cutting daily workflow, not code-specific** → `skills/productivity/`
   - **Not ready to ship** → `skills/in-progress/`
2. Create `skills/<bucket>/<skill-name>/SKILL.md` with frontmatter:

   ```markdown
   ---
   name: skill-name
   description: One sentence describing when Claude should use this skill — name concrete trigger situations or phrases, not just topics.
   ---

   # Body
   ```

3. Add a one-line entry in the bucket's `README.md` and in the top-level `README.md` reference section.
4. Bump the plugin's `version` in `.claude-plugin/plugin.json` (semver — patch for tweaks, minor for new skills, major for renames or removals).

Skills in `in-progress/` and `deprecated/` **do not** appear in the top-level `README.md`.

See [CLAUDE.md](./CLAUDE.md) for the full contributor rules.

## License

[MIT](LICENSE)
