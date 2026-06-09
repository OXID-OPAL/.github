# OXID-OPAL

Internal tools and modules for OXID eSales development.

**[OPAL Processes](../OPAL_PROCESSES.md)** — Module lifecycle from ticket to release: naming conventions, Jira workflow, AQUA reports, quality gates, GitHub setup, usage tracking.

---

## 🔮 Quality Tools — Freya MCP Server

> Available since v2.1.0, **HTTP transport since v2.0** &bull; `b-7.4.x`

AI agents can now **browse the full quality rule catalog before writing code**,
eliminating trial-and-error cycles with the quality gate.

### Freya — Rule & Workflow Browsing for AI Agents

**Freya** is a read-only [MCP](https://modelcontextprotocol.io/) server that
exposes all quality rules and AI development workflows over HTTP (JSON-RPC 2.0
via OXID frontend controller). Four tools:

| Tool | Description |
|------|-------------|
| `list_rules` | Browse all rules with ID, severity, title. Filter by tool (`phpstan`, `phpmd`, `phpcs`, `structure`, `templates`, etc.) |
| `get_rule` | Full details: what the rule detects, what to do instead, severity, documentation link |
| `list_workflows` | Browse AI-RULES.md workflows with "read when" triggers — call at session start |
| `get_workflow` | Load a specific workflow (TDD, retrofitting, evidence rules, etc.) by ID |

> Legacy `list_sections` / `get_section` tool names still work through a
> deprecation wrapper (since v2.7.0) but will be removed in v3.x — migrate
> proactively.

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

## 🧩 Quality Tools v3.1.0 — Per-File Coverage, Module Waivers, Component Targets

> **🔵 OXID 7.5** &bull; `b-7.5.x` &bull; tag `v3.1.0` &bull; 9 June 2026

A backwards-compatible feature + fix release clearing the open GitHub issue
backlog (#10–#19). New functionality is additive and the rule changes only
reduce false positives, so existing consumers are unaffected — a minor bump.

### Added

- **Per-file coverage breakdown** — coverage output now lists each file's
  `path / covered / total / percent`. The `--caller=ai` TOON view shows the
  files below threshold, lowest-coverage first; JSON reports carry the full
  list. No more dropping to a raw PHPUnit run to find what drags coverage down.
- **Module-level waiver API** — a module may ship a `.qualitytools.rules.php` to
  add narrow, reason-tagged waivers for `phpstan.staticAccess` and
  `phpstan.rawSqlDetection`. The merge is **append-only and whitelisted**: a
  module can never weaken a rule's severity, change its detection, or disable it.
  `ContainerFactory::getInstance()` is now a sanctioned default for
  chain-instantiated classes.
- **`oxideshop-component` targets** — `qualitytools:check <package>` now accepts
  OXID 7.4+ components (composer `type: oxideshop-component`, no `metadata.php`),
  analysing them as a directory target.
- **`@mixin` docblock resolution** — `{@inheritDoc}` validation honours a
  class-level `@mixin`, so extension classes whose real parent is a runtime-only
  `_parent` alias can document overrides cleanly.

### Fixed

- **Docblock inheritance resolution** — file-level `use` imports were collected
  as traits, producing spurious "remove {@inheritDoc}" findings; trait collection
  is now in-class only. This also fixed two latent bugs it had been masking
  (qualified-import FQN doubling, and `implements` on anonymous classes).
- **HTML report no longer fatals on docblock findings** — null / missing issue
  fields are defaulted and the rule + severity badge render correctly.
- **`admin_twig` theme directory accepted** — admin-only modules overriding admin
  Twig blocks no longer carry an unfixable structure violation.
- **Composer constraint** relaxed to `^7.5 || dev-b-7.5.x` so the tools install
  alongside a released 7.5 metapackage, not only the dev branch.

Full per-issue disposition is in the
[CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.5.x/CHANGELOG.md).

### Stats

- Full gate: **exit 0, 0 blocking issues, 1083 tests pass, coverage ~83.9%**,
  docblock 100%, max complexity 8

---

## 🚀 Quality Tools v3.0.0 — PHPMD-Free, Native Metrics Engine

> **🔵 OXID 7.5** &bull; `b-7.5.x` &bull; tag `v3.0.0` &bull; 26 May 2026

The first major since the v2.0 rebuild. v3.0.0 **removes the external PHPMD
dependency entirely** — every former PHPMD check is reimplemented in-tree, **with
no loss of v2.8 quality scope**. `phpmd/phpmd` is no longer a composer dependency,
the `--phpmd` / `--no-phpmd` flags are gone, and `qualitytools:check` no longer
dispatches a PHPMD analyser.

### PHPMD → native, no scope loss

- **Native metrics engine** (nikic/php-parser, in-tree) — method/class length,
  method & public-method counts, N-Path, WMC, field counts, plus the OO metrics
  number-of-children, depth-of-inheritance and coupling-between-objects.
- **PHPStan AST rule ports** — `addslashesDetection`, `avoidOxNewInServices`,
  `deprecatedDatabaseProvider`, `magicPropertyAccess`, `directViewDataAccess`,
  `rawSqlDetection` (expanded), `staticAccess`, the naming-convention family
  (long/short class, variable, and method names; boolean getters), and the
  DTO-aware parameter-list / boolean-flag rules.
- Every v2.x `phpmd.*` rule_id has a documented disposition (ported / covered /
  dropped-with-reason) — see the migration table in the
  [CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.5.x/CHANGELOG.md).

### Also new in v3

- **PHPUnit notice surfacing** — PHPUnit 12 "mock without expectations" notices
  surface as first-class per-test events (`tests.phpunitNoticeEvent`) plus a
  counter, the same way warnings and deprecations already did.
- **Execution hardening** — internal subprocess execution no longer builds shell
  command strings. A structured `ProcessCommand` (argv + cwd + env + timeout) runs
  through Symfony Process with no shell, so tool arguments can never be re-parsed
  by a shell.
- **PHPUnit test-selection passthrough** — `--filter`, `--group`,
  `--exclude-group` modifiers forward straight to PHPUnit (full gate or `--test`)
  without tripping purpose-flag exclusivity.
- **Non-blocking advisories** — new `blocks_gate` rule-data flag. The first
  advisory, `metrics.shellCommandExecution` (`info`), flags direct
  `exec()` / `shell_exec()` / `proc_open()` / `Process::fromShellCommandline()` and
  recommends Symfony Process — it surfaces in reports and `total_issues` but
  **never fails a module gate**.

### Compatibility

- **OXID 7.5 only** (`b-7.5.x`). Modules still on **OXID 7.4** stay on the **2.8**
  line (below). A pre-cutoff snapshot of the v2.9.0 PHPMD-tagged rules is preserved
  on the `b-7.5.x-v2.9` branch and the `v2.9.0` tag for modules that still want
  PHPMD coverage.

### Stats

- **147 rules** total — incl. 47 PHPStan and 12 native-metrics rules; PHPMD
  composer dependency **removed**
- Full gate: **exit 0, 0 blocking issues, 1052 tests pass, coverage 83.09%**,
  docblock 100%, max complexity 8

---

## 🌉 Quality Tools v2.9.0 — OXID 7.5 Bridge

> **🔵 OXID 7.5** (first 7.5 release) &bull; `b-7.5.x` &bull; 20 May 2026

The branch point from `b-7.4.x` to `b-7.5.x`. **v2.9 targets OXID 7.5 shops only**
— composer constraints do not resolve on a 7.4 stack by design; modules still on
7.4 keep the 2.8 line. PHPMD still ships in 2.9, but every `phpmd.*` rule now
carries a Freya deprecation banner flagging its removal in v3.0.

- **New PHPStan engine arms** — `method_call_banned` (inheritance-aware via
  `target_class_extends`), a `method_name` filter on `static_call_banned`, and a
  new `function_call_banned` engine (`ConfigDrivenFuncCallRule`, fail-closed on
  unknown argument matchers).
- **New 7.5 forward-compatibility rules** — `phpstan.coreGetContainer`,
  `phpstan.coreDispatchEvent`, `phpstan.idFromUid` (→ `Id::fromString()`),
  `phpstan.legacyGlobalFunctions`.
- **Freya `## Deprecated` banner** on every `tool=phpmd` rule — telegraphs the v3
  PHPMD removal to AI agents, keyed off rule data.

> A snapshot of the v2.9.0 rule set (PHPMD still present) is preserved on the
> `b-7.5.x-v2.9` branch and the `v2.9.0` tag.

---

## 🛡️ Quality Tools v2.8.0 — PHPUnit Non-Fatal Surfacing + Cross-Module Activation-Gate Detection

> **🟢 Last OXID 7.4-compatible release** &bull; `b-7.4.x` &bull; 20 May 2026

Large additive release. Three new detection capabilities plus an output
surface that no longer hides PHPUnit's non-fatal events behind an opaque
`total_issues: 1`. **Not a major** — no CLI removed, no schema break, no
deprecation actually retired; 3.0.0 is reserved for those.

### New: PHPUnit Non-Fatal Surfacing in TOON

`qualitytools:check ... --caller=ai` now exposes what was previously collapsed
into an opaque `summary.total_issues: 1`:

- **Counters** under `summary.tests.warnings`,
  `summary.tests.deprecations`, `summary.tests.phpunit_deprecations` — driven
  by new `summary_field` rule-data metadata
- **Per-event detail** under top-level `test_events[]` with
  `{rule_id, test, file, line, message}` for `tests.warningEvent` and
  `tests.deprecationEvent`; `file`/`line` are empty for
  `tests.phpunitDeprecationEvent` whose runner-deprecation source format
  carries only test name + message on the title line
- **Summary-footer counter notices** under top-level `test_notices[]` with
  `{rule_id, message}` — one entry per category that triggered

Six new tests-engine rules in `config/rules.php` (3 counters + 3 per-event)
drive this end-to-end. `TestCheck` reads `summary_field` / `event_output`
from rule data — **no hardcoded rule_id mappings**. All five output plugins
(Ai/Human/Html/Json/Markdown) render the new fields.

### New: `phpstan.taggedIteratorWithoutActivationGate`

New PHPStan rule plus generic engine
`ConfigDrivenIterationGateRule`. Detects classes that consume a Symfony
`!tagged_iterator` constructor argument and `foreach` it without calling an
activation/availability gate on each element. The engine correlates the
constructor's iterable parameter with `foreach` AST in other methods of the
same class and enforces a **strict receiver**: `$item->isActive()` on the
iterated element OR `$this->property->isActive()` where the property type
matches `activation_service_class_pattern`. Heuristic-free.

Severity: warning. `fix_suggestion` quotes the framework-blessed
`ModuleActivationBridgeInterface` / `ModuleStateServiceInterface` recipe
verbatim — agents see the exact API name, not a hand-wavy "use the bridge".

### New: `structure.crossModuleTagActivationGate`

New structure rule plus cross-module YAML scanning infrastructure that
qualitytools previously lacked:

- `Service/Yaml/YamlLoader(+Interface)` — shared
  `Yaml::parse(..., PARSE_CUSTOM_TAGS)` surface, factored out of
  `AdminStructureValidator` so both consumers use one parser
- `Service/Module/ServicesYamlScanner(+Interface)` enumerates every module's
  `services.yaml`, indexes tag producers + `!tagged_iterator` consumers
  (`ServicesYamlIndex`)
- `Service/Module/CrossModuleActivationGateValidator` uses
  `RecursiveDirectoryIterator` for a true recursive source scan (PHP's
  `glob('src/**/X')` does not actually descend) and runs the rule's
  `gate_method_pattern` against method names extracted with a generic regex
- `symfony/yaml` promoted from transitive to direct composer dependency

Catches the OPALE-97 antipattern at module-development time: tag producers
in module A consumed by module B with no activation/availability gate
method.

### New: `phpstan.providerContractMissingAvailabilityGate`

New PHPStan rule via `interface_member_required` arm on
`ConfigDrivenClassNodeRule`. For interfaces matching
`*Provider|*Strategy|*Extension` whose implementations are tagged across
modules (per the cross-module index), the rule requires the interface to
declare a non-nullable `bool` member matching
`isActive|isAvailable|isModuleActive|isModuleAvailable`.

**Return-type matching is strict** — non-nullable `ReflectionNamedType`
only; nullable `?bool`, union `bool|string`, and intersection types are
all rejected because they would widen the boolean gate contract and let
non-bool values pass through consumer filters.

`Service/Module/ProviderInterfaceIndex` derives the cross-module scan
root **per analysed file** (composer.json walk + `OXID_QT_PROVIDER_INDEX_ROOT`
env override) so the rule works regardless of how the project lays out
its module tree.

### Plus: `phpstan.extensionOverrideTypeHint`

Flags type hints added to parent-method overrides in Extension classes —
they break the module-chain contract by narrowing what subsequent overriders
can accept.

### Integration Test Pattern (Items 2A + 2B)

Fixture producer/consumer module trees under
`tests/Fixtures/PhpStan/ClassNode/{InterfaceMember,IterationGate}/`. Driving
tests call PHPStan as a subprocess with `--error-format=json` and assert
exit-code + JSON validity before checking for the rule identifier — silent
PHPStan crashes can no longer masquerade as a clean run.

### Stats

- **122 rules** total (Freya `list_rules`), up from 116 at v2.7.0 release
- Per-tool: phpstan 17, phpmd 20, phpcs 2, structure 20, admin 9,
  testStructure 6, docblock 2, template 21, suppression 4, complexity 3,
  coverage 4, tests 13, i18n 1
- **2760 tests** pass, 0 issues across all tools, **coverage 81.64%**,
  docblock 100% (1429/1429), module structure score 82 with 0 violations

<details>
<summary><b>Previous: v2.7.0 — Theme-Override Intelligence + Freya API Refresh (April 2026)</b></summary>

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

### Stats (at release)
- 115 rules, 2598 tests, 0 violations
- Catalog-driven coverage: 35 widget-only methods discovered live vs an
  8-method curated draft

</details>

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

- **oxid-quality-tools** — Unified quality checking for OXID modules (PHPStan, PHPCS, native metrics, PHPUnit, coverage; PHPMD removed in v3, retained on the 2.x / OXID 7.4 line)
