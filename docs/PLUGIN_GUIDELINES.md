# Plugin Development Guidelines

Guidelines for developing and contributing plugins to the Mondoplatz marketplace.

## Plugin Structure

Each plugin should follow this structure:

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── commands/                    # Slash commands (optional)
│   └── your-command.md
├── skills/                      # Skills (optional)
│   └── your-skill/
│       ├── SKILL.md
│       └── references/
├── agents/                      # Custom agents (optional)
│   └── your-agent.md
└── README.md                    # Plugin documentation
```

## Plugin Metadata (plugin.json)

```json
{
  "name": "plugin-your-name",
  "description": "Brief description of your plugin",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "repository": "https://github.com/esmondo/mondoplatz",
  "keywords": ["keyword1", "keyword2"]
}
```

## Skills Best Practices

1. **Progressive Disclosure**: Keep the main SKILL.md small (~2KB), use references/ for detailed content
2. **Clear Triggers**: List clear trigger keywords that activate your skill
3. **Token Efficiency**: Use grep patterns for finding specific sections in large reference files
4. **Domain Expertise**: Focus on specific, deep knowledge rather than broad, shallow coverage

## Commands Best Practices

1. **Clear Names**: Use descriptive command names (e.g., `/plugin-name:action`)
2. **Documentation**: Include usage examples in the command markdown
3. **YAML Frontmatter**: Use frontmatter for command metadata

## Testing Your Plugin

Before submitting:

1. Test plugin installation locally
2. Verify all commands work as expected
3. Test skill triggering with various inputs
4. Check that all file paths are relative to plugin root
5. Ensure README.md is comprehensive

## Submission Process

1. Fork the mondoplatz repository
2. Create a new branch: `git checkout -b plugin/your-plugin-name`
3. Add your plugin to `plugins/your-plugin-name/`
4. Update `/.claude-plugin/marketplace.json` to include your plugin
5. Update main `/README.md` to list your plugin
6. Submit a pull request

## Marketplace Listing Requirements

Your plugin should:
- Have a clear, descriptive name
- Include comprehensive documentation
- Follow the standard plugin structure
- Be tested and functional
- Provide unique value to Claude Code users

---

Questions? Open an issue on GitHub!
