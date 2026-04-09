# turbotools

A curated collection of Claude Code plugins from [mysticflounder](https://github.com/mysticflounder).
Polished, general-purpose tools for everyday use with Claude Code.

## Installation

```
/plugin install <plugin-name>@turbotools
```

Or browse in `/plugin > Discover`.

## Plugins

### domain-experts

Specialist consultant personas for structured critique and domain-specific analysis.

| Skill | Command | Description |
|-------|---------|-------------|
| DB Engineer | `/domain-experts:db-engineer` | Authoritative review of database design, SQL schema correctness, engine behavior, data modeling, and technology selection. Covers SQLite, PostgreSQL, MySQL, Redis, and NoSQL. |
| Film Critic | `/domain-experts:film-critic` | Professional script coverage and structural analysis for film/TV outlines, screenplays, beat sheets, and scene breakdowns. |
| UX Expert | `/domain-experts:ux-expert` | Structured feedback on UI/UX design — layout, interaction patterns, information architecture, navigation flows, accessibility, and visual hierarchy. |

### session-wrap

End-of-session knowledge preservation. Reviews the conversation and saves anything worth remembering to the auto-memory system so the next session starts with full context.

| Skill | Command | Description |
|-------|---------|-------------|
| Session Wrap | `/session-wrap:session-wrap` | Scans the conversation for new project context, corrections, preferences, infrastructure changes, and reference info. Writes memories, updates docs, and creates a handoff file for the next session. |

### specforge

Spec authoring and design tools for structured YAML specifications.

| Skill | Command | Description |
|-------|---------|-------------|
| Specforge Design | `/specforge:specforge-design` | Converts a 1-3 sentence module description into a valid specforge YAML spec through interactive interview. |

## See also

- **[super-cow-powers](https://github.com/mysticflounder/super-cow-powers)** — Debian packaging skill with deep dh-virtualenv support. Available as its own marketplace.

## Plugin structure

Each plugin follows the standard Claude Code layout:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # required: name, description, author
├── skills/              # subdirs with SKILL.md → /{plugin}:{skill} slash commands
├── commands/            # flat .md files → /{plugin}:{stem} slash commands
├── agents/              # .md files injected into subagent system prompts
├── hooks/               # hooks.json + scripts → lifecycle event handlers
└── output-styles/       # custom output formatting
```

## Contributing

Plugins here are promoted from a private development repo when ready for
public release.
