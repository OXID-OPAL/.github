# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## 🔮 Quality Tools — Freya MCP Server

> Available since v2.1.0, **HTTP transport since v2.0** &bull; `b-7.4.x`

AI agents can now **browse the full quality rule catalog before writing code**,
eliminating trial-and-error cycles with the quality gate.

### Freya — Rule Browsing for AI Agents

**Freya** is a read-only [MCP](https://modelcontextprotocol.io/) server that
exposes all quality rules over HTTP (JSON-RPC 2.0 via OXID frontend controller).
Two tools:

| Tool | Description |
|------|-------------|
| `list_rules` | Browse all rules with ID, severity, title. Filter by tool (`phpstan`, `phpmd`, `phpcs`, `structure`, `templates`, etc.) |
| `get_rule` | Full details: what the rule detects, what to do instead, severity, documentation link |
| `list_sections` | Browse AI-RULES.md sections with "read when" triggers — call at session start |
| `get_section` | Load a specific workflow section (TDD, retrofitting, evidence rules, etc.) |

### Setup (one command)

```bash
vendor/bin/freya --setup
```

Writes `.mcp.json` to the shop root with a `streamable-http` entry pointing to
`http://localhost:<port>/index.php?cl=freya_mcp`. Port auto-detected from
`docker-compose.yml`. Restart Claude Code to connect.

> **v2.0:** Freya runs as an OXID frontend controller — no separate process,
> no Docker path issues, no stdio pipe crashes. Stdio transport deprecated.

### Workflow

1. `list_rules` → see what rules apply to your area
2. `get_rule` → understand detection pattern + fix suggestion
3. Write compliant code on the first attempt
4. `qualitytools:check` → verify

---

## 🩹 Quality Tools v2.2.4 — Suppression Detection & OXID Certification Thresholds

> `b-7.4.x` &bull; 19 March 2026

- **Suppression reporting restored** — `@codeCoverageIgnore`, `@SuppressWarnings`,
  `phpcs:ignore`, `@phpstan-ignore` visible in TOON output again (v2.0 regression)
- **Class-level suppressions are now errors** — `@codeCoverageIgnore` always error;
  other class-level annotations promoted via `class_level_severity` rule attribute
- **DTO-aware PHPMD rules** — `ExcessiveParameterList` and `BooleanArgumentFlag`
  auto-exempt DTO constructors with promoted properties (configurable heuristic)
- **OXID certification thresholds** — CC two-tier (warning ≥4, error ≥8),
  method length 80, NPath <200, CRAP <30 — from official OXID certification docs
- **ExcessiveClassComplexity enabled** — WMC>50 (was excluded since v1.0)
- **13 classes decomposed** to meet WMC threshold — 28 new focused helper classes
- **All standard PHPMD rules in `rules.php`** — Freya can now expose all thresholds

---

## 🩹 Quality Tools v2.2.3 — SQL Detection Fixes

> `b-7.4.x` &bull; 18 March 2026

- **Concatenated constants detected** — multi-line `const Q = '...' . '...'` now
  has all fragments joined before SQL keyword matching. Previously only the first
  fragment was inspected.
- **ExpressionBuilder false positives fixed** — `$expr->or()`, `$expr->eq()` passed
  to QueryBuilder methods no longer trigger the interpolated-SQL pattern. Possessive
  quantifier `\w++(?!->)` distinguishes object calls from string interpolation.
- **`const_pattern` is now data** — constant extraction regex moved from hardcoded
  PHP into `rules.php`, consistent with all other detection patterns.
- **Mixed-quote constants** — `"CASE WHEN col = 'val'"` now parses correctly.

## 🔮 Quality Tools v2.2.2 — Freya v2.0 HTTP Transport

> `b-7.4.x` &bull; 17 March 2026

- **Freya v2.0** — runs as OXID frontend controller over HTTP instead of fragile
  stdio process. No Docker path resolution, no pipe crashes.
- **`list_sections` / `get_section`** — AI-RULES.md served through Freya with
  section markers and "read when" triggers. Agents load workflows on demand.
- **MCP `instructions` field** — agents get Freya's self-description in system prompt
- **`--setup` defaults to HTTP** — port auto-detected. Stdio transport deprecated.
- 25 new unit tests

## 🩹 Quality Tools v2.2.1 — Detection & Resilience Fixes

> `b-7.4.x` &bull; 17 March 2026

- **SQL detection broadened** — catches qualified names, function calls, aggregates,
  CASE WHEN, and raw SQL fragments passed to QueryBuilder methods. All patterns
  data-driven via `rules.php`.
- **Freya v1.1.0** — Docker-safe `--setup`, dynamic container paths, connection resilience
- **Method prefix** — fixed garbled messages, added position validation (start / after verb)
- **PHPStan baseline eliminated** — autoloader in `phpstan.neon` resolved all 101
  Symfony/OXID class-not-found entries

## 🚀 Quality Tools v2.2.0 — Auto-Bootstrap

> `b-7.4.x` &bull; 17 March 2026

Modules without a `bootstrap` attribute in their `phpunit.xml` now get the OXID
shop bootstrap (`source/bootstrap.php`) injected automatically when running tests
or coverage through quality tools. Modules with their own bootstrap are unaffected.

---

<details>
<summary><b>Previous: v2.1.1 — Bugfixes (March 2026)</b></summary>

- **`admin.languageFiles` rule** — correct path and filename patterns in docs
- **`--report-dir` / `--coverage-html`** — relative paths resolve to target module
- **`admin.unknown`** — recognizes list-only and simple admin patterns
- **`phpmd.rawSqlDetection`** — now scans class constants

</details>

---

<details>
<summary><b>Previous: v2.1.0 — Freya MCP Server (March 2026)</b></summary>

AI agents can now **browse the full quality rule catalog before writing code**
via **Freya**, a read-only MCP server. Two tools: `list_rules` (browse/filter)
and `get_rule` (full detail with fix suggestions). `--setup` auto-detects Docker.

</details>

<details>
<summary><b>Previous: v2.0.0 — Architecture Rebuild (March 2026)</b></summary>

**BREAKING:** Complete internal rebuild. Data-driven rule engine (`config/rules.php`),
crash diagnostics, output plugin architecture, Check/Result pattern. All CLI options
and 24 detection rules preserved. See
[CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.4.x/CHANGELOG.md).

</details>

<details>
<summary><b>Previous: v1.4.1 — Bug Fixes & Output Parity (March 2026)</b></summary>

**Fixed:** Version-aware fix_suggestion URLs, AI mode error detection, exit code propagation, bootstrap warnings, human mode inline output, hasToolFailures() reliability

</details>

<details>
<summary><b>Previous: v1.4.0 — Admin Validation & Smarter Reporting (March 2026)</b></summary>

**Admin validator (8 checks):** Menu.xml wiring, three-controller pattern, template format/placement, controller key conventions, dual registration, language files
**Template linting:** OXID custom Twig tags (capture, ifcontent, include_content, include_dynamic, hasrights, insert_tracker)
**Reporting:** Per-file infrastructure leaks, errors in test summary, StaticAccess hint differentiation, YAML snippets, exit code on structure ERRORs
**Other:** Mock/assertion exclusion from infrastructure scanning, blanket PHPMD suppressions narrowed

</details>

<details>
<summary><b>Previous: v1.3.0 — Test Intelligence & Stricter CLI (February 2026)</b></summary>

**Test structure:** Infrastructure leak detection, suite config check, bootstrap quality, suppression reporting
**Static analysis:** ContainerFacadeInModulesRule (PHPStan), enhanced DatabaseProvider detection (PHPMD)
**CLI:** Exclusive purpose flags, `--stop-on-error` + `--report-format` error, TOON output, PHPUnit exit 255 differentiation
**Fixes:** Twig modules false positive, report data propagation

</details>

<details>
<summary><b>Previous: v1.2.0 — Major Rule Expansion (February 2026)</b></summary>

**PHPStan:** RegistryInExtensionsRule, QueryBuilderFromMisuseRule, DatabaseValueTypeComparisonRule
**PHPMD:** MagicPropertyAccess, AddslashesDetection, RawSQLDetection *(enhanced)*
**Structure:** services.yaml exclusion, Twig blocks metadata, Migration documentation
**Other:** Template `data-testid` check, ExtensionMethodPrefix false-positive fix, path fixes

</details>

<details>
<summary><b>Previous: v1.1.0 (February 2026)</b></summary>

- Markdown report format (`--report-format=markdown`)
- Combined report generation (`--report-format=all`)
- Fixed "Action Required" bug, exit code summarizing, legacy pattern suggestions

</details>

See [oxid-quality-tools](https://github.com/OXID-eSales/oxid-quality-tools) for full changelog and source.

---

## Repositories

- **oxid-quality-tools** — Unified quality checking for OXID modules (PHPStan, PHPMD, PHPCS, PHPUnit, coverage)
