# create-tutorial

An Agent Skill that creates interactive Claude Code tutorials and setup wizards for any repository. Point it at a repo with complex setup, and it produces a guided, conversational experience that walks users from clone to fully functional — with a single slash command.

## What it produces

A set of files inside the target repo's `.claude/` directory:

- **Slash command launcher** — A minimal file that triggers the tutorial via `/<name>`
- **Behavioral rules** — How Claude should behave during the tutorial (pacing, security, error handling)
- **Tutorial script** — The actual step-by-step content with markers for pauses, user input, conditional logic, and automated actions
- **README update** — A one-liner quick start so users can `git clone && claude /setup`

## Install

### Via the Skills CLI (recommended)

The [Vercel Skills CLI](https://github.com/vercel-labs/add-skill) installs to 37+ agents (Claude Code, Cursor, Codex, Copilot, and more) automatically:

```bash
# Install to all detected agents
npx skills add rkirkendall/create-tutorial-skill

# Install to specific agents
npx skills add rkirkendall/create-tutorial-skill -a claude-code -a cursor

# Install globally (available across all projects)
npx skills add rkirkendall/create-tutorial-skill -g
```

### Manual

Copy the directory into your agent's skills folder:

```bash
# Claude Code
mkdir -p ~/.claude/skills && cp -r create-tutorial ~/.claude/skills/create-tutorial

# Cursor
mkdir -p ~/.cursor/skills && cp -r create-tutorial ~/.cursor/skills/create-tutorial
```

See the [full agent path table](https://github.com/vercel-labs/add-skill#supported-agents) for other agents. Restart your session for the skill to take effect.

## Usage

Once installed, tell the agent:

> "Create an interactive setup tutorial for this repo"

or

> "Build a Claude Code wizard that walks users through getting started"

The skill activates automatically based on these triggers and walks through a phased process: analyze the repo, design the flow, create the file structure, write the content, update the README, and validate.

## Included templates

Starter templates live in `templates/` — they're copy-paste skeletons with placeholders:

| Template | Purpose |
|---|---|
| `command-launcher.md` | The slash command file (`.claude/commands/<name>.md`) |
| `behavior-rules.md` | Behavioral rules for Claude during the tutorial |
| `tutorial-script.md` | Tutorial content with the marker system |

## The marker system

Tutorial scripts use a simple marker language:

| Marker | Meaning |
|---|---|
| `STOP:` | Pause and wait for user input |
| `USER:` | Describes the expected user response |
| `ACTION:` | Claude executes a command or file operation |
| `[Condition]` | Conditional block — only if the condition is true |
| `---` | Phase or section break |

## Key principles baked in

- **One-command quick start** — The README gets a `git clone ... && claude /setup` line
- **No secrets in conversation** — Users are always directed to a separate terminal for credentials
- **No filler prompts** — No "are you ready?" — jumps straight into the first real question
- **Plain text input** — Numbered lists for choices, not interactive UI that may break in some terminals
- **Three-file separation** — Launcher, rules, and script stay separate for maintainability
