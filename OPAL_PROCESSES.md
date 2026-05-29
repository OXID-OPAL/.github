# OPAL Processes — Compiled from Serena Memories

This document describes the standard processes for developing, releasing, and managing OPAL modules for OXID eShop. All examples assume a Linux environment.

## Conventions Used in This Document

| Placeholder | Meaning |
|-------------|---------|
| `<project-root>` | Host path to the OXID Docker project (contains `docker-compose.yml`) |
| `<docroot>` | Host path to the OXID shop docroot (e.g. `<project-root>/docroot`) |
| `<shop-root>` | Shop root **inside** the Docker container (typically `/var/www/html`) |
| `<web-container>` | Name of the Docker web/PHP container (e.g. `web`) |
| `<module-id>` | OXID module ID, pattern: `opal<featurename>` (lowercase, no separators) |
| `<your-email>` | Your Jira account email address |
| `OPALE-XX` | Jira ticket ID in the OPALE project |

---

## 0. Tool Installation

### Docker & Docker Compose

Docker is required to run the OXID eShop environment. All PHP commands (quality tools, oe-console, composer, tests) run inside the Docker container.

```bash
# Install Docker Engine (official method)
# See: https://docs.docker.com/engine/install/

# Docker Compose is included with Docker Engine as a plugin (v2+)
# Verify:
docker compose version
```

### GitHub CLI (`gh`)

Used for creating repositories, setting permissions, and managing PRs in the OXID-OPAL GitHub org.

```bash
# Install via package manager
# Debian/Ubuntu:
sudo apt install gh

# Or via official repo:
# See: https://github.com/cli/cli/blob/trunk/docs/install_linux.md

# Authenticate:
gh auth login
```

### Jira CLI (`jira-cli`)

Used for ticket management (assign, transition, comment, worklog). Installed via snap on Ubuntu/Debian.

```bash
# Install via snap
sudo snap install jira-cli

# Configure (first run — interactive):
jira-cli init

# The binary is at /snap/bin/jira-cli
# IMPORTANT: Always invoke as /snap/bin/jira-cli or snap run jira-cli (not just "jira")
```

If installed via a different method (e.g. `go install`), adjust the binary path accordingly but keep all command arguments the same.

### Git

```bash
# Debian/Ubuntu:
sudo apt install git
```

### curl

Used for Jira REST API workarounds where jira-cli has bugs.

```bash
# Debian/Ubuntu (usually pre-installed):
sudo apt install curl
```

### Claude Code

AI coding assistant used for module development. Token usage is tracked per module session.

```bash
# Install via npm:
npm install -g @anthropic-ai/claude-code

# See: https://docs.anthropic.com/en/docs/claude-code
```

---

## 1. New Module Preparation

When starting work on a new OPAL module from the OPALE Jira board:

1. **Pick a ticket** from "OPAL Idee" status
2. **Create module folder:**
   ```bash
   mkdir -p <docroot>/vendor/opal/<module-id>/assets
   ```
   Module ID pattern: `opal<featurename>` (lowercase, no separators)
3. **Create `SCOPE.md`** — copy the ticket's full description (requirements, acceptance criteria, comments) into `SCOPE.md` in the module folder. This serves as the working specification.
4. **Copy LICENSE and logo** from any existing opal module:
   ```bash
   cp <docroot>/vendor/opal/<existing-module>/LICENSE <docroot>/vendor/opal/<module-id>/LICENSE
   cp <docroot>/vendor/opal/<existing-module>/assets/logo.png <docroot>/vendor/opal/<module-id>/assets/logo.png
   ```
5. **Assign ticket:**
   ```bash
   /snap/bin/jira-cli issue assign OPALE-XX "<your-email>"
   ```
6. **Move to In Progress:**
   ```bash
   /snap/bin/jira-cli issue move OPALE-XX "In Progress"
   ```

**Important:** Run jira-cli commands one at a time — they hang when run concurrently.

---

## 2. Module Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Module Title | `Opal <Descriptive Title>` | Opal Stock Management |
| Module ID | `opal<moduletitle>` | `opalstockmanagement` |
| Composer package | `opal/opal<moduletitle>` | `opal/opalstockmanagement` |
| Namespace | `Opal\Opal<ModuleName>` | `Opal\OpalStockManagement` |
| Branch | `b-<major>.<minor>.x` | `b-7.4.x` |
| DB table prefix | `<module_id>_` | `opalstockmanagement_levels` |
| Company | `OXID eSales AG` | |
| License | OXID Modul & Komponenten-Lizenz 2023 (proprietary) | |

Branch uses a **hyphen** (`b-7.4.x`), NOT a dot. Use `b-<major>.x` only if compatible across all minors within the major version.

### Testing Handoff

After development is complete, Support will test OPALs using **Playwright** tests. For this to work, the module **must include a README with installation instructions**. The README is the basis for Support's test setup — without it, testing cannot proceed.

---

## 3. Jira Interaction

### 3.1 Status Transitions (OPALE project)

The OPALE project (next-gen/team-managed) has mismatched transition vs. status names. The `jira-cli` shows **transition names**, not target status names:

| jira-cli Transition Name | Actual Target Status | Category |
|--------------------------|---------------------|----------|
| `To Do` | OPAL Idee | To Do (blue-gray) |
| `In Progress` | In Arbeit | In Progress (yellow) |
| `Done` | Fertig | Done (green) |
| **`Review it`** | **Support-Loop** | **To Do (blue-gray)** |
| `Support-Ready` | Support-Ready | To Do (blue-gray) |
| `Abgelehnt` | Abgelehnt | Done (green) |

All transitions are global (available from any status).

### 3.2 Priority Editing (Workaround)

`jira-cli issue edit --priority` is **broken** (always returns 400). Use the Jira REST API directly:

```bash
# Extract auth header from jira-cli's debug output
AUTH_HEADER=$(snap run jira-cli issue view OPALE-XX --raw --debug 2>&1 | grep -oP 'Authorization: Basic \K\S+' | head -1)

# Set priority via REST API (returns HTTP 204 on success)
curl -s -X PUT \
  -H "Authorization: Basic $AUTH_HEADER" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  "https://oxid-esales.atlassian.net/rest/api/3/issue/OPALE-XX" \
  -d '{"fields":{"priority":{"id":"PRIORITY_ID"}}}'
```

**Priority Name → ID Map:**

| Name | ID |
|------|-----|
| Blocker | 1 |
| Muss | 10000 |
| Kritisch | 2 |
| Wichtig | 3 |
| Soll | 10001 |
| Geringfügig | 4 |
| Kann | 10002 |
| Trivial | 5 |
| Zurückgestellt | 10003 |

### 3.3 Unassigning a Ticket

```bash
/snap/bin/jira-cli issue assign OPALE-XX x
```

### 3.4 jira-cli Rules & Pitfalls

- Always use `/snap/bin/jira-cli` (not `jira`) — the bare command may resolve to a different tool
- Do NOT use undocumented parameters — `--no-input` does not exist and silently fails
- `issue assign` requires the **full email address** as the second argument, not `$(whoami)`
- Run commands **one at a time** (never in parallel — they hang)
- If a command requires interactive input and fails with EOF, fix the arguments instead of inventing flags

### 3.5 Verifying Ticket State

```bash
/snap/bin/jira-cli issue view OPALE-XX --raw 2>&1 | \
  python3 -c "import sys,json; d=json.load(sys.stdin); print('Status:', d['fields']['status']['name']); print('Assignee:', d['fields']['assignee'])"
```

---

## 4. AQUA Report Handling

### 4.1 When to Use

When picking up a ticket with priority **Blocker** — its latest comment is typically an AQUA Pipeline report.

### 4.2 Blocker Policy

When a ticket has priority **Blocker**, **ALL** findings must be resolved regardless of their stated severity (Minor, Info, etc.). Do not dismiss, downplay, or deprioritize any finding. No exceptions.

### 4.3 Workflow

1. **Read the report** — if `AQUA.md` exists in the module directory, use that as the report. Otherwise fetch from Jira:
   ```bash
   /snap/bin/jira-cli issue view OPALE-XX --comments 5
   ```

   AQUA reports contain: overall status (PASS/PARTIAL/FAIL), gate results (Gate 0–2), feature inventory, code quality issues with severity, and prioritized recommendations.

2. **Assign and transition:**
   ```bash
   /snap/bin/jira-cli issue assign OPALE-XX "<your-email>"
   /snap/bin/jira-cli issue move OPALE-XX "In Progress"
   ```

3. **Verify module is activated** before running tests or the quality gate:
   ```bash
   docker compose -f <project-root>/docker-compose.yml exec <web-container> \
     vendor/bin/oe-console oe:module:list | grep <module-id>
   ```
   If it shows "Not active", run the full module lifecycle (see section 10). Integration tests will fail with `ServiceNotFoundException` if the module isn't activated, producing misleading error counts.

4. **Analyze all findings** — typical AQUA findings include:
   - `innerHTML` without client-side escaping (XSS)
   - Missing `data-testid` attributes
   - Inconsistent styling approaches
   - Unused translation keys
   - Code quality patterns (Registry usage, concrete class injection, etc.)

5. **Fix, test, report:**
   1. Fix all findings from the report
   2. Run the full quality gate with report generation (see section 7)
   3. Commit and push
   4. Add a completion comment to Jira with changes and quality gate results
   5. Follow the completion workflow (see section 5.9)

---

## 5. Module Release Workflow

### 5.1 Prepare Local Git

From the module directory (`<docroot>/vendor/opal/<module-id>/`):

```bash
git init
git checkout -b b-<major>.<minor>.x
```

### 5.2 Generate Quality Reports (Mandatory)

Before committing, generate fresh reports in all formats. QA requires these in the repo.

```bash
docker compose -f <project-root>/docker-compose.yml exec <web-container> \
  vendor/bin/oe-console qualitytools:check vendor/opal/<module-id> --caller=ai --report-format=all --report-dir=reports
```

This produces `reports/quality-report.html`, `reports/quality-report.json`, and `reports/quality-report.md`. All three **must be committed**.

### 5.3 Update .gitignore

Standard `.gitignore` content — `reports/` is deliberately NOT excluded:

```
.phpunit.cache/
vendor/
```

### 5.4 Replace Development Files with README

- **Remove** `SCOPE.md`, `TODO.md`, and `AQUA.md` (internal planning/evidence documents, not for the repo)
- **Create** `README.md` following the established opal module pattern (use any existing released OPAL module in the `OXID-OPAL` GitHub org as a reference)

README structure:
1. Title + one-line description
2. Features (bullet list)
3. Requirements (OXID version, PHP version)
4. Installation
5. Usage — admin configuration and frontend behavior
6. How It Works — technical explanation
7. Uninstallation steps
8. Quality Assurance — PHPStan, PHPCS, native metrics, coverage, test count (PHPMD for pre-v3 toolchains)
9. Support + License

### 5.5 README Content Rules

**Installation section** — do NOT document steps handled automatically by `composer require`:
- `oe:module:install`, `oe:module:install-assets`, `oe:module:apply-configuration` (all automated by `oxideshop-composer-plugin`)

DO document (manual CLI steps):
- Migrations: `vendor/bin/oe-eshop-db_migrate migrations:migrate <module-id>` (if module has them)
- DB views regeneration: `vendor/bin/oe-eshop-db_views_generate` (if module touches DB schema)

Obsolete commands that must NOT appear: `oe:database:migrate`, `oe:migration:run`, `generate_views.php`, raw SQL snippets.

**Uninstallation section:**
1. Deactivate: `vendor/bin/oe-console oe:module:deactivate <module-id>`
2. Remove: `composer remove opal/<package-name>`
3. Clear cache: `vendor/bin/oe-console oe:cache:clear`
4. Optionally: `DROP TABLE` statements for module tables

Do NOT mention `oe:module:uninstall` — it is deprecated in OXID 7.x (handled by composer).

### 5.6 Commits

```bash
# First commit: all module code
git add <all files>
git commit -m "OPALE-XX: Add opal/<module-id> module

<One-line description of what the module does.>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"

# Second commit: replace dev docs with README
git rm SCOPE.md TODO.md
git add README.md .gitignore
git commit -m "Replace SCOPE.md and TODO.md with comprehensive README.md

Remove internal planning documents and add user-facing documentation
following the established opal module README pattern.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 5.7 Create GitHub Repository

Create a **private** repo in the `OXID-OPAL` GitHub org, named after the module ID:

```bash
gh repo create OXID-OPAL/<module-id> --private --description "<module description>"

# If --source/--push flags fail (SSH issues), set HTTPS remote and push manually:
git remote set-url origin https://github.com/OXID-OPAL/<module-id>.git
git push -u origin b-<major>.<minor>.x
```

### 5.8 Set Default Branch and Team Permissions

```bash
# Set default branch
gh repo edit OXID-OPAL/<module-id> --default-branch b-<major>.<minor>.x

# Give market-bridge team admin access
gh api -X PUT orgs/OXID-OPAL/teams/market-bridge/repos/OXID-OPAL/<module-id> -f permission=admin
```

Expected permissions after setup:

| Team | Permission |
|------|-----------|
| market-bridge | admin |
| opal-manager | push |
| oxid-support | pull |
| oxid-product-management | pull |
| oxid-tools | pull |

Verify:
```bash
gh api repos/OXID-OPAL/<module-id>/teams --jq '.[] | "\(.slug): \(.permission)"'
```

### 5.9 Update Jira on Completion

1. **Add completion comment** with implementation details, repo URL, features, and quality gate results:
   ```bash
   /snap/bin/jira-cli issue comment add OPALE-XX --template - <<'COMMENT'
   h3. Implementation Complete

   Module *opal/<module-id>* has been implemented and is ready for review.

   h4. Repository
   [https://github.com/OXID-OPAL/<module-id>|https://github.com/OXID-OPAL/<module-id>] (branch: b-<major>.<minor>.x)

   h4. What it does
   <Brief description>

   h4. Key features
   * <Feature 1>
   * <Feature 2>

   h4. Quality
   * PHPStan Level 8: 0 issues
   * PHPCS PSR-12: 0 violations
   * Native metrics: 0 blocking violations
   * Test coverage: XX% (N unit tests)
   COMMENT
   ```

2. **Set priority to "Soll"** (ID `10001`) via REST API workaround (see section 3.2)

3. **Unassign:**
   ```bash
   /snap/bin/jira-cli issue assign OPALE-XX x
   ```

4. **Move to Support-Loop:**
   ```bash
   /snap/bin/jira-cli issue move OPALE-XX "Review it"
   ```

5. **Verify:** status = "Support-Loop", assignee = None (see section 3.5)

---

## 6. Claude Code Usage Tracking

**Must be the last step** after all other completion steps.

### 6.1 Session Data Location

Claude Code stores session data as JSONL files under:

```
~/.claude/projects/<project-path-with-dashes>/*.jsonl
```

The project path is derived from the absolute path to the module directory with slashes replaced by dashes. Each JSONL file = one session. Each line is a JSON object with `type` (user, assistant, progress, etc.) and `timestamp` (ISO 8601).

### 6.2 Token Extraction

Token data is in `assistant` type messages under `message.usage`:

| Field | Meaning |
|-------|---------|
| `input_tokens` | Direct input tokens |
| `output_tokens` | Model output tokens |
| `cache_read_input_tokens` | Tokens read from prompt cache |
| `cache_creation_input_tokens` | Tokens written to prompt cache |

Sum across all lines in all session files for the module.

### 6.3 Active Time Calculation

Use message timestamps with a **5-minute idle threshold**:

1. Collect all `timestamp` fields from all messages (all types) across all session files
2. Parse ISO timestamps and sort chronologically
3. Sum gaps between consecutive timestamps where `gap <= 300 seconds`
4. Gaps > 5 minutes = idle time (overnight, breaks, abandoned sessions)

Why 5 minutes: gap distribution analysis showed 99.1% of gaps are under 1 minute (agent working continuously). Only 0.1% fall between 5–30 minutes — a sharp cliff after 5 min.

### 6.4 Report to Jira

1. **Add comment** with usage table:
   ```bash
   /snap/bin/jira-cli issue comment add OPALE-XX "## Claude Code Usage Report

   | Metric | Value |
   |---|---|
   | Sessions | N |
   | Messages | N |
   | Input tokens | N |
   | Output tokens | N |
   | Cache read tokens | N |
   | Cache write tokens | N |
   | **Total tokens** | **N** |
   | **Active work time** | **Xh Ym** |

   Active work time calculated from message timestamps with a 5-minute idle threshold."
   ```

2. **Add worklog:**
   ```bash
   /snap/bin/jira-cli issue worklog add OPALE-XX "Xh Ym" \
     --comment "Claude Code active work time (calculated from session message timestamps, 5-min idle threshold)" \
     --no-input
   ```

3. **Incremental updates:** If a previous report exists, only log the **delta** (subtract previously reported values). The new comment should show Previous / Current / Delta columns.

### 6.5 Verify Time Tracking

```bash
AUTH_HEADER=$(/snap/bin/jira-cli issue view OPALE-XX --raw --debug 2>&1 | grep -oP 'Authorization: Basic \K\S+' | head -1)
curl -s -H "Authorization: Basic ${AUTH_HEADER}" \
  "https://oxid-esales.atlassian.net/rest/api/3/issue/OPALE-XX?fields=timetracking"
```

Note: Jira uses German locale — `timeSpent` shows as "0,56t" (Tage = days). Worklogs are additive.

---

## 7. Quality Gate Requirements

| Tool | Requirement |
|------|-------------|
| PHPStan | Level 8, 0 errors (includes the ported PHPMD-equivalent OXID rules) |
| PHPCS | 0 violations (PSR-12 + OXID rules) |
| Native metrics | 0 blocking violations (method/class length, method counts, N-Path, WMC, coupling, fields) |
| Tests | 100% passing |
| Coverage | >= 80% |
| Complexity | CRAP < 30, CC, NPath < 200 |

> **PHPMD removed in v3.** As of `oxid-quality-tools` v3.0.0 there is no external
> PHPMD dependency — its checks are reimplemented as the in-tree native metrics
> engine plus PHPStan AST rule ports. Older 2.x docs that list a separate "PHPMD"
> tool refer to the pre-v3 toolchain.

### Running Quality Checks

All commands run inside the Docker container. Never run underlying tools (phpunit, phpstan, etc.) directly — always use the `qualitytools:check` wrapper.

```bash
# Full quality gate (run before every commit)
docker compose -f <project-root>/docker-compose.yml exec <web-container> bash -c \
  'cd <shop-root> && vendor/bin/oe-console qualitytools:check vendor/opal/<module-id> --caller=ai'

# Individual check (replace --phpstan with --phpmd, --phpcs, --test, etc.)
docker compose -f <project-root>/docker-compose.yml exec <web-container> bash -c \
  'cd <shop-root> && vendor/bin/oe-console qualitytools:check vendor/opal/<module-id> --phpstan --caller=ai'

# Coverage (implicitly runs tests — NEVER combine with --test)
docker compose -f <project-root>/docker-compose.yml exec <web-container> bash -c \
  'cd <shop-root> && vendor/bin/oe-console qualitytools:check vendor/opal/<module-id> --coverage --caller=ai'

# Generate reports in all formats (for release)
docker compose -f <project-root>/docker-compose.yml exec <web-container> bash -c \
  'cd <shop-root> && vendor/bin/oe-console qualitytools:check vendor/opal/<module-id> --caller=ai --report-format=all --report-dir=reports'
```

### Key `--caller=ai` Behavior

Using `--caller=ai` produces output in TOON format (compact, ~30-60% fewer tokens than JSON) — the source of truth for all quality assessments. Without it, output is formatted for human consumption with colors.

`exit_code` is the gate verdict. `total_issues` is the count of **all** surfaced
findings and **includes non-blocking advisories** (rules with `blocks_gate: false`,
e.g. `metrics.shellCommandExecution`). Therefore `exit_code: 0` together with
`total_issues > 0` is valid — it means the only findings were advisories, which
surface for visibility but never fail the gate.

### Toolchain Version & OXID Targeting

`oxid-quality-tools` (`OXID-eSales/oxid-quality-tools`) is the quality gate behind
every OPAL release. Tool version maps to a single OXID minor — use the version that
matches the shop:

| Tool version | Branch | OXID target |
|---|---|---|
| v1.x | `b-7.4.x` | OXID 7.4 |
| v2.9+ | `b-7.5.x` | **OXID 7.5 only** |
| v3.x | `b-7.5.x` | **OXID 7.5 only** |

- **v2.9 and later are OXID 7.5 releases only.** There is no OXID 7.4 backport of the
  2.9+ line — run it against an OXID 7.5 shop. The last 7.4-targeted line is v1.x on
  `b-7.4.x`.
- **v3.0.0 is published** — tag `v3.0.0` on branch `b-7.5.x`. v3.0.0 **removes the
  external PHPMD dependency**: the former PHPMD checks are reimplemented as an in-tree
  native metrics engine (method/class length, method counts, N-Path, WMC, coupling,
  fields) plus PHPStan AST rule ports, so reports and gate output no longer reference
  PHPMD.
- v3 also introduces **non-blocking advisories** (e.g. `metrics.shellCommandExecution`,
  which recommends Symfony Process with an argument vector / explicit cwd+env / bounded
  timeout instead of `exec()`/`shell_exec()`/`proc_open()`/`Process::fromShellCommandline()`).
  Advisories surface in reports but never fail a module gate.

---

## 8. Quality Tools Improvement Suggestions

When encountering things that `qualitytools` should detect or report better, file them as GitHub issues on the [oxid-quality-tools](https://github.com/OXID-eSales/oxid-quality-tools) repository. Be precise, separate concerns clearly — each suggestion should be its own issue.

---

## 9. Session Initialization Checklist (AI Agents)

This section applies when using AI coding agents (Claude Code with Serena/Freya MCP servers).

**On session start:**
1. Call Freya `list_workflows()` to load available workflows and quality rules (the older `list_sections()` is a deprecated shim)
2. Read relevant Serena memories (project context, module patterns, conventions)
3. Tests before code (TDD workflow)

**After context window compaction:**
1. Re-read AI-RULES.md from the quality tools module directory
2. Read relevant Serena memories
3. Verify all rules are understood
4. Resume work following defined workflows

---

## 10. Module Lifecycle (Refresh/Reinstall)

Run this sequence inside the Docker container whenever `metadata.php`, `services.yaml`, `composer.json`, or controllers change:

```bash
docker compose -f <project-root>/docker-compose.yml exec <web-container> bash -c 'cd <shop-root> && \
  vendor/bin/oe-console oe:module:deactivate <module-id> && \
  vendor/bin/oe-console oe:module:uninstall <module-id> && \
  vendor/bin/oe-console oe:cache:clear && \
  composer update && \
  vendor/bin/oe-console oe:module:install vendor/opal/<module-id> && \
  vendor/bin/oe-console oe:module:activate <module-id>'
```

Skip `composer update` if only `services.yaml` or templates changed.
