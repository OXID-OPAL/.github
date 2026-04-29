# OXID-OPAL

Internal tools and modules for OXID eSales development.

**[OPAL Processes](../OPAL_PROCESSES.md)** — Module lifecycle from ticket to release: naming conventions, Jira workflow, AQUA reports, quality gates, GitHub setup, usage tracking.

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

## 🧩 Quality Tools v2.7.0 — Theme-Override Intelligence + Freya API Refresh

> `b-7.4.x` &bull; 29 April 2026

### New Theme-Override Rules (all DATA in `config/rules.php`)
- `template.unknownBlockOverride` (error) — overrides defining `{% block X %}`
  with names absent in the parent theme template AND every other active
  module's override at the same path. Catches typos and AI-hallucinated block
  names that would silent-no-op at runtime. Multi-module catalog aggregates
  blocks from theme source + sibling module overrides; the file under
  inspection is excluded so it can't self-validate
- `template.widgetOnlyMethodInControllerContext` (error) — calls to widget-only
  methods from controller-context theme overrides (top-level path, NOT under
  `inc/`) crash at runtime with `SystemComponentException`. The catalog of
  widget-only methods is **discovered statically from OXID-core widget vs
  controller class files at gate-time** and auto-adapts to upstream changes —
  35 methods found live, vs an 8-method curated draft. No hardcoded list to
  drift when core evolves

### Smarter Existing Rule Descriptions
- `template.extendsPathMismatch` description rewritten with the kausal model:
  theme overrides register against file path, `{% extends %}` controls content
  source, divergence means file at A renders content extended from B.
  Concrete `tabs.html.twig` ↔ `footer.html.twig` example
- `template.incExtendedFromParentLevel` description names representative
  widget-only methods and references Phase 2 widget-context rendering
  explicitly so the SystemComponentException trail of causation is concrete

### Freya: `sections` → `workflows` API Refresh
- MCP tools renamed: `list_sections`/`get_section` → `list_workflows`/
  `get_workflow`. The IDs themselves (`setup`, `core-rules`, `freya`, …) are
  unchanged — calls with the legacy `section_id` argument resolve identically
  through the shim
- Old tool names remain functional via a thin deprecation wrapper that
  prepends `Warning: <oldname> is deprecated, use <newname> instead.` to the
  response. Old names are NOT advertised in `tools/list`; new MCP clients
  discover only workflow tools
- AI-RULES.md markers renamed `<!-- section-id: … -->` →
  `<!-- workflow-id: … -->`
- **The deprecation wrapper will be removed in a future major version (v3.x)**
  — every consumer (Steuerungsdateien, scripts, prompts) referencing the old
  names should migrate proactively

### Architecture
- **2 new detection engines** — `theme_block_override`
  (UnknownBlockOverrideDetector) and `controller_context_widget_method`
  (WidgetOnlyMethodDetector + WidgetMethodCatalog). Both read their config
  (regex, glob patterns, paths) from `config/rules.php` as data; engine code
  stays generic
- **`FileSystemInterface::glob()`** added; mockable wrapper around PHP's
  `glob()`, foundational for catalog-style detectors

### Stats
- 115 rules, 2598 tests, 0 violations
- Catalog-driven coverage: 35 widget-only methods discovered live vs an
  8-method curated draft

<details>
<summary><b>Previous: v2.6.0 — Smarter Rules, Less Friction (April 2026)</b></summary>

**New rules (all DATA):** `structure.legacySmartyTemplatesArray` (warning),
`structure.missingTemplateFile` (error), `tests.runnerDeprecations` (warning),
`i18n.hardcodedUserFacingString` (warning).

**Smarter existing rules:** `phpstan.interfaceInjection` exempts value-shaped
types via suffix (`Dto`/`DTO`/`Vo`/`VO`/`ValueObject`); `template.rawAssetInclusion`
exempts admin templates (raw `<script>`/`<link>` by convention).

**Architecture:** 3 new generic engines (`tests_output`, `admin_template_resolve`,
`php_source`) + `I18nCheck`. All patterns and exclusions live as data.

**Stats:** 113 rules, 2526 tests, 80.76% coverage, 0 violations. Validation
sweep across 6 OPAL modules pre-release caught 2 hotfix bugs.

</details>

<details>
<summary><b>Previous: v2.5.0 — New Rules, False-Positive Fixes, Security Hardening (April 2026)</b></summary>

**New rules (all DATA):** `template.extendsPathMismatch` (error),
`template.incExtendedFromParentLevel` (error), `structure.tableNamingConvention`
(warning), `phpstan.extensionOverrideTypeHint` (error)

**False-positive fixes:** `phpmd.dtoAwareBooleanFlag`,
`phpmd.extensionMethodPrefix`, `phpstan.serviceInExtension`,
`testStructure.infrastructureInUnit`, `testStructure.bootstrapHardcodedPath`

**Security:** FileLoaderController moved to admin-only, CORS wildcard removed,
error responses sanitized, reports use relative paths, path traversal blocked
at input level

**Stats:** 108 rules, 2427 tests, 80.71% coverage, 0 violations

</details>

<details>
<summary><b>Previous: v2.4.0 — Module Settings Translation Rules (March 2026)</b></summary>

- **3 new settings translation rules** — all DATA in `config/rules.php`,
  detected by new `ModuleSettingsValidator` engine:
  - `structure.missingModuleOptionsFile` (error) — metadata.php has settings
    but `module_options.php` missing in de/en
  - `structure.missingModuleOptionsKeys` (warning) — missing `SHOP_MODULE_*`,
    `SHOP_MODULE_GROUP_*`, or select option `SHOP_MODULE_{name}_{value}` keys
  - `structure.settingsLangInWrongFile` (warning) — `SHOP_MODULE_*` keys in
    module lang files instead of module_options.php (wrong loading behavior)
- **Select constraint validation** — `type=select` settings with pipe-delimited
  constraints get each option value checked for its translation key
- **`structure.blockNaming` false positive fixed** — `block` field in metadata
  blocks[] references theme blocks modules cannot rename
- **`template.rawAssetInclusion` fix_suggestion enhanced** — documents admin
  two-phase `style()` collect/render pattern

</details>

<details>
<summary><b>Previous: v2.3.1 — Template Pattern Rule Fix (March 2026)</b></summary>

- **False positive eliminated** — `template.missingThemeParent` no longer flags theme
  override templates that already contain `{% extends %}`. TemplateAnalyzer now delegates
  to PatternRuleEvaluator, correctly handling path filters and absence rules.
- Verified across 8 OPAL modules (15 false positives removed)

</details>

<details>
<summary><b>Previous: v2.3.0 — 16 OXID Certification Rules (March 2026)</b></summary>

- **16 new OXID certification rules** — template conventions, asset structure,
  metadata validation, block naming, global functions, cross-context inheritance.
  All defined as DATA in `config/rules.php` — no per-rule code
- **Pattern engine: path filtering, absence detection, metadata scope**
- **Structure engine: asset validation, metadata cross-checks**
- **Rule cross-listing** — `also_listed_under` for multiple Freya tool filters
- **AI-RULES v3.3** — Test Semantics section: unit vs integration test rules

</details>

<details>
<summary><b>Previous: v2.2.4 — Suppression Detection & OXID Certification (March 2026)</b></summary>

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

</details>

<details>
<summary><b>Previous: v2.2.3 — SQL Detection Fixes (March 2026)</b></summary>

- Concatenated constants, ExpressionBuilder false positives, `const_pattern` as data, mixed-quote constants

</details>

<details>
<summary><b>Previous: v2.2.2 — Freya v2.0 HTTP Transport (March 2026)</b></summary>

- Freya runs as OXID frontend controller over HTTP, `list_sections`/`get_section` for AI-RULES.md, `--setup` defaults to HTTP

</details>

<details>
<summary><b>Previous: v2.2.1 — Detection & Resilience Fixes (March 2026)</b></summary>

- SQL detection broadened, Freya v1.1.0 Docker-safe setup, method prefix fixes, PHPStan baseline eliminated

</details>

<details>
<summary><b>Previous: v2.2.0 — Auto-Bootstrap (March 2026)</b></summary>

- Modules without bootstrap in phpunit.xml get OXID shop bootstrap injected automatically

</details>

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
