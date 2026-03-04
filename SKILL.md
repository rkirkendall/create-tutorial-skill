---
name: create-tutorial
description: Creates an interactive Claude Code tutorial or setup wizard for any repository through a collaborative dialogue. Analyzes the repo, asks clarifying questions, proposes an outline for user approval, then generates the tutorial files. Use when asked to create a tutorial, setup wizard, onboarding flow, or getting-started guide for a repo.
disable-model-invocation: true
---

# Create Tutorial

Build an interactive Claude Code tutorial or setup wizard for a repository. This is a **collaborative process** — you and the user design the tutorial together through a series of checkpoints before any files are generated.

The end result is a set of files inside the target repo's `.claude/` directory so that a user can run a single slash command and be guided through setup, configuration, or learning.

**This skill is a dialogue, not a script.** You will:

1. Analyze the repo and present your findings
2. Ask questions about anything unclear
3. Propose a tutorial outline and get approval
4. Draft each section and confirm before finalizing
5. Generate the files only after alignment

Never silently generate tutorial files without the user reviewing and approving the plan first.

---

## Phase 1: Analyze the Repository

Read the repo thoroughly before talking to the user. Understand what the project does, how it's set up, and where the friction is.

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

### CHECKPOINT 1: Present your findings

After analyzing, present a summary to the user. Format it like this:

> **Here's what I found in the repo:**
>
> - **What it does:** [one-sentence project summary]
> - **Prerequisites:** [list]
> - **Setup steps I identified:** [numbered list of the major steps a user goes through]
> - **Decision points:** [where users choose between options]
> - **External actions required:** [things users do outside the terminal — tokens, OAuth, etc.]
> - **Things I can automate:** [file generation, CLI commands, validation]
> - **Secrets/credentials needed:** [list, noting these will never be handled in conversation]
>
> **Questions I have:**
> - [Anything unclear, ambiguous, or undocumented]
> - [Anything that could go multiple ways and you need the author's intent]
> - [Any steps where you're unsure of the order or dependencies]
>
> **Does this look right? Anything I'm missing or getting wrong?**

Wait for the user to respond. They may correct your understanding, fill in gaps, add context about their intended audience, or clarify priorities. Incorporate their feedback before moving on.

If you have no questions, still present the findings summary and ask the user to confirm or correct before proceeding.

---

## Phase 2: Propose the Tutorial Outline

Based on your analysis and the user's feedback, propose a structured outline for the tutorial. This is where the user shapes the flow before you write anything.

### Design principles to follow

- **Linear by default.** The user moves forward step by step. Avoid branching trees — use conditional sections within a linear path instead.
- **Minimize friction.** No "are you ready?" prompts in the final tutorial. No unnecessary confirmations. Jump straight into the first real question.
- **Ask only at real decision points.** Don't ask when there's a sensible default. Don't ask when Claude can detect the answer.
- **Give instructions for external actions.** When the tutorial user must leave the terminal (e.g., to create tokens), tell them exactly where to go, what to click, and what to come back with. Provide direct URLs when possible.
- **Never handle secrets.** Instruct tutorial users to run sensitive commands in a separate terminal. Never accept, echo, or store secrets in conversation.

### CHECKPOINT 2: Present the outline for approval

Present the proposed tutorial flow as a numbered outline. For each phase, show:

- The phase name
- What happens in that phase (bullet points)
- What the tutorial user will be asked or told to do
- What Claude will automate

Example format:

> **Proposed tutorial outline:**
>
> **Phase 1: Welcome + Provider Selection**
> - Welcome message explaining what this tutorial sets up
> - User picks which integrations to configure (numbered list)
>
> **Phase 2: GitHub Setup** *(conditional — only if selected)*
> - Instructions to create a Personal Access Token with link to GitHub's token page
> - User stores the token as a secret in a separate terminal
> - Claude validates the secret exists
>
> **Phase 3: Generate Configuration**
> - Claude generates the config file using all collected inputs
> - Shows summary of what was generated
>
> **Phase 4: Validate + Next Steps**
> - Claude runs validation command
> - Success message with links to docs for advanced configuration
>
> **Slash command name:** `/setup`
>
> **Want me to adjust anything? Add or remove phases? Change the order? Rename the command?**

Wait for the user's feedback. They may want to:

- Reorder phases
- Add or remove steps
- Change what's automated vs. manual
- Adjust the level of detail in certain sections
- Rename the slash command
- Change the target audience or tone

Iterate on the outline until the user approves it. Only then move to file generation.

---

## Phase 3: Generate the Tutorial Files

Once the outline is approved, generate the files. The tutorial lives inside the target repo under `.claude/`.

### Files to create

#### 1. Slash command launcher: `.claude/commands/<name>.md`

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

The `description` field appears in Claude Code's command list, so make it clear and user-facing.

If the tutorial needs tools beyond the defaults (Read, Write, Bash, Glob, Grep), add an `allowed-tools` list in the frontmatter.

#### 2. Behavioral rules: `.claude/<name>-instructions.md`

This file tells Claude **how to behave** during the tutorial. It is read once at the start and governs the entire session. Include these rules:

- **No fourth-wall breaking.** Never mention scripts, instruction files, or internal mechanics.
- **Follow the script.** Execute each step in order. Don't skip or improvise unless the user asks.
- **Respect STOP markers.** Pause and wait for user input. Never continue past a STOP without a response.
- **Handle USER markers.** Accept reasonable variations of expected input.
- **Execute ACTION markers.** Run commands or file operations, then report the result.
- **Process conditional blocks.** `[Bracketed text]` is conditional — only execute if the condition is met.
- **No filler prompts.** Never add "Ready?", "Shall we begin?", "Let's go!" — just proceed.
- **Security.** Never accept secrets in conversation. Always direct users to a separate terminal.
- **Numbered lists for choices.** Users reply with numbers, not sentences.
- **Provide direct URLs** for any external action (token creation pages, OAuth setup, etc.).
- **Track all collected input** — it feeds into file/config generation later.
- **Define success criteria** — what "done" looks like at the end.

#### 3. Tutorial script: `.claude/<name>-script.md`

The actual content of the tutorial, using the marker system:

| Marker | Meaning |
|---|---|
| `STOP:` | Pause and wait for user input. Include what you're waiting for. |
| `USER:` | Describes the expected user response. |
| `ACTION:` | Claude executes this (command, file write, etc.). |
| `[Condition]` | Conditional block — only if the condition is true. |
| `---` | Phase or section break. |

Write the script following the approved outline from Checkpoint 2. For each phase:

- Be specific — exact URLs, exact commands, exact scopes/permissions.
- Use code blocks for every command the user needs to run.
- Include examples when asking for input (e.g., "Enter your repo, like `org/repo-name`").
- Validate after each critical step.
- End with next steps and links to further documentation.
- Add an "Important Notes for Claude" section at the end for edge cases and fallback behaviors.
- End with a "Success Criteria" checklist of verifiable outcomes.

#### 4. Project context: `CLAUDE.md` (root)

If the repo doesn't already have a `CLAUDE.md`, create one with a project description, key file paths, and available slash commands. If one exists, add the new command to it.

### CHECKPOINT 3: Review generated files

After generating the files, present a summary to the user:

> **Here's what I created:**
>
> - `.claude/commands/<name>.md` — Slash command launcher
> - `.claude/<name>-instructions.md` — Behavioral rules ([brief summary of key rules])
> - `.claude/<name>-script.md` — Tutorial script with [N] phases:
>   1. [Phase 1 summary]
>   2. [Phase 2 summary]
>   3. ...
> - `CLAUDE.md` — [Created / Updated] with project context
>
> **Want me to walk through any section in detail, or adjust anything before we update the README?**

Let the user review. They may want to read specific files, adjust wording, change the flow, or tweak the tone. Make edits as requested.

---

## Phase 4: Update the README

The tutorial only works if people know it exists.

### What to add

Add a section near the top of the README (before any manual setup instructions):

```markdown
## Quick Start with Claude Code

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed.

```bash
git clone <repo-url> && cd <repo-name> && claude /<command>
```

That's it — the wizard walks you through the rest.
```

Key rules:

- **One command.** Clone, cd, and launch the wizard in a single copy-paste line.
- **Put it before manual instructions.** The guided path should be the default. Manual steps become the fallback.
- **Keep it short.** The README section is a signpost — the wizard handles the detail.

If manual instructions already exist, don't remove them. Rename the section to "Manual Setup" and place it after the guided setup section.

### CHECKPOINT 4: Confirm README changes

Show the user the README diff or the new section before committing it. Ask:

> **Here's the Quick Start section I'd add to the README. Look good?**

Wait for approval before writing.

---

## Phase 5: Final Validation

After all files are created and the README is updated:

1. **Read through the complete flow** end to end. Check that every STOP has a matching USER, every ACTION is achievable, and every conditional block closes properly.
2. **Check that no secrets are handled in conversation.** Every credential step should instruct the user to use a separate terminal.
3. **Verify all URLs are correct and point to the right pages.**
4. **Ensure the config generation logic accounts for all combinations** of user choices.
5. **Confirm the success criteria are testable** — each checkbox should be verifiable with a command or file check.
6. **Verify the README** has the one-line quick start command with the correct repo URL and slash command name.

### CHECKPOINT 5: Final sign-off

> **Everything looks good on my end. Here's a summary of what was created:**
>
> - [List of all files created/modified]
> - Slash command: `/<name>`
> - Quick start: `git clone <url> && cd <repo> && claude /<name>`
>
> **Want to test it, or are we good?**

---

## Key Lessons and Anti-patterns

### Do

- Present findings and get confirmation before designing.
- Propose an outline and get approval before generating.
- Show generated files and get sign-off before updating the README.
- Jump straight into the first meaningful question (in the final tutorial).
- Use numbered lists for choices — users type a number instead of a sentence.
- Provide the exact URL, exact command, exact path — no ambiguity.
- Design for the happy path first, handle edge cases in "Important Notes for Claude."
- Keep the slash command launcher tiny. All logic lives in the instructions and script files.
- Separate behavioral rules from content. The instructions file is reusable; the script is specific.
- Make the README's quick start a single `git clone ... && cd ... && claude /<command>` line.

### Don't

- Don't generate files without the user reviewing the outline first.
- Don't assume you understand the repo perfectly — always present findings and ask.
- Don't skip checkpoints even if you're confident. The user is the author; their intent matters.
- Don't accept secrets in conversation, ever, for any reason.
- Don't use interactive terminal UI features that may not render in all environments. Stick to plain text prompts.
- Don't make Claude announce what files it's reading or what internal steps it's taking (in the final tutorial).
- Don't put all content in a single file. The three-file split (launcher, rules, script) keeps things maintainable.
- Don't forget to track user inputs throughout — they feed into the final generation step.
- Don't include validation steps that require services or APIs that might not be set up yet.
