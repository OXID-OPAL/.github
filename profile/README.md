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

## 🔁 Quality Tools v4.0.0 — One Engine on Every Shop Line

> **🔵 OXID 7.6 / 7.5 / 7.4 / 7.3** &bull; `b-7.6.x` → `b-7.5.x` → `b-7.4.x` → `b-7.3.x` &bull; tag `v4.0.0` on `b-7.6.x`, lines on branch heads &bull; 2 November 2026

Module CIs that install Quality Tools from a **branch head** (`dev-b-7.x.x`) pick up
the following changes the moment a line branch is merged. The series is complete: the trunk `b-7.6.x` is tagged
**[`v4.0.0`](https://github.com/OXID-eSales/oxid-quality-tools/releases/tag/v4.0.0)**, and the lines `b-7.5.x`,
`b-7.4.x` and `b-7.3.x` carry the same engine as untagged branch heads.

### Release `v4.0.0` (2 November)

A major release (semver). Breaking for module CIs and report consumers:

- **Composer:** every branch aliases to **`4.x-dev`** (one Quality Tools version on every line). A module CI pinned
  to `3.x-dev` or `^3.0@dev` of a branch no longer resolves it; require `^4.0@dev`, `4.x-dev` or the branch name
  (`dev-b-7.x.x`).
- **JSON report:** `test_structure.issues` is a list of objects instead of strings; new keys
  (`summary.failed_checks`, `summary.advisory_violations`, `tools.tests`, `tools.coverage`, `tools.phpcbf`).
- **Gate:** a module's own PHPStan config can no longer lower the gate's level or silence its run; `--coverage`
  judges the tests of its PHPUnit run; a module without tests fails with `tests.noTests` / `coverage.noTests`; PHPMD
  is bound to the line (below), and a shop below the PHP minimum of its OXID minor is stopped.
- **Rule ids:** `admin.unknown` is removed; on the 7.3 line the PHPStan identifiers and sniff codes are the
  trunk's, so ignores under the old ids stop matching; every counted finding names its rule id in every output.

Details: [CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/v4.0.0/CHANGELOG.md).

**What changes:** all shop lines now run **one engine** (the engine of `b-7.6.x`,
released as 4.0.0); rule data decides which rules apply to which OXID and PHP version. Each
line keeps its own shop and PHP constraints. Every line serves the OXID minor it targets **and the prior minor
migrating to it, each on its own PHP** (a 7.4 shop runs the 7.5 line on PHP 8.2):

| Line | OXID | PHP | PHPUnit | Status |
|---|---|---|---|---|
| `b-7.6.x` | 7.6 (and 7.5) | 8.3+ | 12 | trunk |
| `b-7.5.x` | 7.5 (and 7.4) | 8.3+ (8.2 for a 7.4 shop) | 11 or 12 | follows the trunk since 28 October; merged again **31 October** and **2 November** |
| `b-7.4.x` | 7.4 (and 7.3) | 8.2+ | 11 | moved from the **2.8 engine to the v3 engine** on **31 October**; merged again **2 November** |
| `b-7.3.x` | 7.3 (and 7.2) | 8.2+ | 11 | moved from the **v1 engine to the v3 engine** on **1 November**; merged again **2 November** |

**PHPMD is bound to the line:** PHPMD 2.15 runs with its stock rules **only on the
`b-7.4.x` and `b-7.3.x` lines, and only below PHP 8.3**. Nothing on PHP 8.3 or later
runs PHPMD, and the `b-7.5.x` and `b-7.6.x` lines never do, on any PHP: there its
PHPStan/native successors report the same defects. A `@SuppressWarnings("PHPMD.X")`
counts for the successor too. A suppression that names no PHPMD rule (or is written
in a form PHPMD ignores, e.g. a space before `(`) now **fails the gate**; one that
names a stock PHPMD rule the gate does not check is advisory.

### All lines (merges of 2 November, after `v4.0.0`)

- **Installed-package lookups skip PHPStan's own bundled packages** (#581). Once PHPStan's phar was loaded, a
  lookup could find PHPStan's copy of a package (e.g. `symfony/process`) instead of the shop's, or count a package
  as installed that only PHPStan ships. No check reads these lookups yet, so **no finding changes**; measured with
  the OPAL modules on OXID 7.6, 7.5, 7.4 and 7.3 before and after.

- **The cross-module rules measure the module and the modules it declares** (#569). `structure.crossModuleTagActivationGate`
  and `phpstan.providerContractMissingAvailabilityGate` used to count a tag as consumed across modules whenever any module
  installed next to the analysed one consumed it, declared or not. Now only the module and the OXID modules it declares
  in its composer.json `require` (directly or transitively) count, wherever they are installed. A finding can therefore
  **vanish** (only an undeclared neighbour consumed the tag) or **appear** (a declared module installed in another vendor
  directory consumes it). Both rules' finding texts now come from the rule data; the wording is unchanged. Measured
  with the OPAL modules on OXID 7.6 and 7.5 (opalerpodoo with its declared modules, migrations run): **no finding
  changes**, since none of them has a cross-module finding before or after. Merged into `b-7.6.x` and `b-7.5.x`;
  `b-7.4.x` and `b-7.3.x` follow.

### `b-7.6.x` (merges of 1 November)

Four fix packages on the trunk. A module CI on `dev-b-7.6.x` gets them now, `b-7.3.x` gets them with its merge below,
`b-7.5.x` and `b-7.4.x` got them on 2 November (below):

- **`--coverage` fails on failing tests again.** A run with coverage judged only the coverage: a failing test or an
  error in its PHPUnit run left the exit code at 0. The tests of the coverage run are now judged as with `--test`
  (exit code 1, the failures in every output), still from one PHPUnit run. The same holds when `--complexity`,
  `--coverage-html` or `--coverage-threshold` enable the coverage run; `--no-test` keeps the tests out. **A CI step
  that runs `--coverage` on a module with failing tests now fails.**
- **A metadata.php that cannot be loaded no longer ends the run.** When QT searches for a module that is not in the
  shop configuration, it skips a metadata.php that does not parse or throws, and names it in the "module not found"
  message. This crash hit OXID 7.2 shops.
- **Every finding is shown with its rule id and its rule's advice, in every output** (TOON, JSON, Markdown,
  HTML, console). The advice is never cut short. A test failure's advice now reads: find out whether the code or
  the test is wrong, and never skip, delete or loosen an assertion to make the test pass.
- **JSON shape change:** `test_structure.issues` is now a list of objects (`path`, `message`, `severity`, `rule`,
  `blocks_gate`, `fix_suggestion`), not a list of strings. The former string is `message`, unchanged. A parser that
  reads this list must be adapted.
- **A module without tests** fails with its own rules `tests.noTests` and `coverage.noTests` instead of a
  "Failed to load test suites" crash. The verdict is the same.
- **Plain functions** (`function name()`) are checked like methods: unused parameters and locals, boolean flags,
  parameter lists, complexity and length.

Measured with six OPAL modules on OXID 7.6 and 7.4 (PHP 8.3 and 8.2): **no verdict of the full gate changed**.

### `b-7.5.x` (merge of 2 November)

The 7.5 line takes the trunk fixes of 1 November (above) and the line-bound PHPMD rule:

- **A 7.4 shop on PHP 8.2 is now checked by the 7.5 line.** Before, every run stopped with
  `runtime.phpVersionTooOld` ("needs PHP 8.3"). Now it gets the full gate on its own PHP minimum, with the PHPStan and
  native metrics successors and **never PHPMD**. A 7.5 shop still needs PHP 8.3, and a shop below its own minor's PHP
  minimum is still stopped.
- **A module CI that runs the 7.5 line on a 7.4 shop under PHP 8.2 now gets a verdict:** in the six OPAL module runs
  its PHPStan and metrics findings are those of the 7.5 line on OXID 7.5 under PHP 8.3, with no `phpmd` section.

Measured on OXID 7.5 (PHP 8.3) with six OPAL modules, each activated alone, before and after the merge:
**no verdict of the full gate changed**.

### `b-7.5.x` (merge of 31 October)

Measured on OXID 7.5 with six OPAL modules, each activated alone, before and after
the merge: **no verdict changed**. One new advisory finding
(`suppression.ungatedPhpmdRule` for a `@SuppressWarnings("PHPMD.CamelCasePropertyName")`
in opalxrechnung); the other five reports have the same findings and gate effects.

### `b-7.4.x` (merge of 2 November)

The 7.4 line takes the trunk fixes of 1 November (above). PHPMD keeps running on this line below PHP 8.3, and its
successors run from PHP 8.3 on, as before. A 7.3 shop is held to its own PHP minimum, 8.2 like 7.4's. Measured with
six OPAL modules on OXID 7.4 (PHP 8.3 and 8.2) before and after the merge: **no verdict of the full gate changed**,
and the PHPMD findings are unchanged where it runs (PHP 8.2). What a module CI sees comes from the trunk fixes:
`tests.noTests` / `coverage.noTests` instead of a crash for a module without tests, `test_structure.issues` as
objects in JSON, and coverage findings under `tools.coverage`.

### `b-7.4.x` (merge of 31 October)

The 7.4 line takes the v3 engine. OXID 7.3 shops (the prior minor) run it too. Measured with the same six modules on OXID 7.4
(PHP 8.3 and PHP 8.2) before and after the merge: **no verdict changed**, but reports
grow. This replaces the "last OXID 7.4-compatible release" note on v2.8.0 below.

- **Gating findings 462 → 504** (PHP 8.3) / **503** (PHP 8.2), mostly because the
  type-aware PHPStan successors see more than the 7.4 PHPMD rules did:
  `phpstan.directViewDataAccess` +34, `phpstan.rawSqlDetection` +11,
  `phpstan.avoidOxNewInServices` +3. Method complexity is measured by the native
  metrics (`metrics.cyclomaticComplexity`, `metrics.longMethod`).
- **Advisory findings 18 → 37**, mostly suppression governance
  (`suppression.netNew` / `suppression.overBroad`).
- **Renamed rule:** `phpstan.extensionOverrideTypeHint` is reported as
  `phpstan.extensionSignatureDrift`. Waivers and suppressions with the old id still
  apply; a PHPStan ignore of the old identifier (`@phpstan-ignore
  extensionOverrideTypeHint`, neon `ignoreErrors`) no longer matches — switch it to
  `extensionSignatureDrift`.
- **Composer:** the line now requires the Codeception packages; a `composer update`
  installs six packages (Codeception modules, page objects and php-webdriver).

### `b-7.3.x` (merge of 2 November)

The 7.3 line takes the trunk's line-bound PHPMD rule and its texts. PHPMD keeps running on this line below PHP 8.3,
and its successors run from PHP 8.3 on, as before. A 7.2 shop is held to its own PHP minimum, 8.2 like 7.3's.
Measured with the five OPAL modules that install on OXID 7.3, on PHP 8.3 and PHP 8.2, before and after the merge:
**no finding changed**, PHPMD's included. All four lines now run the same engine.

### `b-7.3.x` (merge of 1 November)

The 7.3 line moves from its own **v1 engine to the v3 engine**, with the trunk fixes above. OXID 7.2 shops (the
prior minor) run it too. The line keeps PHP 8.2, PHPUnit 11 and PHPMD 2.15. Measured with the OPAL modules that install on OXID 7.3
(opalquoterequest requires OXID 7.4), on PHP 8.3 and PHP 8.2, before and after the merge:

| Module | Exit code PHP 8.3 | Exit code PHP 8.2 | Findings |
|---|---|---|---|
| opalattributemanager | 1 → 1 | 1 → 1 | 8 → 44 (11 advisory) |
| opaldashboard (no tests) | 1 → 1 | 1 → 1 | 390 → 451 / 450 (3 advisory) |
| opalfilters | **0 → 1** | 1 → 1 | 0 → 1 |
| opallastseenproducts | 1 → 1 | 1 → 1 | 2 → 27 (11 advisory) |
| opaltrackingstatus | 0 → 0 | **1 → 0** | 2 → 2 (both advisory) |

- **opalfilters now fails** on `phpstan.serviceInExtension`: the rule checks every non-private method of an
  extension, and `Search::opalHitSelect()` has complexity 6 (max 5).
- **opaltrackingstatus now passes on PHP 8.2:** the v1 engine read PHPUnit's exit code as -1 although every test
  passed.
- **`--coverage` judges the tests of its run,** as the v1 engine did; `--no-test` keeps them out.
- **A module without tests fails** (`tests.noTests`, `coverage.noTests`); the v1 engine skipped its tests.
- **Ignores that stop matching:** neon ignores by the v1 identifiers `oxid.methodPrefix`, `oxid.noMethodPrefix`
  and `oxidQualityTools.avoidConfigInjection` (now `prefixedMethodName` and `avoidConfigInjection`), ignores by the
  message of a v1 rule, and `phpcs:ignore` of the v1 sniff codes `PhpCs.Files.ModuleAssetStructure.<Category>` and
  `PhpCs.NamingConventions.UnderscoreMethodPrefix` (now `PhpCs.Files.ConfigDrivenFile.<Category>` and
  `PhpCs.NamingConventions.ConfigDrivenToken.UnderscoreMethodPrefix`).
- **Report files** move into `<module>/quality-reports/`; `--report-format=all` writes HTML, JSON and Markdown
  (v1 wrote JSON to `quality-report.all`).
- **Output for parsers:** the exit code is 0 or 1 as before. The AI output is TOON, so `jq` does not read it; the
  JSON report changes its layout (findings under `tools.<tool>.violations[]` with `rule`, `blocks_gate` and
  `fix_suggestion`). The CHANGELOG entry lists the paths.

Details per line: [CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.6.x/CHANGELOG.md)
(entries "Lines: 7.5", "Lines: 7.4" and "Lines: 7.3") and the tracking issue
[#572](https://github.com/OXID-eSales/oxid-quality-tools/issues/572).

---

## 🛡️ Quality Tools v3.2.0 — Suppression Governance, Migration Safety, Override-Signature Backbone

> **🔵 OXID 7.5** &bull; `b-7.5.x` &bull; tag `v3.2.0` &bull; 22 June 2026

A backwards-compatible feature release (minor bump) closing the remaining open
GitHub issues (#20, #21). New rules, a suppression-governance surface, and honest
reporting are additive; the only behaviour that can newly fail a gate is stricter
checks within the existing quality scope.

### Added

- **Suppression & waiver governance (SUP)** — a unified, normalized inventory of
  every escape channel (in-code annotations, the module waiver, `phpstan.neon`
  `ignoreErrors`/baseline, `phpcs.xml` excludes) surfaced in every report, plus
  advisory findings for unreasoned / net-new / over-broad suppressions and a
  clean/dirty **debt metric**. A committed `.qualitytools.suppressions.json`
  baseline drives the net-new check and an **opt-in dirty-debt gate**; everything
  else stays advisory.
- **Private service-locator detection (closes #21)** — flags a private same-module
  service fetched via `ContainerFactory`/`ContainerFacade` in an Extension class
  (a runtime `ServiceNotFoundException` that previously passed the gate green),
  cross-checked against the module's `services.yaml` visibility.
- **Migration `--dry-run` safety (MIG/SQL)** — `phpstan.directMigrationExecute`
  (error + reason-bound waiver) flags directly-executed mutating SQL that bypasses
  the `--dry-run` preview; `rawSqlDetection` is migration-aware and its waiver now
  matches `sprintf()`-composed SQL (closes #20).
- **Override-signature backbone (SIG)** — authoritative shop-root resolution,
  `extensionParentUnresolved`, and a full Core-signature drift rule, so an
  Extension override with a silently-broken signature no longer passes green.
- **Pre-encoded accessor / variant-name escaping (TPL)** — catches OXID accessors
  like `Selection::getName()` (already HTML-encoded) when double-encoded in Twig
  output or used as a lookup / array-subscript key in PHP.
- **Missing referenced classes (REF)** — validates that classes referenced from
  `services.yaml` and both sides of the `metadata.php` `extend` map exist.
- **Non-blocking finding channel (POL-1A)** — rules can be advisory
  (`blocks_gate: false`): counted and shown in every report, but they never fail
  the build.

### Changed

- **Honest reporting (TEST/MET/REP)** — loud PHPUnit capture-failure, runner
  warnings, per-test skipped/risky detail, explicit suite-crash diagnostics; the
  complexity section is CRAP-only with per-method CCN gated statically; the
  Markdown report discloses its truncation and points at the JSON source.
- **Tagged-iterator rule gated on real wiring (GATE)** — fires only for an
  object/interface value type actually wired via `!tagged_iterator` in the
  module's `services.yaml` (or whose wiring is unknown), eliminating the
  scalar-map and manually-passed-iterable false positives.

Full per-cluster disposition is in the
[CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.5.x/CHANGELOG.md).

### Stats

- Full gate: **exit 0, 0 blocking issues, 1470 tests pass, coverage ~86.4%**,
  docblock 100%, **168 rules** total (54 PHPStan), GitHub issue backlog cleared
  (#20, #21 closed)

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
