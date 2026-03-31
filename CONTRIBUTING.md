# Contributing to sf-agent-toolkit

This toolkit improves through real-world usage. Contributions from lessons learned on actual Salesforce projects are the most valuable kind. Whether you're fixing a bug, improving an existing agent, or adding a new one, your contribution helps every developer using Claude Code to build Salesforce features faster.

## How to Report a Bug

Include all of the following in your issue:

- **The relevant log entry** from `.claude/session-notes/<branch-slug>.log.md` showing the failure
- **Claude Code version:** Run `claude --version` to confirm
- **Org type:** Scratch org, sandbox, developer edition, or production
- **Which agent failed** and at what point in the pipeline (e.g., "sf-implementer, during deploy")
- **The exact error message** (full stack trace if available)
- **Steps to reproduce:** What task triggered the failure? What were the inputs?

**Example issue title:** `sf-pr-raiser fails with 404 when repo has capital letters in name`

## How to Improve an Existing Agent

Follow this workflow:

### Step 1: Make the change in your project

Edit `.claude/agents/<agent>.md` (or `.claude/commands/<command>.md`) and test it on a **real org with a real pipeline run**.

Examples of good improvements:
- Clearer role description in "Responsibilities"
- New Hard Rules based on a production failure you hit
- Expanded Input Schema with new optional parameters
- Better error handling or fallback logic in workflows

### Step 2: Confirm the change is generic

Before pushing to the toolkit, scrub the file for any project-specific values:

```bash
grep -E "your-org|your-project|your-company|specific-record-id|specific-field-name|internal-system" \
  .claude/agents/<agent>.md \
  .claude/commands/<command>.md
```

**Zero matches required.** No exceptions. Project-specific names must be replaced with placeholders or examples.

### Step 3: Submit a PR

Include:
- **Which agent was changed** (e.g., `sf-implementer`)
- **What problem it solves** (e.g., "Handles 404 errors from GitHub MCP with gh CLI fallback")
- **Before/after log entry** from a real run showing the problem and the fix
- **Link to the change** in your project's git history (if public)

**Example PR title:** `fix(sf-pr-raiser): derive GitHub owner/repo from git remote before MCP call`

**Example PR body:**
```markdown
## Problem
sf-pr-raiser was hardcoding GitHub owner/repo values from context.
This caused failures on projects where the repo name didn't match expectations.

## Solution
Added Step 0 pre-check that:
1. Runs `git remote -v` to extract origin URL
2. Parses HTTPS or SSH format
3. Derives owner and repo values
4. Reports blocked if origin is not a GitHub URL
5. Falls back to `gh` CLI if MCP fails with 404/422

## Evidence
Before fix: PR creation failed with MCP 404
After fix: PR created successfully via derived owner/repo
Log entry: https://github.com/<your-project>/commit/<sha>
```

## How to Add a New Agent

New agents must be **high-value, reusable across all Salesforce projects, and non-trivial.**

Before starting, ask yourself:
- Does this fit within the existing pipeline flow?
- Would most Salesforce teams benefit from this?
- Is it more than 100 lines of documentation?
- Does it require a new MCP or external tool?

### Requirements Every New Agent Must Have

#### 1. Input Schema
```json
{
  "org_alias": "from sf-agent.config.json — NEVER hardcode",
  "branch_name": "feature/my-change",
  "feature_description": "Plain language description"
}
```

#### 2. Output Schema
```json
{
  "status": "complete | blocked | partial",
  "blockers": [],
  "notes": "What happened during this run",
  "checkpoint_write": "orchestrator writes stages.<agent> = <status>"
}
```

#### 3. Hard Rules Section
At least 3-5 non-negotiable rules for safe operation.

#### 4. Log Update Rule
How this agent updates the run log — must be documented in `sf-change.md` Mandatory Write Rules.

#### 5. Bypass Registration
Documented in `.claude/commands/sf-change.md`:
- Is it bypassable? (yes/no)
- Can both QA agents be skipped? (if applicable)
- What happens if it's bypassed?

#### 6. State File Integration
New checkpoint stage key in `sf-change.md` State File Format section.

#### 7. Zero Project-Specific Values
Scrub before pushing:
```bash
grep -rE "your-org|your-project|your-company|internal-system" .claude/agents/
```

### Implementation Steps

1. **Create the agent file** modelled on an existing agent:
   ```bash
   cp .claude/agents/sf-implementer.md .claude/agents/sf-my-new-agent.md
   ```
   Update role, responsibilities, input/output schemas, hard rules.

2. **Add to the pipeline diagram** in `.claude/commands/sf-change.md`

3. **Add to orchestrator decision logic** tables in `sf-change.md` (when to invoke, what to do with output)

4. **Add to state file format** in `sf-change.md` Checkpoint System section

5. **Add to bypassable vs required table** in README.md

6. **Test end-to-end** on a real org with a real task

7. **Scrub for project-specific values** — zero matches on the grep above

8. **Submit a PR** with:
   - The new agent file
   - Updated `sf-change.md`
   - Updated `README.md`
   - Real log entry showing the agent running successfully

## How to Test Changes

Before submitting **any** contribution:

### 1. Run /sf-change with your changes

Use a real task on a real org. Don't test with dummy data.

```
/sf-change

Task: [real task you're working on]
...
qa_method: playwright | postman
```

### 2. Confirm the log entry

The toolkit is production-ready only if the log shows:
- All expected stages completed (or correctly bypassed)
- QA evidence attached (screenshots, test output)
- Cost tracking recorded
- PR successfully raised (if applicable)

### 3. Test bypass scenarios

If your change touches an optional agent, test that agent being bypassed:

```
/sf-change
...
bypass: ["sf-architect"]  # if you modified architect
```

### 4. Include the log in your PR

Attach the relevant `.log.md` entry as evidence that the change works.

## Scrub Requirement — No Exceptions

Every contribution must pass this check before pushing to the toolkit:

```bash
grep -rE "your-org|your-project|your-company|specific-record-id\
|specific-field-name|internal-system|localhost|127.0.0.1" \
  .claude/agents/ \
  .claude/commands/ \
  .claude/sf-agent.config.template.json
```

**Zero matches required.** If you find any, replace with:
- Generic placeholders: `<org_alias>`, `<owner>`, `<repo>`
- Example values: `"my-org-alias"`, `"salesforce-developer"`
- Documentation: Describe what should go there in the relevant file

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/) with scope:

```
<type>(<scope>): <description>

<body>

Co-Authored-By: [Your Name] <your-email@example.com>
```

**Scopes:**
- `sf-architect`
- `sf-implementer`
- `sf-qa-playwright`
- `sf-qa-postman`
- `sf-doc-writer`
- `sf-pr-raiser`
- `sf-change`
- `sf-setup`
- `toolkit` (docs, general improvements)

**Examples:**
- `fix(sf-pr-raiser): derive GitHub owner/repo from git remote`
- `feat(sf-implementer): add skip_apex_tests flag for fast iteration`
- `docs(toolkit): add cost tracking to README`

## Before You Push

1. ✅ Your change works on a real org (log entry proof)
2. ✅ Scrub passes (zero project-specific values)
3. ✅ Bypass scenarios tested (if applicable)
4. ✅ Hard rules are clear and non-negotiable
5. ✅ Checkpoint integration documented
6. ✅ Commit message follows Conventional Commits

## Code of Conduct

We're building a toolkit for the global Salesforce developer community. Keep contributions:
- **Respectful:** Assume good intent in all discussions
- **Professional:** Titles and descriptions should be clear and accurate
- **Inclusive:** Examples should represent diverse Salesforce use cases and team structures

---

Thank you for contributing to sf-agent-toolkit. Your improvements help thousands of developers ship faster. 🚀
