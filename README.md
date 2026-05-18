# mgrabovskyi-skills

**[Claude Code](https://docs.claude.com/en/docs/claude-code) skills for engineering leaders.**

The coding-agent ecosystem optimizes for engineers writing code. An engineering leader's job is wider than that — code, operational queues, communication, decisions. These skills are the parts of that wider job I've shaped Claude around. They're small, opinionated, and meant to make your week *less* reactive, not just faster.

## Quickstart (30-second setup)

```bash
/plugin marketplace add mgrabovskyi/skills
/plugin install mgrabovskyi-skills@mgrabovskyi-skills
```

That installs the plugin. Skills load on demand from their `description` frontmatter — you describe what you want, the right skill triggers. No commands to memorize.

## Why these skills exist

I built these to fix three failure modes I kept hitting when applying a coding agent across the whole job, not just code.

### #1 — The agent overbuilds and "improves" things you didn't ask it to touch

> "Perfection is achieved not when there is nothing more to add, but when there is nothing left to take away."
>
> — Antoine de Saint-Exupéry

**The Problem.** You ask for a small change. You get a sprawling diff. Adjacent code gets "improved." Variables get renamed. Error handling appears for impossible cases. Abstractions appear "for flexibility." Reviewing the PR takes longer than writing it would have.

**The Fix** is to load discipline rules **before** the agent writes anything:

- [`karpathy-coding-guidelines`](./skills/engineering/karpathy-coding-guidelines/SKILL.md) — surfaces assumptions, enforces minimal diffs, demands verifiable success criteria, and pushes back against speculative abstraction or unrequested cleanup.

The bar to clear: every changed line traces directly back to what you asked for.

> [!TIP]
> Install this skill on the repo *before* the first agent-written change, not after. Once a codebase has been "improved" by an undisciplined agent, the cleanup cost is higher than the original work.

### #2 — Triage is half mechanical work and half judgment, and the mechanical half eats your week

> "There is nothing so useless as doing efficiently that which should not be done at all."
>
> — Peter Drucker

**The Problem.** Most issue triage is mechanical: parse priority hints, route by domain, dedupe. Linear's native features (Triage Rules, Triage Intelligence) already cover the deterministic half. The judgment half — "this `_Priority:_ p1` buried in the description means Urgent", "this is the same root cause as INF-2104" — is what actually burns calendar time when a human does it.

**The Fix** is to layer agent behavior *on top of* Linear's native triage, not in place of it:

- [`linear-triage-automation`](./skills/management/linear-triage-automation/SKILL.md) — runs when an issue enters Triage. Handles only the fuzzy parts: free-text priority parsing, duplicate confirmation, domain inference, and the decision to move out of Triage or leave for a human. Org-agnostic — takes a routing table and a fallback owner from the calling team.

<details>
<summary>What this skill <em>refuses</em> to do</summary>

The skill is explicit about scope. Anything expressible as a Triage Rule belongs in Linear, not in the agent:

- `label → assignee`, `label → priority`, `label → team` → Triage Rule
- Assignee suggestions, related-issue surfacing → Triage Intelligence (read its output; don't duplicate it)
- Free-text priority parsing, duplicate confirmation, domain inference from title + description → this skill

If a triage task can be expressed deterministically, it does not belong in an agent loop.

</details>

### #3 — Signal is scattered across Slack, Teams, and email, and the morning catch-up has no edges

> "Clarity about what matters provides clarity about what does not."
>
> — Cal Newport, *Deep Work*

**The Problem.** Your morning starts with hundreds of unread items across multiple tools. Most are noise. The useful items — explicit asks, places you were mentioned, things that shipped, things that are blocked — are buried. You spend the first hour of every day doing the same synthesis job.

**The Fix** is a scheduled task that does the synthesis before you sit down:

- [`morning-briefing`](./skills/productivity/morning-briefing/SKILL.md) — configures a recurring scheduled task. Each morning it pulls the last 24h from Slack/Teams + email and posts a four-section summary (needs from me, mentions, updates, blockers) to a destination you pick (Slack self-DM, private channel, or email).

The skill *builds and installs* the task. The task does the daily work. You spend the morning reading the synthesis, not building it.

> [!TIP]
> Pair the Slack self-DM delivery target with mobile push and your morning briefing reaches you before you open a laptop. Best leverage I've found for a five-second investment.

### #4 — Handoffs between engineers lose context, and the new owner pays the tax

> "Adding manpower to a late software project makes it later."
>
> — Frederick P. Brooks Jr., *The Mythical Man-Month*

**The Problem.** An engineer goes on PTO, switches teams, or leaves. Their tickets get reassigned — usually with a one-line comment like "handing this to Bob" — and the new owner spends a day reconstructing where the work actually is. What branch? Which PR? What's already been decided in review? What did the last comment thread agree on? Context lives in three places (Linear, git, GitHub), and the reassignment carries none of it.

**The Fix** is to make context the **precondition** for reassignment, not the afterthought:

- [`linear-handoff`](./skills/management/linear-handoff/SKILL.md) — given two engineers, gathers Linear state plus git branch and PR status for every open ticket, composes a structured handoff comment per ticket (what's done, what's left, key decisions, suggested next steps, quick-start command), shows the drafts back to you for confirmation, and only then posts and reassigns. Requires a Linear MCP, git, and `gh`.

The skill posts the comment **before** reassigning — the new owner never sees the ticket in their queue without the context already on it.

> [!TIP]
> Run it once before PTO and once again the day before you return. The pre-return run catches anything that moved while you were out, with the same context discipline applied in reverse.

### #5 — Multi-session strategy work loses state, and you re-derive it every time

> "The faintest ink is more powerful than the strongest memory."
>
> — Chinese proverb

**The Problem.** The biggest leader work — strategy, roadmap, hiring loops, postmortems, OKR shaping — doesn't fit in one chat. You hit the context limit mid-thought, or you stop for the day. When you come back, the first twenty minutes go to re-explaining where things landed, what was decided, and what's still open. The agent re-asks questions you already answered. Decisions that were *closed* get quietly re-litigated. The cost compounds across sessions.

**The Fix** is to compact each session into a *structured* artifact — not a free-form chat summary — that the next session can load as state:

- [`session-handoff`](./skills/productivity/session-handoff/SKILL.md) — produces a leader-shaped brief (decisions made, decisions still open, asks, stakes, what the next session should do) and writes it to a destination you pick: temp file, project state file, Notion page, or Slack canvas. The next session reads it, picks up exactly where you left off, and never re-asks resolved questions.

The bar to clear: a fresh agent reading the handoff should be able to **act**, not summarize.

> [!TIP]
> For work that spans weeks — a hiring loop, an OKR draft, a vendor selection — make the destination a Notion page in append mode. Each session adds a dated section; the page becomes the canonical state, and the chat is just the working surface.

### Summary

The bet behind this collection: the highest-leverage thing an engineering leader can do with an agent is **not** "have it write more code." It's **shape your existing work so less of it lands on your calendar**. These skills are the part of that bet I've shipped publicly.

## Reference

### Engineering — daily code work

- **[karpathy-coding-guidelines](./skills/engineering/karpathy-coding-guidelines/SKILL.md)** — Karpathy-style coding discipline: surface assumptions, favor minimal diffs, define verifiable success criteria, avoid speculative abstraction. Use before letting Claude write or edit application code.

### Management — people, process, operational queues

- **[linear-triage-automation](./skills/management/linear-triage-automation/SKILL.md)** — Event-driven Linear triage: duplicate detection, free-text priority parsing, owner assignment via Triage Intelligence with a routing-table fallback, and moving issues out of the Triage column.
- **[linear-handoff](./skills/management/linear-handoff/SKILL.md)** — Hand off an engineer's open Linear tickets to another engineer with full context (Linear state + git branch + PR status), as a confirmed batch. PTO coverage, role changes, team transfers.

### Productivity — cross-cutting daily workflow

- **[morning-briefing](./skills/productivity/morning-briefing/SKILL.md)** — Recurring daily rollup of Slack/Teams + email activity, delivered as a four-section summary (asks, mentions, updates, blockers) at the time and destination you pick.
- **[session-handoff](./skills/productivity/session-handoff/SKILL.md)** — Compact the current conversation into a structured leader-shaped handoff (decisions made, decisions open, asks, stakes, next-session prompt) and write it to a temp file, project state file, Notion page, or Slack canvas. Distinct from [`linear-handoff`](./skills/management/linear-handoff/SKILL.md): this is chat→chat, not engineer→engineer.

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

1. **Pick a bucket:**
   - Code-writing or code-reviewing work → `skills/engineering/`
   - People, process, or operational queues → `skills/management/`
   - Cross-cutting daily workflow, not code-specific → `skills/productivity/`
   - Not ready to ship → `skills/in-progress/`
2. **Create** `skills/<bucket>/<skill-name>/SKILL.md` with frontmatter:

   ```markdown
   ---
   name: skill-name
   description: One sentence describing when Claude should use this skill — name concrete trigger situations or phrases, not just topics.
   ---

   # Body
   ```

3. **List it** in the bucket's `README.md` and in the top-level `README.md` Reference section.
4. **Bump** the plugin's `version` in `.claude-plugin/plugin.json` (semver — patch for tweaks, minor for new skills, major for renames or removals).

Skills in `in-progress/` and `deprecated/` **do not** appear in the top-level `README.md`.

See [CLAUDE.md](./CLAUDE.md) for the full contributor rules.

## License

[MIT](LICENSE)
