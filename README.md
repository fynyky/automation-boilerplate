# AI Automated Developer Template
This is a blank starter project with basic AI tooling setup. Clone this repo and follow the instructions to get started with AI development. This includes:
- Using autonomous agents safely through remote dev environments
- Running multiple agents simultaneously
- Triggering agents to handle tasks
- Reviewing code automatically
- Periodic testing and codebase maintenance

The goal is to close the loop with AI as much as possible to speed development, while still allowing human intervention to steer the project when needed.

This template is intentionally generic to be a starting point for any project. It provides the basic working tools to get you going, but as your project develops you'll want to customize it to get the most value.

---

# What it does

The core idea is to use GitHub as a coordination backbone for AIs to work off of. Instead of a single super agent, you can have smaller agents pick up and drop off tasks on a repo, just like a team of people would. This makes reasoning about the automation much simpler, provides clear intervention points, and makes it easy to adjust.

Off this backbone we need the following capabilities to have a full development loop:
- Isolated environments to pair program with unrestricted AI safely
- Spawn autonomous agents to go on long running tasks and return with PRs
- Automatically spawn agents from issues
- Automatically review PRs and merge trivial ones
- Automatically create issues from errors and feedback
- Periodically test the codebase for problems and create issues accordingly



---
 # Using it 


## How it works

This template sets up a layer of automation on top of GitHub using Claude. Once configured, the following happens automatically:

```
User/Sentry/Tests report a problem
        ↓
Issue is created in GitHub
        ↓
Analyzer agent triages the issue, adds context, tags implementer
        ↓
Implementer agent creates a branch and opens a PR
        ↓
Reviewer agent reviews the PR and requests changes or approves
        ↓
PR is merged into main (branch protection enforces checks pass first)
        ↓
Weekly: Security agent scans the repo and opens issues for findings
```

All agents are powered by Claude via the Claude GitHub App and GitHub Actions workflows. Agent behavior is controlled by `CLAUDE.md` at the root of the repo.

---

## What's included

```
.
├── .devcontainer/
│   └── devcontainer.json       # Codespaces config with Claude tooling pre-installed
├── .github/
│   └── workflows/
│       ├── issue-analyzer.yml  # Triages new issues, adds context, tags implementer
│       ├── issue-implementer.yml # Picks up tagged issues, opens a fix PR
│       ├── pr-reviewer.yml     # Reviews incoming PRs and leaves feedback
│       └── weekly-scan.yml     # Runs weekly, opens issues for security findings
├── .vscode/
│   └── settings.json           # Recommended VSCode settings for Codespaces
├── CLAUDE.md                   # Agent instructions and constraints — customize this
├── Makefile                    # Standard targets your project must implement (see below)
└── README.md
```

---

## Prerequisites

- A GitHub account with Actions enabled
- A [Claude GitHub App](https://github.com/apps/claude) installation
- An Anthropic API key

---

## Setup

### 1. Create your repo from this template

Click **Use this template** at the top of this repo, or clone it and push to a new repo.

### 2. Install the Claude GitHub App

Go to [github.com/apps/claude](https://github.com/apps/claude) and install it on your new repository.

### 3. Add your API key as a secret

In your repo, go to **Settings → Secrets and variables → Actions** and add:

| Secret | Value |
|---|---|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |

### 4. Configure branch protection

Go to **Settings → Branches** and add a rule for `main`:

- ✅ Require a pull request before merging
- ✅ Require status checks to pass before merging
- ✅ Require approvals: `1`
- ✅ Dismiss stale pull request approvals when new commits are pushed
- ✅ Do not allow bypassing the above settings

This ensures agents can never directly push to main.

### 5. Implement the Makefile targets

The workflows invoke standard Makefile targets so they stay stack-agnostic. Implement these in your project's `Makefile`:

```makefile
# Required
test:       # Run the full test suite
lint:       # Run linting / static analysis
build:      # Build the project (if applicable)

# Optional but recommended
security:   # Run a security scan (e.g. npm audit, bandit, trivy)
```

If a target isn't relevant for your project, have it exit 0 silently.

### 6. Customize CLAUDE.md

Open `CLAUDE.md` and update the sections marked `TODO`. At minimum, set:

- What this project does (agents need context to make good decisions)
- What's in and out of scope for autonomous changes
- Any directories or files agents should never touch
- Your preferred branch naming convention

### 7. Enable the weekly scan

Go to **Actions → weekly-scan** and enable the workflow. It's disabled by default so it doesn't run until you're ready.

---

## Using Codespaces

This template includes a dev container so you can work in a fully configured cloud environment with Claude available via the CLI.

1. Click **Code → Codespaces → Create codespace on main**
2. Once it loads, Claude Code is available in the terminal: `claude`
3. Your `ANTHROPIC_API_KEY` secret is automatically available if added to Codespaces secrets under **Settings → Codespaces**

---

## Agent reference

### Issue Analyzer (`issue-analyzer.yml`)
Triggers on new issues. Claude reads the issue, researches relevant code, and posts a comment with: a summary of the problem, relevant files, and a suggested approach. It then labels the issue `agent-ready` to signal the implementer.

### Issue Implementer (`issue-implementer.yml`)
Triggers when an issue is labeled `agent-ready`. Claude creates a branch, implements the fix, runs `make test` and `make lint`, and opens a PR linked to the issue.

### PR Reviewer (`pr-reviewer.yml`)
Triggers on new pull requests. Claude reviews the diff, checks against `CLAUDE.md` constraints, runs any checks it can, and either requests changes or approves. It does not merge — a human approval or a second passing review is required by branch protection.

### Weekly Scanner (`weekly-scan.yml`)
Runs every Monday at 09:00 UTC. Claude runs `make security` (if defined), reviews dependencies for known issues, and opens labeled issues for anything it finds. Findings are deduplicated so re-runs don't create duplicate issues.

---

## Customization

**To change agent behavior** — edit `CLAUDE.md`. This is the primary control surface. Be explicit: vague instructions produce inconsistent behavior.

**To restrict what agents can touch** — add paths to the `off-limits` section of `CLAUDE.md`. Agents will not open PRs that modify those paths.

**To add a new agent** — copy an existing workflow file and update the trigger, prompt, and step logic. Follow the same pattern of running `make test` before opening any PR.

**To connect an error tracker** — add a workflow that listens to your tracker's webhook and creates a GitHub issue in a standard format. The issue analyzer will pick it up from there.

---

## Security notes

- Agents operate with the permissions of the GitHub App installation — review what access you grant
- No agent can push directly to `main`; branch protection enforces this independently of agent instructions
- The `ANTHROPIC_API_KEY` secret is only accessible to workflow runs on your repo — it is not exposed to PRs from forks
- Review agent-opened PRs before merging; treat them the same as you would a junior developer's output

---

## Contributing

Issues and PRs welcome. For significant changes, open an issue first to discuss.
