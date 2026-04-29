# Atlassian CLI

Unified command-line tool for Atlassian Cloud — manage Jira, Confluence, and Bitbucket from your terminal.

## What is this?

A single binary that lets you interact with your Atlassian tools without opening a browser.
No Python, no runtime, no dependencies — just download and run.

## Features

- **150+ commands** across Jira, Confluence, and Bitbucket — **1325+ tests**
- **Cross-service workflows** — `dashboard`, `link-page`, `pr-transition`, `sprint-report`, `release-notes`
- **Named profiles** — switch between multiple sites or accounts with `--profile`
- **OAuth 2.0 (3LO)** — browser-based login with secure PKCE flow and automatic token refresh
- **Secure credential storage** — tokens stored securely in your OS keyring (macOS Keychain, Windows Credential Manager, Linux Secret Service) when available; on headless systems (CI/CD, Docker, WSL2), credentials are stored in a local file with restricted permissions
- **Shell completion** — press Tab to auto-complete commands, options, profile names, project/space keys
- **Interactive REPL** — run `atlassian-cli` with no arguments for a persistent interactive prompt
- **Batch PR monitoring** — check multiple pull requests at once
- **JSON + human output** — `--json` for scripting, readable tables by default
- **JMESPath filtering** — `--filter` to extract exactly the fields you need
- **Cross-platform** — Linux, macOS, Windows (native and WSL2)
- **License tiers** — Free trial (14 days, all features), Solo ($9/mo), Team ($15/mo per seat)

## Quick Start

### 1. Install

**Binary (recommended — no Python required):**

Download the latest binary for your platform from the [Releases page](https://github.com/home-dev-lab/atlassian-cli/releases), then make it executable:

```bash
# Linux / macOS
chmod +x atlassian-cli
./atlassian-cli --version

# macOS (Homebrew) — coming soon
brew install atlassian-cli
```

On Windows, download `atlassian-cli.exe` and run it from PowerShell or Command Prompt.

**Python users (pip):**

```bash
pip install atlassian-cli
```

> Developers who want to contribute or build from source: see [CONTRIBUTING.md](CONTRIBUTING.md).

### 2. Configure (interactive — recommended)

Run the setup wizard. It guides you step by step:

```bash
atlassian-cli auth setup --interactive
```

The wizard asks for:
- A profile name (e.g. `work`)
- Your Jira URL (e.g. `https://mycompany.atlassian.net`)
- Your Atlassian email and API token
- Your Confluence URL (usually auto-derived)
- Your Bitbucket username, app password, and workspace

> **What is an API token?** It is a password substitute you generate in your Atlassian account settings.
> Go to [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens) to create one.

Tokens are stored securely in `~/.atlassian-cli/.env` with restricted file permissions (600).
Your `config.yaml` only contains `$ENV{VAR_NAME}` references — no secrets on disk in plain text.

### 3. Verify

```bash
atlassian-cli auth check
```

### 4. Start using

```bash
# Dashboard: your open issues, recent PRs, and unread activity at a glance
atlassian-cli dashboard

# Jira
atlassian-cli jira projects
atlassian-cli jira search "assignee = currentUser()"
atlassian-cli jira get PROJ-123

# Confluence
atlassian-cli confluence spaces
atlassian-cli confluence search "type = page AND space = DEV"

# Bitbucket
atlassian-cli bitbucket pr list myws/myrepo
atlassian-cli bitbucket pipelines list myws/myrepo
```

## Configuration

### Profiles

A profile is a named set of credentials. You can have several — one per site, account, or environment.

```bash
# Create profiles interactively
atlassian-cli auth setup --interactive --profile work
atlassian-cli auth setup --interactive --profile staging

# List all profiles
atlassian-cli auth profiles

# Change the default profile
atlassian-cli auth switch staging

# Use a specific profile for a single command
atlassian-cli --profile staging jira projects
```

### Config file

`~/.atlassian-cli/config.yaml` stores your profile settings (no secrets):

```yaml
default_profile: work

profiles:
  work:
    jira:
      url: https://mycompany.atlassian.net
      username: user@company.com
      token: $ENV{ATLASSIAN_API_TOKEN}
      auth_method: api_token
    confluence:
      url: https://mycompany.atlassian.net/wiki
      username: user@company.com
      token: $ENV{ATLASSIAN_API_TOKEN}
    bitbucket:
      username: user@company.com
      token: $ENV{BITBUCKET_API_TOKEN}
      workspace: mycompany
```

Actual token values live in `~/.atlassian-cli/.env` (permissions 600):

```bash
ATLASSIAN_API_TOKEN=ATATT3x...
BITBUCKET_API_TOKEN=ATBB...
```

### OAuth 2.0

For organizations that require browser-based login instead of API tokens:

```bash
atlassian-cli auth login --oauth --profile my-oauth
atlassian-cli auth refresh --profile my-oauth
atlassian-cli auth logout --profile my-oauth
```

### OS Keyring (optional — extra security)

> **What is an OS keyring?** Your operating system has a built-in encrypted password manager:
> Keychain on macOS, GNOME Keyring / KWallet on Linux, Windows Credential Manager on Windows.
> Storing tokens there means they are protected by your OS login and never written to disk in plain text.

First, enable keyring support:

```bash
pip install atlassian-cli[keyring]
```

Then manage credentials via keyring:

```bash
# Store credentials for a profile in the OS keyring
atlassian-cli auth keyring store --profile work

# Check whether credentials are in the keyring
atlassian-cli auth keyring status --profile work

# Migrate existing .env credentials into the keyring
atlassian-cli auth keyring migrate --profile work

# Remove credentials from the keyring
atlassian-cli auth keyring remove --profile work
```

Once stored, the CLI uses them automatically — your workflow stays the same.

## Shell Completion

> **What is shell completion?** When you press `Tab` while typing a command, your shell auto-completes
> the rest. With this setup, it works for commands, options, profile names, Confluence space keys,
> and Jira project keys.

### Bash

```bash
# Option A: add to ~/.bashrc (runs at startup)
eval "$(_ATLASSIAN_CLI_COMPLETE=bash_source atlassian-cli)"

# Option B: generate a file (faster startup)
atlassian-cli completion bash > ~/.atlassian-cli-complete.bash
echo 'source ~/.atlassian-cli-complete.bash' >> ~/.bashrc
```

Reload your shell: `source ~/.bashrc`

### Zsh

```bash
# Option A: add to ~/.zshrc
eval "$(_ATLASSIAN_CLI_COMPLETE=zsh_source atlassian-cli)"

# Option B: generate a file
atlassian-cli completion zsh > ~/.atlassian-cli-complete.zsh
echo 'source ~/.atlassian-cli-complete.zsh' >> ~/.zshrc
```

Reload your shell: `source ~/.zshrc`

### Fish

```bash
atlassian-cli completion fish > ~/.config/fish/completions/atlassian-cli.fish
```

After setup, pressing `Tab` completes:
- Commands and sub-commands
- Options (`--profile`, `--space`, `--project-key`, …)
- Profile names from your config
- Space keys and project keys (fetched live from your site)

## Interactive REPL

> **What is a REPL?** A Read-Eval-Print Loop — an interactive prompt where you type commands
> and see results immediately, without repeating `atlassian-cli` each time.

Run the CLI with no arguments to enter interactive mode:

```bash
atlassian-cli
```

```
atlassian> jira get PROJ-<Tab>        # completes issue keys live
atlassian> --profile <Tab>            # lists your profiles
atlassian> confluence spaces
atlassian> exit
```

Full tab completion is built in — the same completions as the shell setup above.

## Commands Overview

| Service | Commands | Highlights |
|---------|----------|------------|
| **Jira** | 55+ | Issues, sprints, boards, transitions, comments, attachments, worklogs, epics, filters, bulk ops, SLA |
| **Confluence** | 43+ | Pages, spaces, comments (inline + footer, with `resolutionStatus` open/reopened/resolved), content properties, labels, attachments, restrictions, export PDF/Word, hybrid inline-comment reanchoring (`--reanchor-map` + `--orphan-strategy=auto`: manual + LCS + footer fallback) |
| **Bitbucket** | 30+ | PRs (+ default reviewers, comments, tasks), batch PR status (multi-repo, conflict detection), pipelines (trigger, rerun), branches, commits, repos, clone |
| **Cross-service** | 5 | dashboard, link-page, pr-transition, sprint-report, release-notes |
| **Auth** | 12 | setup, status, check, profiles, switch, login, refresh, logout, keyring store/remove/status/migrate |

### Cross-Service Workflows

```bash
# My work dashboard — open issues, recent PRs, unread activity
atlassian-cli dashboard

# Create a Confluence page from a Jira issue
atlassian-cli link-page PROJ-123 --space DEV

# Transition a Jira issue when a PR merges
atlassian-cli pr-transition myws/myrepo 42 --status "Done"

# Generate a sprint report as a Confluence page (--dry-run previews without creating)
atlassian-cli sprint-report 42 --space DEV --dry-run
atlassian-cli sprint-report 42 --space DEV

# Generate release notes as a Confluence page
atlassian-cli release-notes --project PROJ --version "1.2.0" --space DEV --dry-run
```

All workflow commands support: `--dry-run` (preview only), `--update` (update if page exists),
`--target-profile` (write to a different site), `--filter` (JMESPath output filtering).

### Batch PR Status

Monitor several pull requests at once:

```bash
# Single repo
atlassian-cli bitbucket pr status myws/myrepo 858,338,667

# Multi-repo (repo:PR pairs)
atlassian-cli bitbucket pr status myws/repo1:94 myws/repo2:95

# From a file (one repo:PR per line)
atlassian-cli bitbucket pr status --from-file prs.txt
```

Each PR report includes:
- Title, state, author, reviewers, and approval status
- `has_conflicts` — merge conflict detection via diffstat
- `merge_blockers` — list of conditions preventing merge (open tasks, missing approvals, failing builds, conflicts)
- `merge_ready` — boolean indicating whether the PR can be merged cleanly

Useful for release checklists or reviewing a stack of dependent PRs.

### PR Comments and Tasks

```bash
# Resolve / unresolve a comment
atlassian-cli bitbucket pr resolve-comment myws/myrepo 42 --comment-id 12345
atlassian-cli bitbucket pr unresolve-comment myws/myrepo 42 --comment-id 12345

# List tasks on a PR
atlassian-cli bitbucket pr tasks myws/myrepo 42

# Create a task
atlassian-cli bitbucket pr create-task myws/myrepo 42 "Fix the typo in README"

# Resolve / unresolve tasks
atlassian-cli bitbucket pr resolve-task myws/myrepo 42 --task-id 99
atlassian-cli bitbucket pr resolve-task myws/myrepo 42 --all
atlassian-cli bitbucket pr unresolve-task myws/myrepo 42 --task-id 99
```

### PR Creation

```bash
# Create a PR with description from a file
atlassian-cli bitbucket pr create myws/myrepo \
  --source feature/xyz --destination main \
  --title "Add feature XYZ" \
  --description-file pr-description.md
```

### Pipelines

```bash
# List recent pipeline runs
atlassian-cli bitbucket pipelines list myws/myrepo

# Show steps for a specific build (use the build number shown in the Bitbucket UI)
atlassian-cli bitbucket pipelines steps 315
atlassian-cli bitbucket pipelines steps "{uuid-...}"

# Trigger a new pipeline run
atlassian-cli bitbucket pipelines trigger myws/myrepo --branch main

# Re-run a failed pipeline
atlassian-cli bitbucket pipelines rerun myws/myrepo 315
```

### Global Options

| Option | Description |
|--------|-------------|
| `--json` | JSON output for scripting or piping to other tools |
| `--profile NAME` | Use a named credential profile |
| `--site NAME` | Override site (e.g. `mycompany` → `mycompany.atlassian.net`) |
| `--verbose` / `-v` | Enable debug logging and upstream response body preview on errors |
| `--version` | Show the installed version |

Use `--help` on any command to see all available options:

```bash
atlassian-cli jira --help
atlassian-cli confluence create --help
atlassian-cli bitbucket pr --help
```

## API Client

The built-in HTTP client (`AtlassianClient`) handles authentication, retries, pagination, and security:

- **SSRF-safe redirect handling** — 302 redirects are followed only when the target is a trusted Atlassian domain (`*.atlassian.net`, `*.atlassian.com`, `*.bitbucket.org`). Redirects to unknown hosts are blocked.
- Automatic retry with exponential backoff for transient errors
- Cursor-based and offset-based pagination
- Per-host SSRF validation on all outgoing requests

## Platform Support

| Platform | Status |
|----------|--------|
| Linux | Supported |
| macOS | Supported |
| Windows (WSL2) | Tested and supported |
| Windows (native) | Supported |

## License

Proprietary. See [LICENSE](LICENSE).
