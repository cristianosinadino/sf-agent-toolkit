# sf-agent-toolkit

Automate your complete Salesforce feature development cycle with Claude Code. From architectural design through implementation, QA, documentation, and GitHub PR creation — accomplish everything with a single `/sf-change` command. This toolkit orchestrates a team of specialized AI agents that follow industry best practices for Salesforce development, eliminating manual cycle steps and keeping your team in Claude Code instead of clicking through Salesforce and GitHub UIs.

## Prerequisites

| Tool | Required | Notes |
|------|----------|-------|
| SF CLI | ✅ Required | `sf --version` to verify. [Install](https://developer.salesforce.com/tools/salesforcecli) |
| Claude Code | ✅ Required | v2.1+. [Download](https://claude.com/download) |
| MCP: context7 | ✅ Required | Fetch current Salesforce API docs during implementation |
| MCP: playwright | ✅ Required | Browser automation for UI QA (`sf-qa-playwright`) |
| MCP: github | ✅ Required | GitHub PR creation (`sf-pr-raiser`) |
| MCP: sequential-thinking | ✅ Required | Decompose requirements before architecture/implementation |
| newman | ⚡ Optional | Required only if using `sf-qa-postman` for API testing |
| gh CLI | ⚡ Recommended | Fallback for PR creation if GitHub MCP encounters issues |

## Getting Started

### 1. Clone toolkit into your project

```bash
git clone https://github.com/cristianosinadino/sf-agent-toolkit .claude-toolkit
```

### 2. Copy .claude/ folder into your project

```bash
cp -r .claude-toolkit/.claude ./
rm -rf .claude-toolkit
```

### 3. Open Claude Code

```bash
claude
```

### 4. Run setup — guided configuration from here

```
/sf-setup
```

The setup command will:
- Validate your environment (SF CLI, MCPs, org authentication)
- Configure your org alias, branch strategy, and log location
- Scaffold `.claudeignore`, `CLAUDE.md`, and `.gitignore` entries
- Test all MCP connections

After setup, you're ready to run `/sf-change`.

## The Pipeline

Each `/sf-change` run orchestrates a configurable sequence of specialized agents. Run the full pipeline or bypass optional stages to fit your workflow.

| Agent | Role | Bypassable |
|-------|------|-----------|
| `sequential-thinking` | Decompose requirements into sub-problems; identify unknowns | ❌ No — always runs |
| `sf-architect` | Design review; compare implementation options; write implementer brief | ✅ Yes (auto-skipped for low-complexity changes) |
| `sf-implementer` | LWC + Apex implementation; unit tests; deploy to org | ❌ No — always runs |
| `sf-qa-playwright` | Browser automation; UI flow validation; screenshot evidence | ✅ Yes (if using postman QA) |
| `sf-qa-postman` | Postman/Newman API validation; response assertion evidence | ✅ Yes (if using playwright QA) |
| `sf-doc-writer` | Solution design documentation; architecture walkthrough | ✅ Yes |
| Human Review Gate | Approve changes before PR is raised | ❌ No — mandatory checkpoint |
| `sf-pr-raiser` | GitHub PR creation with title, body, labels, evidence | ✅ Yes |

**Rule:** At least one QA agent must run — both `sf-qa-playwright` and `sf-qa-postman` cannot be bypassed in the same run.

## Running a Change

### Minimal prompt structure

```
/sf-change

Task: [what needs to change and why]

Acceptance Criteria:
- Given [context], when [action], then [outcome]
- Given [edge case], when [action], then [safe fallback]

qa_method: playwright | postman | both
```

The orchestrator will derive `org_alias` and `base_branch` from `.claude/sf-agent.config.json`.

### Full example

```
/sf-change

Task: Add validation to the product selection step

Acceptance Criteria:
- Given product list is empty, when user clicks Next, then show error message
- Given user selects a product, then enable Next button and store selection
- Given user navigates back, then preserve product selection
- Given user can scroll through 100+ products, then validation performance is < 100ms

qa_method: playwright
skip_apex_tests: false
bypass: []
```

## Key Behaviours

- **Checkpoint system:** Interrupted runs resume from the last completed stage. Perfect for context window limits or network issues. State file tracks exact stage and run metadata.
- **Human review gate:** Even with QA passing, a PR is never raised without your explicit approval. Review diff, QA results, and draft PR before proceeding.
- **Context modes:** Full context for `sf-architect` to see data model and API docs; surgical context for all other agents to stay focused on scoped changes.
- **Mandatory run logs:** Every execution logged to `<log_path>/<branch-slug>.log.md` with stage results, QA evidence, cost tracking, and git PR link.
- **Agent bypass:** Skip any optional agent per run — e.g., `bypass: ["sf-architect"]` to skip design review on obvious fixes.
- **Cost tracking:** Estimated API cost per agent and total run cost recorded in logs and solution design docs. No surprises on billing.

## QA Methods

Choose the right QA strategy for your change:

| qa_method | When to use | Evidence attached |
|-----------|-------------|-------------------|
| `playwright` | UI wizard steps, LWC component behaviour, field validation | Screenshots, user journey video |
| `postman` | Apex REST endpoints, API contracts, integration response payloads | Newman test results, curl output |
| `both` | Features with both UI surface and backing API | All of the above |

**Postman setup:** See [postman/README.md](postman/README.md) for collection export and environment configuration.

## Optional Features

### Skip Apex tests for fast iteration

```
/sf-change
...
skip_apex_tests: true
```

Set to `true` only for narrow, confident patches where you accept the risk. Tests are skipped, but the warning is always documented in the run log. Default: `false` (always run tests).

## Typical Run Costs

Estimates based on claude-sonnet-4-6 pricing ($3/1M input, $15/1M output):
- **Surgical patch (no architect, postman QA):** ~$0.30–$0.50
- **Small feature (architect + playwright):** ~$0.40–$0.70
- **Complex feature (full pipeline, both QA):** ~$0.80–$1.50

Actual costs visible at [console.anthropic.com](https://console.anthropic.com).

## Links

- **[Toolkit Architecture](docs/how-it-works.md)** — Deep dive into agent design, decision logic, and examples
- **[Contributing](CONTRIBUTING.md)** — Report bugs, improve agents, add new agents
- **[Postman Setup](postman/README.md)** — API testing configuration and best practices
- **[CLAUDE.md in your project](CLAUDE.md)** — Project-specific rules, domain knowledge, and CI/CD config

## Support

- **FAQ & troubleshooting:** See [docs/how-it-works.md](docs/how-it-works.md)
- **Report an issue:** [GitHub Issues](https://github.com/cristianosinadino/sf-agent-toolkit/issues)
- **Contribute improvements:** [Contributing](CONTRIBUTING.md)

---

**Made with ❤️ for Salesforce developers using Claude Code.**
