# Claude Code Skills — EISART PM Workflows

Shared Claude Code skills for the EISART project at Kroger.

## Skills

### `pm-review`
End-of-sprint PM Review for EISART teams (Product Rangers & Agents Of Item).

**What it does:**
- Fetches all items in "PM Review" status for the active sprint
- Analyzes each item: AC, fix version, PR status, diff summary, evidence
- Lets you close or send back each item with a single command
- Maintains a running tally of story points reviewed

---

## Installation

```bash
# Clone the repo
git clone https://github.com/mtahil/claude-skills.git

# Copy skills to your Claude Code skills directory
cp -r claude-skills/pm-review ~/.claude/skills/
```

Then in Claude Code, run:
```
/pm-review
```

## Updating

```bash
cd claude-skills
git pull
cp -r pm-review ~/.claude/skills/
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) installed
- `gh` CLI authenticated with access to `krogertechnology` GitHub org
- Atlassian MCP plugin configured with access to Kroger Jira
