---
name: create-tutorial
description: Creates an interactive Claude Code tutorial or setup wizard for any repository. Analyzes the repo's documentation, setup steps, and configuration to produce a guided, conversational experience that walks users from clone to fully functional.
disable-model-invocation: true
---

# Create Tutorial

Build an interactive Claude Code tutorial or setup wizard for a repository. The result is a set of files that live inside the target repo so that a user can run a single slash command and be guided through setup, configuration, or learning — conversationally, step by step.

---

## Phase 1: Analyze the Repository

Before writing anything, deeply understand the repo you're building a tutorial for.

### What to read

- **README and setup docs** — Every prerequisite, install step, configuration option, and dependency.
- **Example configs** — Sample YAML/JSON/TOML files, `.env.example`, etc. Understand every field.
- **Provider or integration docs** — If the repo connects to external services, read those guides. Understand what credentials are needed, what scopes/permissions, and where users create them.
- **CLI tools or scripts** — Read help text, flags, and common invocations. Understand what the user runs vs. what can be automated.
- **Existing issues or FAQs** — Common stumbling blocks tell you where the tutorial needs extra clarity.

### What to extract

Build a mental model of:

1. **Prerequisites** — What must be installed or available before starting?
2. **Decision points** — Where does the user choose between options? (e.g., which provider, which deployment mode, which features)
3. **External actions** — What must the user do outside the terminal? (e.g., create API tokens, enable OAuth apps, configure webhooks)
4. **Automatable steps** — What can Claude execute directly? (e.g., creating files, running CLI commands, generating configs)
5. **Sensitive data** — What values are secrets that must never appear in conversation history?
6. **Validation checks** — How do you confirm each step succeeded?

---

## Phase 2: Design the Flow

### Principles

- **Linear by default.** The user moves forward step by step. Avoid branching trees — use conditional sections within a linear path instead.
- **Minimize friction.** No "are you ready?" prompts. No unnecessary confirmations. Jump straight into the first real question.
- **Ask only at real decision points.** Don't ask when there's a sensible default. Don't ask when Claude can detect the answer.
- **Give instructions for external actions.** When the user must leave the terminal (e.g., to create tokens), tell them exactly where to go, what to click, and what to come back with. Provide direct URLs when possible.
- **Never handle secrets.** Instruct users to run sensitive commands in a separate terminal. Never accept, echo, or store secrets in conversation.
- **Track everything collected.** All user inputs feed into later steps (file generation, config creation). Keep a running record.

### Structure

Organize the tutorial into **phases**. Each phase groups related steps. A typical setup wizard has:

1. **Welcome + choices** — What are we setting up? Which options does the user want?
2. **Credential/prerequisite setup** — One section per integration. External actions + secret storage.
3. **Configuration generation** — Use everything collected to generate config files, templates, or resources.
4. **Validation** — Run the result, check for errors, confirm success.

Keep the total phase count low. A setup wizard rarely needs more than 3-5 phases. A learning tutorial can have more, but each should be completable in a few minutes.

---

## Phase 3: Create the File Structure

The tutorial lives inside the target repo under `.claude/`. Create these files:

### 1. Slash command launcher: `.claude/commands/<name>.md`

This is what the user invokes with `/<name>`. It should be minimal — its only job is to silently load the behavioral rules and script, then begin.

```markdown
---
description: <One-line description of what this command does>
---

<Title of the tutorial>

Do this SILENTLY — don't announce what you're doing:

1. Read `.claude/<name>-instructions.md` for behavioral rules
2. Read `.claude/<name>-script.md` for the tutorial content
3. Begin immediately — start with the welcome message from the script
```

The `description` field in the YAML frontmatter appears in Claude Code's command list, so make it clear and user-facing (e.g., "Interactive setup wizard" or "Guided onboarding tutorial").

If the tutorial needs tools beyond the defaults (Read, Write, Bash, Glob, Grep), add an `allowed-tools` list in the frontmatter.

### 2. Behavioral rules: `.claude/<name>-instructions.md`

This file tells Claude **how to behave** during the tutorial. It is read once at the start and governs the entire session. Include these sections:

#### Core behavior rules

- **No fourth-wall breaking.** Never mention scripts, instruction files, or internal mechanics. Act as a knowledgeable guide.
- **Follow the script.** Execute each step in order. Don't skip, reorder, or improvise unless the user asks.
- **Respect STOP markers.** When the script says `STOP:`, pause and wait for user input. Never continue past a STOP without a response.
- **Handle USER markers.** `USER:` describes expected input. Accept reasonable variations.
- **Execute ACTION markers.** `ACTION:` means Claude runs a command or performs a file operation. Do it, then report the result.
- **Process conditional blocks.** `[Bracketed text]` is conditional — only show/execute it if the condition is met.

#### Pacing

- Wait for user responses at every STOP point.
- Don't rush, but stay focused on outcomes.
- Never add filler prompts ("Ready?", "Shall we begin?", "Let's go!").

#### Error handling

- If a command fails, show the error and suggest a fix before continuing.
- If the user seems lost, offer to clarify the current step.
- If a step can't be completed (missing tool, permission denied), acknowledge it and offer an alternative or skip.

#### Security

- Never ask the user to paste secrets or tokens into the conversation.
- Never run secret-storage commands yourself — always instruct the user to do it in a separate terminal.
- If a user accidentally pastes a secret, do not echo it. Remind them to use a separate terminal and move on.

#### Input collection

- When presenting choices, use numbered lists. Users reply with numbers (e.g., "1" or "1, 3").
- Keep prompts short and scannable. Bold the option names, one-line descriptions.
- Track all collected input — it feeds into file/config generation later.

#### Links and external resources

- When directing users to external pages (token creation, OAuth setup, documentation), provide the full URL.
- Tell the user what they're going to and why.

#### Success criteria

- At the end of the behavioral rules file, define what "done" looks like. E.g., "The tutorial is complete when [specific config file] exists and [validation command] succeeds."

### 3. Tutorial script: `.claude/<name>-script.md`

This is the actual content of the tutorial. It uses a marker system that the behavioral rules teach Claude to interpret.

#### Marker reference

| Marker | Meaning |
|---|---|
| `STOP:` | Pause and wait for user input. Include what you're waiting for. |
| `USER:` | Describes the expected user response. |
| `ACTION:` | Claude executes this (command, file write, etc.). |
| `[Condition]` | Conditional block — only if the condition is true. |
| `---` | Phase or section break. |

#### Script template

```markdown
# <Tutorial Title>

## Phase 1: <Phase Name>

<Welcome message and context. Set expectations — what will this tutorial do?>

<First real question or prompt for the user.>

STOP: Wait for the user's response.

USER: <Description of expected input>

ACTION: <What Claude does with the input>

<Feedback to the user about what happened.>

[If user chose option X]
## Phase 2a: <Option X Setup>

<Instructions specific to this choice. Include URLs for external actions.>

<Command the user runs in a separate terminal, formatted as a code block:>

```
<sensitive-command example>
```

**Do not paste the value here — run this in a separate terminal.**

STOP: Wait for user to confirm they've completed this step.

USER: Confirms completion.

ACTION: <Validation step Claude runs to check it worked>
[End condition]

## Phase N: <Final Phase>

ACTION: <Generate config files, run final setup, etc.>

<Success message. Tell the user what was created, what to do next, and where to find things.>

---

## Important Notes for Claude

- <Edge case handling>
- <Fallback behaviors>
- <Things to watch out for>

## Success Criteria

- [ ] <Specific, verifiable outcome 1>
- [ ] <Specific, verifiable outcome 2>
- [ ] <Specific, verifiable outcome 3>
```

### 4. Project context: `CLAUDE.md` (root)

If the repo doesn't already have a `CLAUDE.md`, create one. It provides ambient context that Claude loads automatically.

Include:
- One-paragraph project description
- Key file paths and what they contain
- Available slash commands (including the new tutorial)
- Any repo conventions Claude should know about

If a `CLAUDE.md` already exists, add the new slash command to it rather than replacing it.

---

## Phase 4: Write the Content

With the structure in place, write the actual tutorial content.

### Content guidelines

- **Be specific.** Don't say "create an API token." Say "Go to Settings → Developer → Personal Access Tokens → Generate New Token. Select these scopes: [list]. Set expiration to [recommendation]."
- **Show, don't tell.** When Claude generates a file, show a brief summary of what was generated and where it was saved. Don't dump the entire file into chat.
- **Use code blocks for commands.** Every command the user needs to run should be in a fenced code block they can copy.
- **Provide direct URLs.** Link to the exact page, not the documentation homepage.
- **Include examples.** When asking for input (like a repository name or team ID), show the expected format: "Enter your repo (e.g., `org/repo-name`)."
- **Validate after each critical step.** After credential setup, run a test. After config generation, verify the file. After deployment, check status.
- **End with next steps.** Tell the user what to do after the tutorial. Link to documentation, common workflows, or advanced configuration.

### What Claude generates vs. what the user does

Be deliberate about this split:

| Claude does | User does |
|---|---|
| Create/edit config files | Create API tokens and credentials |
| Run non-sensitive CLI commands | Store secrets via separate terminal |
| Validate setup and report results | Approve choices and provide input |
| Generate templates from collected input | Configure external services (webhooks, OAuth) |

---

## Phase 5: Update the README

The tutorial only works if people know it exists. Update the repo's README so the guided setup is the **first thing users see** in the getting-started section.

### What to add

Add a section near the top of the README (before any manual setup instructions) with this structure:

```markdown
## Quick Start with Claude Code

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed.

```bash
git clone <repo-url> && cd <repo-name> && claude /setup
```

That's it — the wizard walks you through the rest.
```

Key rules:

- **One command.** Clone, cd, and launch the wizard in a single copy-paste line. The user should go from zero to running wizard with one action.
- **Put it before manual instructions.** The guided path should be the default, prominent option. Manual steps become the fallback for users who don't have Claude Code.
- **List only real prerequisites.** If the wizard handles installing or checking dependencies, don't list them here. Only list things the user must have before they can run the command (Claude Code itself, and any system-level tools the wizard can't install).
- **Keep it short.** The README section is a signpost, not documentation. The wizard itself handles explanation and guidance.

### If manual instructions already exist

Don't remove them. Rename the existing section to something like "Manual Setup" or "Setup Without Claude Code" and place it after the guided setup section. Add a horizontal rule (`---`) between them to create clear visual separation.

---

## Phase 6: Validate and Refine

After creating all files:

1. **Read through the complete flow** end to end. Check that every STOP has a matching USER, every ACTION is achievable, and every conditional block closes properly.
2. **Check that no secrets are handled in conversation.** Every credential step should instruct the user to use a separate terminal.
3. **Verify all URLs are correct and point to the right pages.**
4. **Ensure the generated config/template logic accounts for all combinations** of user choices.
5. **Confirm the success criteria are testable** — each checkbox should be something you can verify with a command or file check.
6. **Verify the README** has the one-line quick start command and it points to the correct repo URL and slash command name.

---

## Key Lessons and Anti-patterns

### Do

- Jump straight into the first meaningful question.
- Use numbered lists for choices — users type a number instead of a sentence.
- Provide the exact `cd` path, exact URL, exact command — no ambiguity.
- Design for the happy path first, then handle edge cases in the "Important Notes for Claude" section at the end of the script.
- Keep the slash command launcher tiny. All logic lives in the instructions and script files.
- Separate behavioral rules from content. The instructions file is reusable; the script is specific.
- Make the README's quick start a single `git clone ... && cd ... && claude /<command>` line. One copy-paste action from zero to wizard.

### Don't

- Don't ask "are you ready?" or "shall we continue?" — just continue.
- Don't accept secrets in conversation, ever, for any reason.
- Don't use interactive terminal UI features that may not render in all environments. Stick to plain text prompts with numbered options.
- Don't make Claude announce what files it's reading or what internal steps it's taking.
- Don't put all content in a single file. The three-file split (launcher, rules, script) keeps things maintainable.
- Don't forget to track user inputs throughout — they feed into the final generation step.
- Don't include validation steps that require services or APIs that might not be set up yet.
