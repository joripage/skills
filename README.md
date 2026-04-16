# Claude Code Skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that provide domain-specific expertise and guidance.

## Available Skills

| Skill | Description |
|-------|-------------|
| [fix-protocol](fix-protocol/) | FIX (Financial Information eXchange) protocol expert. Covers message flows, session management, order state machines, and debugging for FIX 4.2 / 4.4 / 5.0 SP2. |

## Installation

Copy any skill folder into your Claude Code skills directory:

```bash
# Install a single skill
cp -r fix-protocol ~/.claude/skills/

# Or install all skills
cp -r */ ~/.claude/skills/
```

## Usage

Once installed, skills can be invoked directly in Claude Code:

```bash
# Use a skill by name
/fix-protocol "your question or task"
```

Skills also auto-activate when Claude detects relevant context in the code you're working on.

## Project Structure

```
skills/
├── README.md
└── fix-protocol/
    ├── SKILL.md              # Main skill definition and rules
    ├── README.md             # Skill-specific documentation
    └── references/           # On-demand reference files
        ├── error-codes.md
        ├── fix-version-differences.md
        ├── order-state-machine.md
        ├── required-fields.md
        └── session-management.md
```

## Creating a New Skill

Each skill is a folder containing at minimum a `SKILL.md` file with frontmatter:

```yaml
---
name: skill-name
description: "When to use this skill"
argument-hint: "[usage hint]"
metadata:
  author: your-name
  version: "1.0.0"
---
```

Place reference files in a `references/` subfolder to keep the main context clean -- they are loaded on demand.

## License

This project is licensed under the [MIT License](LICENSE).
