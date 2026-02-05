---
name: memory-maintenance
version: 2.0.0
description: "Intelligent memory management for OpenClaw agents. Reviews daily notes, suggests MEMORY.md updates, maintains directory health, and auto-cleans old files. Uses Gemini free tier for cost efficiency."
homepage: https://github.com/openclaw/skills/memory-maintenance
author: "Ash (Max Hutchinson)"
tags: ["memory", "maintenance", "automation", "gemini", "cleanup"]
metadata:
  openclaw:
    emoji: 🧹
    requires:
      bins: ["gemini", "jq"]
      env: ["GEMINI_API_KEY"]
    install:
      - id: setup
        kind: script
        script: ./scripts/install.sh
        label: "Install memory maintenance"
---

# Memory Maintenance Skill

Intelligent memory management for OpenClaw agents. Reviews daily notes, suggests MEMORY.md updates, maintains directory health, and auto-cleans old files.

## Features

- **Content Review**: Analyzes daily notes and suggests MEMORY.md updates
- **Directory Health**: Monitors memory/ directory for naming issues, fragmentation, bloat
- **Auto-Cleanup**: Archives old reviews (7+ days) and enforces retention policy (30 days)
- **Cost Efficient**: Uses Gemini free tier (1,500 req/day limit, we use ~1/day)
- **Safe by Default**: Content changes require approval; only safe maintenance auto-applies

## Quick Start

```bash
# Install the skill
clawhub install memory-maintenance

# Configure (edit config/settings.json)
{
  "review_time": "23:00",
  "archive_after_days": 7,
  "retention_days": 30,
  "gemini_model": "gemini-2.5-flash"
}

# Run manually
openclaw skill memory-maintenance run

# Or let it run automatically via cron
```

## Architecture

```
memory-maintenance/
├── scripts/
│   ├── review.sh          # Main review script (daily)
│   ├── apply.sh           # Apply approved changes
│   ├── cleanup.sh         # Archive & retention
│   └── install.sh         # Setup
├── config/
│   └── settings.json      # User configuration
└── templates/
    └── review-prompt.txt  # Customizable Gemini prompt
```

## Workflow

1. **Daily Review** (23:00 by default)
   - Scans last 7 days of notes
   - Checks memory/ directory health
   - Generates suggestions via Gemini
   - Outputs: `agents/memory/review-v2-YYYY-MM-DD.{json,md}`

2. **Human Review**
   - Read `agents/memory/review-v2-YYYY-MM-DD.md`
   - Approve/reject suggestions

3. **Apply Changes**
   ```bash
   # Dry run
   openclaw skill memory-maintenance apply --dry-run 2026-02-05
   
   # Apply safe changes (archiving, cleanup)
   openclaw skill memory-maintenance apply --safe 2026-02-05
   
   # Apply all (requires confirmation)
   openclaw skill memory-maintenance apply --all 2026-02-05
   ```

4. **Auto-Cleanup** (runs after successful review)
   - Archives reviews older than 7 days
   - Deletes archive files older than 30 days
   - Cleans up error logs

## Configuration

Edit `config/settings.json`:

```json
{
  "schedule": {
    "enabled": true,
    "time": "23:00",
    "timezone": "Europe/London"
  },
  "review": {
    "lookback_days": 7,
    "gemini_model": "gemini-2.5-flash",
    "max_suggestions": 10
  },
  "maintenance": {
    "archive_after_days": 7,
    "retention_days": 30,
    "consolidate_fragments": true,
    "auto_archive_safe": true
  },
  "safety": {
    "require_approval_for_content": true,
    "require_approval_for_delete": true,
    "trash_instead_of_delete": true
  }
}
```

## Safety

- **Content suggestions**: Never auto-applied
- **Safe maintenance** (archiving): Auto-applied with `--safe`
- **Risky operations** (delete, rename): Require `--all` + confirmation
- **Trash recovery**: Deleted files go to `agents/memory/.trash/`

## Cost

- **1 Gemini request/day** (~2,000-5,000 tokens)
- **Free tier**: 1,500 requests/day
- **Actual usage**: <0.1% of free tier
- **Cost**: £0

## Commands

```bash
# Run review manually
openclaw skill memory-maintenance review

# Apply changes
openclaw skill memory-maintenance apply [--dry-run|--safe|--all] DATE

# Run cleanup
openclaw skill memory-maintenance cleanup

# Check status
openclaw skill memory-maintenance status

# View stats
openclaw skill memory-maintenance stats
```

## Integration with MEMORY.md

The skill suggests updates to these MEMORY.md sections:
- Agent Identity and Core Preferences
- Infrastructure/Setup
- Memory Management
- Backup & Migration
- Contacts
- Scheduled Operations
- Content Creation & Projects
- Active Projects

## Troubleshooting

**"Gemini failed"**
- Check `GEMINI_API_KEY` is set in `.env`
- Verify Gemini CLI is installed: `gemini --version`

**"No suggestions generated"**
- Check daily notes exist in `memory/YYYY-MM-DD.md`
- Review error logs in `agents/memory/error-*.txt`

**"Too many maintenance tasks"**
- Run `openclaw skill memory-maintenance apply --safe` to archive old files
- Adjust `archive_after_days` in config

## Files

### Output
- `agents/memory/review-v2-YYYY-MM-DD.json` — Structured suggestions
- `agents/memory/review-v2-YYYY-MM-DD.md` — Human-readable report
- `agents/memory/stats.json` — Aggregate statistics

### Archive
- `agents/memory/archive/YYYY-MM/` — Monthly buckets
- `agents/memory/.trash/` — Recoverable deletions

## License

MIT — Free to use, modify, distribute.

---

*Part of the Hybrid Agent Architecture. Built with Gemini free tier for sustainable automation.*
