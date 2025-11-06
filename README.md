# Claude Code Plugins

A marketplace of personal Claude Code plugins - custom API integrations and productivity tools.

## What's Inside

### Plugins

**API Integrations**
- **searching-slack-history** - Search Slack message history using Web API (search.messages, conversations.history)

## Installation

```bash
# In Claude Code
/plugin marketplace add ralphbean/claude-code-plugins
/plugin install searching-slack-history
```

## Structure

This repository is a **marketplace** containing multiple **plugins**:

```
claude-code-plugins/           (marketplace repository)
├── .claude-plugin/
│   └── marketplace.json       (lists all available plugins)
└── plugins/
    └── searching-slack-history/    (individual plugin)
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── searching-slack-history/
                └── SKILL.md
```

Each plugin can be installed independently via `/plugin install <plugin-name>`.

## Marketplace Aggregation

This marketplace automatically syncs selected plugins from other marketplaces using **[claude-marketplace-sync](https://github.com/ralphbean/claude-marketplace-sync)**:

- **Synced from [chmouel/claude-code-plugins](https://github.com/chmouel/claude-code-plugins)**: `deslop` plugin
  - Automatically synced weekly via GitHub Actions
  - Tagged with `chmouel` prefix for provenance tracking
  - Other plugins from chmouel's marketplace are filtered out via denylist

Want to create hierarchical marketplaces for your organization? Check out **[claude-marketplace-sync](https://github.com/ralphbean/claude-marketplace-sync)** - a tool for aggregating multiple Claude Code marketplaces with denylist support and provenance tracking.

Perfect for organizations that want:
- Department-level marketplaces that roll up to org-level
- Distributed ownership of plugin collections
- Selective inclusion/exclusion of plugins from child marketplaces
- Automatic provenance tracking showing where each plugin came from

## Adding New Plugins

To add a new plugin to this marketplace:

1. Create directory: `plugins/your-plugin-name/`
2. Add `.claude-plugin/plugin.json` with metadata
3. Add skills in `skills/your-skill-name/SKILL.md`
4. Update `marketplace.json` to include the new plugin
5. Commit and push

## Contributing

This is a personal plugin marketplace. Feel free to fork and adapt for your own use!

## License

MIT License - see LICENSE file for details
