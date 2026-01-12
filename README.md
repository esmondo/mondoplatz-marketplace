# Mondoplatz

A curated marketplace of Claude Code plugins by Mondo.

## Available Plugins

### 🧠 Neurotech
Comprehensive neuroscience, neurotechnology, and BCI knowledge base for Claude Code.

**Features:**
- MNE-Python guidance for EEG/MEG analysis
- BCI development (P300, SSVEP, motor imagery)
- Signal processing and artifact removal
- Mental health biomarkers
- Neurotech industry analysis

[📖 Plugin Documentation](./plugins/neurotech/README.md)

### 🤖 Business Autopilot
Department-based AI agents to run your business on autopilot.

**Features:**
- 14 specialized agents across 7 departments (Engineering, Product, Marketing, Design, PM, Operations, Testing)
- Workflow commands for common business tasks
- Chainable agent workflows for complex processes
- Customizable agent prompts

[📖 Plugin Documentation](./plugins/business-autopilot/README.md)

## Installation

### Adding the Marketplace

```bash
/plugin marketplace add esmondo/mondoplatz
```

### Installing Plugins

#### Neurotech Plugin
```bash
/plugin install plugin-neurotech@mondo
```

#### Business Autopilot Plugin
```bash
/plugin install business-autopilot@mondo
```

Then restart Claude Code and verify:
```bash
/plugin
```

## Plugin Development

This marketplace follows the Claude Code plugin architecture. Each plugin is contained in the `plugins/` directory with its own:
- `.claude-plugin/` - Plugin configuration
- `commands/` - Slash commands
- `skills/` - Knowledge base skills
- `agents/` - Custom agents (optional)

## Structure

```
mondoplatz/
├── README.md                      # This file
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── plugins/
│   ├── neurotech/               # Neurotech plugin
│   │   ├── .claude-plugin/
│   │   ├── commands/
│   │   ├── skills/
│   │   └── README.md
│   └── business-autopilot/      # Business Autopilot plugin
│       ├── .claude-plugin/
│       ├── agents/
│       ├── commands/
│       ├── skills/
│       └── README.md
└── docs/                         # Additional documentation
```

## Contributing

Want to add your plugin to Mondoplatz? Open an issue or PR!

---

Built by [Mondo](https://github.com/esmondo) for the Claude Code community.
