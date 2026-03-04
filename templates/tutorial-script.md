# <Tutorial Title>

## Phase 1: Welcome and Configuration Choices

<Brief welcome. One or two sentences on what this tutorial will set up.>

<Present choices as a numbered list. Example:>

**Which features would you like to set up?** *(enter numbers, e.g. "1" or "1, 3")*

1. **Option A** — Short description *(Recommended)*
2. **Option B** — Short description
3. **Option C** — Short description

STOP: Wait for the user to reply with their selection.

USER: Enters one or more numbers.

ACTION: Record the user's selections for later use.

---

## Phase 2: Prerequisites and Credentials

[If user selected Option A]

### Option A Setup

You'll need a credential for Option A. Here's how to create one:

1. Go to **<exact location>**: <direct URL>
2. Click **<exact button/link>**
3. Configure with these settings:
   - <Setting 1>: <Value or recommendation>
   - <Setting 2>: <Value or recommendation>
4. Copy the generated value

Now store it securely. Run this in a **separate terminal** (not here):

```
<secret-storage-command with placeholder>
```

**Do not paste the value here — always use a separate terminal for secrets.**

STOP: Wait for user to confirm the secret is stored.

USER: Confirms secret is stored.

ACTION: <Validate the secret exists or the integration is accessible.>

[End Option A]

---

## Phase 3: Generate Configuration

ACTION: Using collected inputs, generate the configuration file(s).

Write the config to `<path>` with the user's selections:
- <Setting from Phase 1>
- <Credential reference from Phase 2>
- <Any defaults>

Tell the user what was generated and where it was saved.

---

## Phase 4: Validate and Finish

ACTION: Run validation command(s) to confirm everything works.

<If validation passes, show success message:>

**Setup complete!** Here's what was configured:

- <Summary item 1>
- <Summary item 2>
- <Summary item 3>

**Next steps:**
- <What to do now>
- <Link to documentation for advanced configuration>
- <How to add more integrations later>

---

## Important Notes for Claude

- If the user wants to skip a phase, let them — note what was skipped in the final summary.
- If validation fails, show the error and walk through common fixes before giving up.
- <Add edge cases specific to this tutorial.>

## Success Criteria

- [ ] User made their configuration choices
- [ ] All selected credentials are stored securely
- [ ] Configuration file(s) generated with correct values
- [ ] Validation passed
