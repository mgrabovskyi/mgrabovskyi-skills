# mgrabovskyi-skills

A personal collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills, agents, and commands — packaged as a plugin marketplace so they can be installed selectively.

The collection is organized around three roles I work across:

| Plugin | Purpose |
|---|---|
| [`eng-leadership`](plugins/eng-leadership) | Director/VP-level work — strategy, planning, OKRs, exec & board communication, org design |
| [`engineering`](plugins/engineering) | Individual engineering — code review, architecture, design docs, debugging, refactors |
| [`eng-management`](plugins/eng-management) | People & process — 1:1s, performance reviews, hiring, interview prep, retros, postmortems |

Cross-cutting or role-agnostic skills (e.g. summarizing transcripts, drafting briefs) live at the top-level [`skills/`](skills) directory rather than inside a plugin.

## Install

Add this repo as a marketplace in Claude Code, then install the plugins you want:

```bash
/plugin marketplace add mgrabovskyi/mgrabovskyi-skills
/plugin install eng-leadership@mgrabovskyi-skills
/plugin install engineering@mgrabovskyi-skills
/plugin install eng-management@mgrabovskyi-skills
```

To use a single top-level skill without installing a whole plugin, copy or symlink it into `~/.claude/skills/`.

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest — lists the plugins below
├── plugins/
│   ├── eng-leadership/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/             # skills scoped to this role
│   ├── engineering/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   └── eng-management/
│       ├── .claude-plugin/plugin.json
│       └── skills/
├── skills/                     # cross-cutting / role-agnostic skills
├── docs/                       # notes, conventions, examples
└── README.md
```

Each plugin can also contain `agents/`, `commands/`, `hooks/`, and `mcp-servers/` subdirectories — added per the [Claude Code plugin spec](https://docs.claude.com/en/docs/claude-code/plugins) as needed.

## Adding a new skill

A skill is a directory with a `SKILL.md` file. The frontmatter `description` is what Claude uses to decide when to invoke the skill, so it should be specific about triggers.

```
plugins/<plugin>/skills/<skill-name>/
└── SKILL.md
```

Minimal `SKILL.md`:

```markdown
---
name: skill-name
description: One sentence describing when Claude should use this skill — be specific about triggers.
---

# Skill body
Instructions, examples, and anything else Claude should know.
```

For role-agnostic skills, drop the directory under top-level `skills/` instead.

## Adding a new plugin

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json` with `name`, `version`, `description`, and `author`.
2. Add the plugin to `.claude-plugin/marketplace.json`.
3. Bump the plugin's `version` whenever you ship a breaking change.

## License

[MIT](LICENSE)
