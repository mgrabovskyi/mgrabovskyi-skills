# CLAUDE.md

This repo is a single [Claude Code plugin](https://docs.claude.com/en/docs/claude-code/plugins) containing skills for engineering leaders. Skills are organized into bucket folders by *kind of work*, not by job title.

## Repo layout

- `.claude-plugin/plugin.json` — plugin manifest (name, version, description, author).
- `.claude-plugin/marketplace.json` — single-entry marketplace pointing at this plugin, so the standard `/plugin marketplace add` / `/plugin install` flow works.
- `skills/` — all skills, organized into buckets:
  - `engineering/` — daily code work (writing, reviewing, debugging, refactoring).
  - `management/` — people, process, operational queues (triage, hiring, reviews, retros).
  - `productivity/` — cross-cutting daily workflow tools (rollups, scheduled tasks, communication automation).
  - `in-progress/` — drafts not yet ready to ship. Excluded from the top-level `README.md`.
  - `deprecated/` — retired skills, kept so old links and intent survive. Excluded from the top-level `README.md`.
- `docs/` — notes, conventions, longer-form references.

## Where a new skill goes — decision rule

1. **Does it write or review code?** → `skills/engineering/`
2. **Does it run a people/process/queue workflow?** → `skills/management/`
3. **Is it cross-cutting daily workflow that doesn't fit either?** → `skills/productivity/`
4. **Not ready to ship?** → `skills/in-progress/`
5. **Unsure?** Pick the closest bucket and move it later. Buckets are organizational, not load-bearing — Claude triggers skills by their `description`, not by their path.

## Skill format

A skill is a directory containing `SKILL.md`:

```markdown
---
name: skill-name
description: One sentence describing when Claude should use this skill. Name concrete triggers — situations, phrases the user might say, or states the conversation might be in. Not topics.
---

# Body
Instructions, examples, conventions. Keep it focused — long skills get skipped or skimmed.
```

**The `description` is load-bearing.** Claude reads it to decide whether to invoke the skill. Write it from the perspective of "use this when…" — name concrete trigger situations, not just the topic.

- Bad: `Helps with performance reviews.`
- Good: `Use when the user is drafting a performance review, calibrating ratings, or writing manager feedback for a direct report.`

## Progressive disclosure

Skills that need supporting material (templates, longer references, scripts) put those files **alongside `SKILL.md`** in the same directory. The main `SKILL.md` stays short and references the supporting files by relative path. Claude loads them on demand.

Example:

```
skills/engineering/tdd/
├── SKILL.md           # short entry point, references the rest
├── mocking.md
├── refactoring.md
└── tests.md
```

Prefer this over writing one giant `SKILL.md`. Long single-file skills get skimmed or truncated.

## Listing rules

- Every skill in `engineering/`, `management/`, or `productivity/` **must** have:
  1. A one-line entry in its bucket's `README.md`.
  2. A one-line entry in the top-level `README.md` reference section.
- Skills in `in-progress/` and `deprecated/` **must not** appear in the top-level `README.md`.
- Each bucket's `README.md` lists every skill in the bucket with a one-line description, with the skill name linked to its `SKILL.md`.

## Versioning

- Bump the plugin's `version` in `.claude-plugin/plugin.json` on every shipped change.
- Semver: patch for tweaks, minor for new skills, major for renaming or removing skills.

## Conventions

- One skill per directory; the directory name matches the `name` in frontmatter.
- Prefer editing an existing skill over creating a near-duplicate.
- Don't add boilerplate skills (`hello-world`, etc.) — every skill should solve a real recurring task.
- Don't preemptively create `agents/`, `commands/`, `hooks/`, or `mcp-servers/` directories. Add them when there's actual content.
- When deprecating a skill, move it to `skills/deprecated/<skill-name>/` and add a one-line note at the top of its `SKILL.md` explaining why it was deprecated and what (if anything) supersedes it. Remove it from the top-level and bucket `README.md`.
