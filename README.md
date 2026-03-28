# turbotools

A curated collection of Claude Code plugins from [mysticflounder](https://github.com/mysticflounder).
Polished, general-purpose tools for everyday use with Claude Code.

## Installation

```
/plugin install <plugin-name>@turbotools
```

Or browse in `/plugin > Discover`.

## Plugin Structure

Each plugin follows the standard Claude Code layout:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # required: name, description, author
├── skills/              # optional
├── commands/            # optional
├── agents/              # optional
└── README.md
```

## Contributing

Plugins here are promoted from the private `claude-plugins` repo when ready for
public release. See that repo for the promotion workflow.
