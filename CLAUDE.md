# CLAUDE.md

This repo is a **Claude Code plugin marketplace** containing skills, agents, and commands for engineering leadership, individual engineering, and engineering management work.

## Repo layout

- `.claude-plugin/marketplace.json` — marketplace manifest. Update whenever a plugin is added, removed, or renamed.
- `plugins/<plugin>/` — one directory per plugin. Each has a `.claude-plugin/plugin.json` and a `skills/` subdir.
- `skills/` (top-level) — cross-cutting, role-agnostic skills that don't belong to any single plugin.
- `docs/` — notes, conventions, examples.

The three plugins map to three personas:

| Plugin | Persona | Examples of what belongs here |
|---|---|---|
| `eng-leadership` | Director/VP work | Strategy memos, OKRs, roadmap reviews, exec/board comms, org design |
| `engineering` | Individual engineering | Code review, architecture review, design docs, debugging, refactors |
| `eng-management` | People & process | 1:1 prep, performance reviews, hiring/JDs, interview prep, retros, postmortems |

## Where a new skill goes — decision rule

1. **Does it fit clearly in one persona?** → `plugins/<persona>/skills/<skill-name>/`
2. **Useful across personas, or doesn't map to one?** → top-level `skills/<skill-name>/`
3. **Unsure?** Ask. Don't guess — putting it in the wrong plugin makes installation noisier than it needs to be.

## Skill format

A skill is a directory containing `SKILL.md`:

```markdown
---
name: skill-name
description: One sentence describing when Claude should use this skill. Be specific about triggers.
---

# Body
Instructions, examples, conventions. Keep it focused — long skills are skipped.
```

**The `description` is load-bearing.** Claude uses it to decide whether to invoke the skill. Write it from the perspective of "use this when the user…" — name concrete trigger phrases or situations, not just topics.

Bad: `description: Helps with performance reviews.`
Good: `description: Use when the user is drafting a performance review, calibrating ratings, or writing manager feedback for a direct report.`

## Plugin format

Each plugin needs `plugins/<name>/.claude-plugin/plugin.json` with at minimum:

```json
{
  "name": "<name>",
  "version": "0.1.0",
  "description": "...",
  "author": { "name": "Michael Grabovskyi" }
}
```

Plugins can also contain `agents/`, `commands/`, `hooks/`, `mcp-servers/` per the [Claude Code plugin spec](https://docs.claude.com/en/docs/claude-code/plugins). **Do not create these directories preemptively** — add them when there's actual content.

## Versioning

- Bump the plugin's `version` on every shipped change. Use semver: patch for tweaks, minor for new skills, major for removing/renaming skills.
- The marketplace itself is unversioned — it's a directory listing.

## Conventions

- One skill per directory; the directory name matches the `name` in frontmatter.
- Skills that need supporting files (templates, examples) put them alongside `SKILL.md` in the same directory.
- Prefer editing an existing skill over creating a near-duplicate.
- Don't add boilerplate skills (`hello-world` etc.) — every skill should solve a real recurring task.
