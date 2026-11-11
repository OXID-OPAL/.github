# OXID-OPAL

Internal tools and modules for OXID eSales development.

**[OPAL Processes](../OPAL_PROCESSES.md)** — Module lifecycle from ticket to release: naming conventions, Jira workflow, AQUA reports, quality gates, GitHub setup, usage tracking.

---

## 🔮 Quality Tools — Freya MCP Server

> Available since v2.1.0, **HTTP transport since v2.0** &bull; on every line: `b-7.6.x`, `b-7.5.x`, `b-7.4.x`, `b-7.3.x`

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

## 🧭 Default branch is now `b-7.6.x`

> **🔵 OXID 7.6** &bull; `b-7.6.x` is the repository's default branch &bull; 5 November 2026

The quality tools repository's default branch moved from `b-7.5.x` to `b-7.6.x`. A clone without a branch,
a pull request without an explicit base and the repository front page now show the 7.6 line. Nothing changed
in the code: all four lines keep the engine of `v4.0.0` and the merges listed below.

**Why now.** The 7.6 line passed a stability run of the quality tools' own making, independent of any downstream
release plan: 15 OPAL modules were selected so that together they exercise every engine and feature marker the
tools react to (admin menus and templates, theme block overrides, tagged services across modules, migrations,
metadata events and controllers, module and library dependencies, PHPUnit and Codeception suites, the known
suppressions, the one legacy-structure module, the largest module). Each module ran through the full gate on an
OXID 7.6 shop with `b-7.6.x` and on an OXID 7.5 shop with `b-7.5.x`, with the shop restored to its reference state
after every run. Result: the normalised finding sets are identical on 14 of the 15 modules; the one difference is a
single integration test of `opalformmanager` that asserts the client address, which OXID 7.6 resolves through the
Symfony request instead of the server variables, so the test fails on 7.6 only. No crash and no wrong finding of
the tools on either line. The module list lives in the repository as data and is reused for the next minor.

**What to do.**

- Module CIs keep their explicit constraint: `dev-b-7.6.x` for OXID 7.6 (and a 7.5 shop migrating to it),
  `dev-b-7.5.x` for 7.5, `dev-b-7.4.x` for 7.4, `dev-b-7.3.x` for 7.3, each with `^4.0@dev` or `4.x-dev`.
- Pull requests against the quality tools name their base branch; the default is the trunk `b-7.6.x`, the lines
  are `b-7.5.x`, `b-7.4.x` and `b-7.3.x`.
- Four OPAL modules could not be installed on either shop and were left out of the run: three require
  `oxid-esales/graphql-base` 12 while the 7.5 and 7.6 shops carry 13, one is Enterprise-only.

Details per line: [CHANGELOG](https://github.com/OXID-eSales/oxid-quality-tools/blob/b-7.6.x/CHANGELOG.md).

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

### All lines (merges of 2 to 6 November, after `v4.0.0`)

- **Installed-package lookups skip PHPStan's own bundled packages** (#581). Once PHPStan's phar was loaded, a
  lookup could find PHPStan's copy of a package (e.g. `symfony/process`) instead of the shop's, or count a package
  as installed that only PHPStan ships. No check reads these lookups yet, so **no finding changes**; measured with
  the OPAL modules on OXID 7.6, 7.5, 7.4 and 7.3 before and after.

- **A module whose metadata.php id is a class constant is checked in full** (#927). An id written as
  `Module::MODULE_ID` used to be invisible: such a module was not found by its id outside the shop configuration, and
  the asset naming, admin controller key, settings and migration checks skipped it silently. One reader now resolves
  the constant from the module's own composer.json without running any code, and a module that is still not found is
  named with the reason. Merged into all four lines.
- **The template name check reads its pattern from the rule data only** (#922). The scanner no longer keeps an older
  copy of the pattern in code as a silent fallback; that copy was never used with the shipped rules and would have
  missed `$this->` assignments. Rule data without the pattern now fails loudly. **No output changes.** Merged into
  all four lines.
- **A test keeps the texts about where PHPMD runs in line with the data** (#917). The rule descriptions and the
  documents that say "the 7.4 and 7.3 lines, below PHP 8.3" are now checked against the PHPMD runtime data, so the
  next change of the PHPMD lines fails the suite until every text follows. **No output changes.** Merged into all four
  lines.
- **Where PHPMD runs and when the core reads bootstrap-services.yaml are data** (#912). The PHPMD lines (7.3, 7.4)
  and PHP bound (8.3), and the core version that loads `bootstrap-services.yaml` (7.6.0), were code constants, and
  the PHPMD fact was written by hand into seven texts (help, labels, Freya's schema); all now come from config data.
  **No output changes.** Merged into all four lines.
- **`template.storefrontHelpersInAdmin` is retired** (#906). It warned against `{{ script() }}` / `{{ style() }}` in admin
  templates although OXID's own admin theme uses them on every line and a relative include resolves against the admin
  theme in admin. It found nothing on 48 OPAL modules; `template.rawAssetInclusion` now says admin templates may use
  either form. **No finding changes.** Merged into all four lines.
- **Freya names the installed release in its MCP handshake** (#901). The `initialize` answer reported
  `serverInfo.version` 2.0.0 for every release since March; it now names the `version` of the module's metadata.php
  (4.0.0 today, the number the shop's module list shows), or `unknown` when it cannot be read; a release test keeps it
  in line with the CHANGELOG. Merged into all four lines.
- **Waiver scopes name what the waiver applies to, and their fingerprints no longer collide** (#592). A static-call
  waiver's scope was its bare method and a SQL waiver's scope its path, so 103 waiver records on 48 OPAL modules
  shared 44 fingerprints and a new waiver on an already-accepted method was never reported net-new. The scope now
  carries class, method, path and markers; every record has its own fingerprint; an old baseline entry accepts its
  record once more by the old scope. 56 waivers that were accepted only through a collision now read as advisory
  net-new; **no gate result changes**. Merged into all four lines.
- **One indent for the Markdown advisory line** (#888). The Structure Violations list indented "Advisory: does not fail
  the gate" under the finding's rule line while every other section prints it beside rule and advice; it now does
  too. Measured on 48 OPAL modules: 57 lines in 28 modules move by two spaces; **no other output changes**. Merged into
  all four lines.
- **The Markdown report labels advisory template and test-structure findings** (#883). Its Template Issues and Test
  Structure Issues lists printed an advisory finding like a blocking one while the tool and structure sections already
  labelled and counted them. Each advisory row now carries "Advisory: does not fail the gate" and the heading counts
  the advisory rows. Measured on 48 OPAL modules: 40 template findings in 3 modules gain the line; **no other output
  changes**. Merged into all four lines.
- **Template warnings are advisory** (#878). The template check failed only on an error-severity finding while every
  report labelled findings by their rule's `blocks_gate`, which no template warning rule set — so a warning that left
  the check passed was shown as a blocking finding. The seven template warning rules now declare `blocks_gate: false`
  and the template check decides by that flag like the structure, PHPStan, PHPMD and metrics checks; a test keeps
  severity and gate flag in line. Measured on 48 OPAL modules: **no check result changes**; 40 warning findings
  (data-testid, inline style) now read as advisory in the JSON, HTML, Markdown and console outputs. Merged into all
  four lines.
- **The finding texts of the test structure check, the crash diagnostics, the underscore-prefix sniff and two
  structure findings are rule data** (#873). Ten texts, from "No phpunit.xml configuration found" to the crash
  diagnosis that quotes the last output line, were composed in the check and service classes; they now sit in the data
  of their five rules and are read from there. Every rendered text is byte-identical before and after, measured over
  the whole test suite and a structure run on 48 OPAL modules, so **no finding changes**. Freya lists them under the
  rules. Fourth and last slice after #848, #853 and #863. Merged into all four lines.
- **Template findings carry their rule's severity** (#868). The template check wrote some severities in its code, and
  the JSON, HTML and Markdown reports gave every template finding of a run the same severity: error when the run had a
  template error, else warning. Each finding now takes its rule's severity from the rule data, in the check and in every
  report. Measured over 48 OPAL modules: 39 findings of 2 modules (data-testid, inline style) read warning instead of
  error in the reports; **no check result changes**. Two rule-data corrections ride along: a missing `{% use %}` parent
  is a warning (it may come from a theme or module not installed here), a template Twig cannot load is an error. Merged
  into all four lines.
- **The finding texts of the template and docblock checks are rule data** (#863). Ten texts, from "Add class
  docblock for X" to "File path … does not match extends path …", were composed in the check classes; they now sit
  in the data of their six rules and are read from there. Every rendered text is byte-identical before and after,
  measured over the whole test suite and 811 findings on 48 OPAL modules, so **no finding changes**. Freya lists
  them under the rules. Third of four slices after #848 and #853. Merged into all four lines.
- **Structure advisories accept the PHPUnit configuration where the test run looks, and an admin-only
  `views/`** (#856). The structure standards checked `phpunit.xml` at the module root only, while the test runner
  accepts it in four places; 33 of 48 OPAL modules keep it in `tests/` and got two advisories telling them to create
  a file they have. Admin-only modules got a third for a missing `views/twig/`. The accepted locations are now one
  list in the standards data, read by the standards and the runner alike; `views/` accepts either template folder.
  The JSON report gives structure findings their rule's severity instead of a fixed ERROR. Measured: 69 wrong
  advisories gone, nothing added. Merged into all four lines.

- **The finding texts of the coverage, complexity and suppression governance checks are rule data** (#853). Eleven
  texts, from "Coverage x% is below threshold y%" to "Net-new suppression … is not in the accepted baseline", were
  composed in the check classes; they now sit in the data of their twelve rules and are read through one reader.
  Every constructed text is byte-identical before and after, so **no finding changes**. Freya lists them under the
  rules. Second of four slices after #848. Merged into all four lines.

- **The finding texts of 12 PHPStan rules are rule data** (#848). Sixteen texts that name what a finding is
  about, from "extends X directly" to "iterates over a constructor-injected iterable without an activation call",
  were composed in the detector classes; Freya could not show them and the rule-data drift test could not guard
  them. They now sit in the rules' data and are read from there; every rendered text is byte-identical before and
  after, so **no finding changes**. Freya lists them under the rules. Three more areas follow the same way.
  Merged into all four lines.

- **The HTML report no longer adds its own hints to failing tests** (#843). A JSON of 21 message patterns gave each
  failing test a hint in HTML only; measured, nine of ten failures got the default, which told the reader to run
  PHPUnit directly outside the gate, and one hint recommended loosening an assertion, against the test rule's own
  advice. The hints are gone; a failing test shows its message and the advice of `tests.failure`, the same in every
  output. **No finding changes.** Merged into all four lines.

- **Advice for 15 native PHPStan and PHP_CodeSniffer checks** (#838). Since the reports stopped inventing hints
  (#833), a native finding such as `return.type` or a too-long line showed no advice anywhere. The 14 PHPStan core
  identifiers and the PSR-12 line-length check that our runs produce are now catalogue rules with a description of
  what the tool reports and the quality tools' own fix advice, written fresh against the tool documentation; the old
  texts were not reused, six of them had steered toward suppression. The advice appears in every output and in
  Freya (293 rules). Gate behaviour is unchanged. Merged into all four lines.

- **The HTML and Markdown reports no longer invent fix hints** (#833). A finding whose rule had no advice got a hint
  from a 48-key table in the code, matched by substring, and a generic "review OXID best practices" paragraph when
  nothing matched; measured, 31 keys matched nothing any more and every producing rule already carries its own
  advice, so every native PHPStan or PHPCS finding showed boilerplate in HTML and Markdown and nothing elsewhere.
  Now every output shows the same: the rule's advice, or none, and the HTML item leaves out its empty "How to fix"
  line. The 17 native checks that had a useful text get it back as rule data in a follow-up. **No finding changes.**
  Merged into all four lines.

- **Rule `structure.blockNaming` is removed** (#828). It said certification requires the `block` names in
  metadata.php to carry the module id, but its check was dropped in 2.4.0 as a false positive, and the rule stayed
  in the catalogue without a producer: the `block` field names the parent template's block, which a module cannot
  rename, and the Twig lines ignore the key. Freya now serves 278 rules; the 7.4 parity list records the removal with
  its reason. Measured on 48 OPAL modules: no `blocks[]` entries, no references, **no finding changes**. Merged
  into all four lines.

- **PHPStan resolves PHP_CodeSniffer's classes in a module that ships sniffs** (#823). PHP_CodeSniffer declares no
  Composer autoload, so the gate's PHPStan run reported a sniff's `Sniff` interface as unknown, non-ignorably. The
  gate's bootstrap now loads PHP_CodeSniffer's classes by namespace when the shop installs it, for any module. On
  the way, four typing slips in the quality tools' own rule classes were cleaned up; none changed a finding. The
  measurement behind it: the quality tools' own sniff and rule directories were excluded from their own PHPStan
  scope, and lifting that shows 120 findings, 111 after this change; the remaining ones are queued and the
  exclusion falls when they are gone. **No finding changes.** Merged into all four lines.

- **The suppression inventory records `phpcs:ignoreFile`** (#818). A `phpcs:ignoreFile` makes PHP_CodeSniffer drop
  every finding of its file, yet no inventory pattern recorded it, so a whole-file suppression appeared in no report
  and no governance saw it. It is now a file-wide record in the `phpcs:ignore` channel, over-broad like a
  `phpcs:disable` block. The legacy `@codingStandardsIgnore*` forms stay unrecorded: PHP_CodeSniffer 4 no longer
  reads them. Measured on 48 OPAL modules: no occurrence today, **no finding changes**. Merged into all four lines.

- **A `phpcs:ignore` / `phpcs:disable` record targets the codes on the marker's own line** (#813). The pattern
  read past the end of the line, so a bare marker took the next line of code as its target rule, and a marker in a
  block comment or with a note kept the `*/` or the note in it; the target is part of the baseline fingerprint, so
  such a record changed whenever the following line moved. The target is now the code list as PHP_CodeSniffer reads
  it, and a phpcs marker counts only where PHP_CodeSniffer acts on it: at the start of its comment text, in any case,
  `@phpcs:` included. Measured on 48 OPAL modules: two records change their target to none, nothing else moves.
  Merged into all four lines.

- **The suppression inventory records markers after code, and a `phpcs:disable` by its reach** (#808). A marker
  that followed code on its line (`$x; // @phpstan-ignore-line`, `…; // phpcs:ignore`) was taken for text inside a
  string and never recorded, in every module. Markers are now judged by the PHP tokenizer: inside a comment they
  count, inside a string they do not. A `phpcs:disable` without `phpcs:enable` is recorded as file-wide, with a later
  enable as a block; record scopes are unchanged, so existing baselines keep matching. Measured on 48 OPAL modules:
  8 trailing markers in 2 modules are now recorded, nothing lost, no gate state changes. Merged into all four lines.

- **The temp cache directories belong to the user running the check** (#803). PHPUnit's and PHPStan's cache
  directories below the system temp directory were only named, never owned: an existing directory was used as
  found, whoever owned it, so another local user could seed a coverage or PHPStan cache that decides findings; when
  PHPUnit could not create its directory it silently wrote into the module's own cache directory (measured: 148
  files). The tools now create these directories themselves with mode 0700 and accept an existing one only when it
  is owned by the running user and not writable by others; otherwise the test, coverage or PHPStan check reports
  `runtime.unsafeTempDirectory` and that tool does not run. Measured on OXID 7.6 with a foreign-owned directory:
  exit 0 before, the finding and exit 1 after, nothing written. **No finding changes on a clean host.** Merged into
  all four lines.

- **The quality tools' own tests are cleaned up, and the core-import case is pinned by a test** (#797, #798).
  PHPStan, run over the quality tools' own tests, found 28 defects: wrong constructor calls, `glob()` loops
  without a guard against `false`, dead code after `fail()`, and unused helpers. All 28 are fixed. None of them
  could stop a test from failing, and tests/ stays outside the gate's PHPStan scope. A new Integration test pins
  that services from the core's imported bootstrap files count as core services, with their public/private
  visibility (#792). **No finding changes.** Merged into all four lines.

- **Container locator rules see the shop core's services, in the container's own order** (#792). The core's DI
  files were never read, so every core service fetched through the locator in a place without constructor injection
  got the "not statically analysed" advisory, and a private one went unreported. The rules now read the installed
  editions' DI files (EE, PE and CE, as rule data) and look a service up in the order the shop's container uses:
  declared components, the module itself, declared modules, then the core. A private core service is the blocking
  error it is at runtime. Measured on three OPAL modules: four advisories become zero. Merged into all four lines.

- **Container locator rules see the services of declared modules and components** (#787). The two rules on
  `ContainerFacade::get()` / `ContainerFactory::getInstance()->getContainer()->get()` looked up a service id only
  in the module's own `services.yaml`, so a public service of a declared module or oxideshop-component package got
  the "not statically analysed" advisory and a private one went unreported. They now look in the module, then in the
  modules and components it declares; a declared package's private service is the blocking error it is at runtime,
  with advice to inject it through one of the module's own services. Measured on an OPAL module with a declared
  component: five advisories become two (the remaining two are core services). On 7.3–7.5 only `services.yaml`
  counts. Merged into all four lines.

- **Admin template references are looked up in the module their id names** (#782). The admin structure check
  stripped the `@<module id>/` prefix of a controller's template reference and looked in the module's own
  `views/twig`, so a declared module's template was reported missing and an undeclared module's reference passed
  whenever the module itself shipped a file of that name. References now resolve in the module the id names, through
  the same namespace mapping the template linter uses; an undeclared id is a finding, and the finding texts live in rule
  data. Measured on four OPAL modules with their declared dependencies: identical results. Merged into all four lines.

- **Declared modules' templates resolve by module id; undeclared ones no longer pass** (#776, #777). The template
  linter registers a module and its declared modules as Twig namespaces under their module ids, as the shop's own
  template resolver does, so a `{% use '@<module id>/…' %}` of a declared module (or of the module itself) compiles,
  while the former scan of every module directory without a namespace, which let an undeclared module's template pass,
  is gone. `template.twigParentMissing` now says what is checked: a `{% use %}` name, since `{% extends %}` and
  `{% include %}` resolve at render time. Measured on four OPAL modules with their declared dependencies installed:
  identical results. The repository's own CLAUDE.md gate command names a placeholder service and shop root instead of
  one machine's path. Merged into all four lines.

- **Structure recommendations are advisory findings in every output** (#771). The module-structure check built
  recommendations (a missing LICENSE, tests/ or phpunit.xml, a services.yaml without the Extension exclusion, a
  migration without a README note, a table without the module prefix, an admin menu pattern) as a separate list that
  only JSON, Markdown and HTML showed, while five catalogue rules for exactly these cases were never reported. They
  are now advisory findings of seven rules (two new, five that came alive) with their texts in rule data, rendered in
  TOON, JSON, Markdown, Human and HTML alike and documented by Freya as advisory. The JSON `recommendations` key stays
  empty for one release. Score and gate unchanged. Merged into all four lines.

- **One fix text per finding, from the catalogue** (#766). The docblock, template and structure checks built
  per-finding fix texts in PHP that no output ever showed; the rule catalogue's text is now the one fix in TOON,
  JSON, Markdown, Human and HTML. Four catalogue texts were widened for the cases those PHP texts named
  (`{@inheritDoc}` versus a native type, a missing class docblock, a broken symlink, a Twig extension that is not
  available), the deprecated-path messages carry their migration step, and the dead HTML advice branch is gone.
  Measured on a real module: the same 545 findings before and after, 57 of them now carrying their catalogue fix in
  every output. **No finding count changes.** Merged into all four lines.

- **A file-reading test leaves the Unit suite** (#761). `LineTargetTest` wrote temporary composer.json files and
  so broke the rule that unit tests do no filesystem IO; it now runs in the Integration suite, and the value path
  stays covered in the Unit suite. The sweep behind it found the same pattern in 71 more unit test files, queued
  as its own item. **No finding changes.** Merged into all four lines.

- **Texts claim no OXID version they cannot back; the MANUAL lists match the code** (#755, #756). Twelve places
  said "OXID 7.3+", "7.4+ components", "OXID 7.6 best practices" or that "OXID 7.5 ships" the JWT and session
  authentication components: the structure validator's docblocks and user-facing messages, the `--structure` option
  text, README, MANUAL and AI-RULES. None of it was verifiable (the component installer exists since 6.5, the
  `assets/` link since 7.0, the two components are standalone packages no compilation requires). The texts now
  name what the tools check: the module structure standards, `assets/` linked into `out/`, and the packages as
  optional. The MANUAL's `next_actions` list gains `repeat_test_order` and `upgrade_php`, and its PHPStan "Ignored
  Patterns" name the real path globs of the gate's neon, pinned by an integration test. **No finding changes.**
  Merged into all four lines.

- **The package ships its LICENSE file** (#750). README and MANUAL pointed at a `LICENSE` file that did not exist.
  The file now carries the OXID-OPAL modules' license text ("OXID Modul & Komponenten-Lizenz 2023", byte-identical
  to the modules' copy); composer.json keeps `"license": "proprietary"`. The quality tools' module-structure score on
  their own package goes from 82 to 100, since LICENSE was the one recommended file missing. **No finding changes**
  for modules. Merged into all four lines.

- **Admin-theme and storefront header overrides are no longer cross-context findings** (#745).
  `template.crossContextInheritance` accepted any directory before an admin base name, so a storefront override
  extending apex's `layout/header.html.twig` was a blocking "Extends admin base layout" finding, and the rule also
  fired on overrides of the admin theme (`views/twig/extensions/themes/admin_twig/`), where extending an admin base
  is correct. The admin bases now match only as the admin theme names them (its root and `popups/headitem`), and
  admin_twig overrides count as admin templates for this rule and for `template.adminFunctionInStorefront`. Rule
  data only. Measured over 137 OXID-OPAL clones: the one live finding (an apex `layout/header.html.twig` override)
  is gone, nothing else changes. Merged into all four lines.

- **No tracker ticket ids in shipped texts** (#738). Eight places, among them one rule description shown by Freya,
  named a real Jira ticket. Each now names the pattern it means (a consumer that iterates tagged providers from
  other modules without an activation or availability gate), and an Integration test keeps tracker-id shapes out
  of `src/`, `config/`, `tests/` and `stubs/`. Required by the engineering handbook's rule R0004. **No finding
  changes.** Merged into all four lines.

- **The texts say what the gate analyses in `tests/`** (#733). The gate's PHPStan run never analyses a module's
  `tests/`, but the `phpstan.undeclaredPackageClass` description promised extra credit "in tests/" and
  `phpstan-oxid.neon` carried three ignore entries for test code that could never apply. The description now says
  the gate does not analyse `tests/`, the unreachable entries are gone, and MANUAL names the gate tools that skip
  `tests/` (PHPStan, PHPMD, PHP_CodeSniffer, docblock check). Measured on QT and five OPAL modules: analysing tests
  would add 4437 findings and block every module, with the test-scope credit changing none of them. **No finding
  changes.** Merged into all four lines.

- **Admin menu labels count every admin translation of the shop and of required modules** (#571). The admin
  translation checks credited only the module's own files and the shop's `lang.php`. They now also credit **every
  admin language file the shop core loads** (`help_lang.php`, `translit_lang.php`, theme `*_options.php`; the list is
  rule data, `cust_lang.php` excluded) and **the admin language files and `module_options.php` of the modules a
  module requires**; installed modules it does not require never count. **Key-shaped OXMENU labels are now checked**
  like MAINMENU/SUBMENU (pattern in rule data); literal labels stay unchecked. Measured over 91 OPAL admin menus on
  OXID 7.6 and 7.3: no finding changes. Merged into all four lines.

- **Unknown Twig functions and filters are findings** (#724). The template check accepted any function or filter
  name, so a misspelled or unavailable one passed and only failed when the shop rendered the template; the three
  `template.twigUnknown*` rules could never fire. QT now judges every function and filter name a template uses
  against what is really available: **Twig itself, the shop's installed components (e.g. twig-component), the
  module's own extensions and those of the modules it requires**. Any other name is a `template.twigUnknownFunction`
  / `template.twigUnknownFilter` finding on its first line; an unknown tag is `template.twigUnknownTag`; severity
  comes from the rule data (blocking). Measured on five OPAL modules: exactly one real finding, no false ones.
  Merged into all four lines.

- **Freya names only the migrate command of the line's own OXID minors** (#623). The state-changing commands and
  the quick reference listed every minor from 7.2 to 7.6, so an agent on a 7.3 shop also read the 7.6-only
  `oe:database:migrate`. The commands are now data (`config/oxid-commands.php`), and Freya fills in the line's
  target and prior minor: **7.6 line: `oe-console oe:database:migrate` for 7.6 plus the script for 7.5; 7.5, 7.4
  and 7.3 lines: `oe-eshop-db_migrate migrations:migrate` for both minors**. A new EE line says every `oe-console`
  command takes `--shop-id=<id>`. QT's own suite fails when a line's minors have no data, and checks every listed
  command against the installed shop. Checked live in Freya on 7.6, 7.5, 7.4 EE and 7.3. Merged into all four lines.

- **Tool time limits come from one data file, fit measured run times, and can be raised per run** (#715). PHPUnit
  test and coverage runs were capped at 600 s; QT's own coverage run already took 472 s on 7.4 / PHP 8.2, so a
  slower CI runner would have stopped an honest suite. Limits are now **1200 s for tests, 1440 s with coverage and
  120 s for each PHPStan, PHPMD, PHPCS and PHPCBF run** (PHPStan and PHPMD were 300 s; values = measured maximum
  on five shops × 3), all read from `config/tool-limits.php`. Each one can be replaced for a single run with
  **`QUALITYTOOLS_LIMIT_PHPSTAN`, `_PHPMD`, `_PHPCS`, `_PHPCBF`, `_PHPUNIT_TEST` or `_PHPUNIT_COVERAGE`** (whole
  seconds, 1 to 3600; anything else stops the run before it starts). Every override, `--phpunit-timeout` included,
  is named in all reports and now also under the console header. Merged into all four lines.

- **A failing test run says when the module is not active in the shop** (#710). `--test` or `--coverage` against a
  module that is installed but not activated failed by the hundreds (missing services, tables and templates) without
  a word about the cause. The run now reads the module's activation state from the shop and, when tests failed or
  errored, adds one advisory `tests.moduleInactive` (does not block the gate) and puts **`activate_module` first in
  the next actions: `oe:module:activate <id>`, then the shop's state-changing commands** (Freya workflow). The run,
  its counts, failure rows and exit code are unchanged; a green run, an active module or an unreadable state gets no
  notice. Verified with opaltrackingstatus installed but not activated on OXID 7.6 and 7.3: the same 273 failure
  rows plus exactly one notice. Merged into all four lines.

- **PHPStan reports classes from packages the module does not declare** (#705). A module could use a class of any
  installed package and pass, although a shop without that package fails. Every class use (new, extends, types,
  static calls, instanceof, catch, attributes, PHPDoc types) must now come from the module itself, a package its
  composer.json `require` names (transitively, as installed), the shop or PHP. The shop is the installed compilation
  metapackage (else the core packages) without the modules a compilation ships, so another module counts only when
  it is required; in tests/ the `require-dev` tree and the test frameworks count too. Any other package is the
  blocking `phpstan.undeclaredPackageClass`; **fix: add the package to `require`**. A package named only under
  `suggest` is the advisory `phpstan.optionalPackageClass` (guard its use); `phpstan.packageScopeFallback` notes a run
  that could not read composer.json or installed.json. Eight OPAL modules on OXID 7.6 and 7.5 (and the two that
  suggest packages on 7.5 EE): no new blocking finding, no exit code changed; the advisory appeared for opalordermerge
  (19, graphqlite attributes) and, where visualcms is installed, opalaiimagemap (15). **Composer:** QT now requires
  `oxid-esales/twig-component`, which its template check uses (7.6 `^2.8`, 7.5 `^2.7`, 7.4 `^2.6`, 7.3 `^2.5`; every
  compilation already installs it). Merged into all four lines.

- **Changed: PHPStan resolves extension parents only from the module and its declared dependencies** (#700). The
  PHPStan bootstrap read the metadata.php of every installed module, so a module that uses another
  module's extension class passed even when it does not require that module, and would break in a shop without it.
  The gate now reads the metadata.php of the analysed module and of the modules its composer.json `require` names
  (transitively, as installed). Calling an inherited method of an undeclared module's extension class is now
  PHPStan's `method.notFound`; **fix: add that module to `require`**. When the dependencies cannot be determined
  (no readable composer.json, a resolver error), every installed module counts as before and the advisory
  `phpstan.moduleScopeFallback` says so. Six OPAL modules on OXID 7.6 gave identical results before and after.
  Merged into all four lines.

- **PHPMD's missing-import finding is no longer reported for a global class used outside a namespace** (#693).
  In a file without a namespace (or in the bracketed global `namespace { }` block), `new ReflectionClass(…)` or
  `new \ArrayObject()` was reported as `phpmd.missingImport`, although no `use` statement can import a global class
  there. Such references are now left out; a namespaced class in that code (`\Vendor\Name`) is still reported,
  because `use Vendor\Name;` fixes it. The case is named in the rule data (`not_reported` on `phpmd.missingImport`),
  so Freya's `get_rule` shows it. PHPMD runs on the 7.4 and 7.3 lines below PHP 8.3. QT's own gate is now at
  0 findings there as well. Merged into all four lines.

- **Test order per run: `--test-order=random|reverse`, `--test-order-seed=N`, and the order in every output** (#688).
  The gate's PHPUnit runs always used the order of the module's phpunit.xml, so a test that depends on what an
  earlier test left behind stayed hidden, and a random order set in phpunit.xml could not be repeated because its
  seed was never reported. `--test-order` asks PHPUnit for another order for one run; `--test-order-seed` (only with
  `random`, 0 to 2147483647) fixes the seed, and without it a seed is generated. Anything else is rejected before any
  check runs. Every output now names the order the run had, where it came from, the seed of a random order
  (`unknown` when PHPUnit printed none) and the options that repeat it; a failed or stopped run gets the next action
  `repeat_test_order`. Orders, seed bounds and wording are data in the new `config/test-order.php`.
  Merged into all four lines.

- **PHPUnit time limit per run: `--phpunit-timeout`, and 600 s for a test-only run** (#681). A test-only run
  (`--test`) was stopped after a fixed 300 s, while the run with coverage got 600 s, so a suite that fit the
  coverage run could fail `--test`. Both runs now get 600 s by default, and `--phpunit-timeout=<seconds>` (a whole
  number from 1 to 3600) gives one run its own limit; anything else is rejected before any check runs. Defaults and
  bounds are data in the new `config/tool-limits.php`. `tests.timeout` and `coverage.timeout` now say that the
  stopped tests did not run their tearDown, so the shop may keep what they wrote, and name the option.
  Merged into all four lines.

- **AI-RULES: the attribute-route note in `module-lifecycle` is corrected** (#678). The "OXID 7.5 note" said that
  `oe:module:activate` fires `ClearShopCacheEvent` (only `oe:cache:clear` does), was served on the 7.3 and 7.4 lines
  as well, and listed a second, shorter lifecycle sequence. The note now covers OXID 7.5 and later only and points to
  `state-changing-commands`, which stays the one list of lifecycle commands. Text only; **no finding changes**.
  Merged into all four lines.

- **`template.unknownBlockOverride` also checks `default` and `admin_twig` theme overrides** (#673). Only overrides
  for apex were checked. twig-component uses a module's `extensions/themes/default/<path>` override for the active
  theme when the module has none for it; such an override is now checked against apex, unless your module also
  overrides that template for apex. An `extensions/themes/admin_twig/<path>` override is checked against the admin
  theme. **Visible to module CIs:** an unknown block in such an override is now a finding. In the twelve QTdev shops
  no existing `default` or `admin_twig` override has an unknown block. Merged into all four lines.

- **A mistyped module id no longer runs other modules' metadata.php** (#668). For an id that is not in the shop's
  module configuration (a typo, a module that is not installed), `qualitytools:check <id>` searched for the module by
  including metadata.php files from vendor/ and the shop's modules/, so their code ran inside the gate; a file that
  calls `exit` ended the run silently. The id is now read from each file without running it. **Behaviour change:** a
  module whose `'id'` is a constant (e.g. `Module::MODULE_ID`) and that is not in the module configuration is no
  longer found by its id; pass its directory instead: `qualitytools:check vendor/<vendor>/<package>`. Merged into
  all four lines.

- **`docblock.invalid` names the fix for an unresolved OXID parent everywhere** (#663). For an extension whose
  `*_parent` resolves through neither `@mixin` nor metadata.php `extend`, the check carried a fix text of its own
  that no report printed. The rule's fix suggestion now covers that case, so Freya, TOON, JSON, HTML, Markdown and
  the console all show the same text. **No finding changes.** Merged into all four lines.

- **`template.unknownBlockOverride` counts only the modules your module requires** (#570). A block that your
  template override adds was treated as known whenever any other installed module's override of the same parent
  template defined it, so an unrelated module could hide a real unknown block. Now only the modules your module
  requires in its composer.json (transitively, as installed) count, and the parent of a module-template extension
  is looked up only among them. **Behaviour change:** your module CI can report an unknown block that an
  unrelated installed module used to hide; and a module-template extension whose parent module you do not require
  is no longer checked (require it to get the check). Merged into all four lines.

- **Freya's `get_rule` shows the texts a finding of the rule carries** (#654). It used to show the rule's title,
  description and fix suggestion, but not what a finding of it will say. Now a "Finding texts" section lists each
  text verbatim with its key (e.g. `messages.unknown_id`, `advice.di_less`) and its placeholder style, so you or
  your AI can match a finding to the rule's case. Fix texts that a finding prints (e.g. `runtime.phpVersionTooOld`)
  are listed too. `list_rules` is unchanged. **No finding changes.** Merged into all four lines.

- **A tool stopped at its time limit says so; a PHPCS run without a report no longer passes** (#649). The PHPCS
  check never read PHPCS's exit code, so a run that crashed, could not load its ruleset or hit its 120 s limit passed
  with 0 findings. A PHPUnit run that hit its limit was reported as the crash "Failed to load test suites", and PHPStan
  and PHPMD as "exited with code -1". Now they report `tests.timeout`, `coverage.timeout`, `phpstan.timeout`,
  `phpmd.timeout` and `phpcs.timeout`, each with the seconds the tool ran, its limit and its last output. A PHPCS run
  counts only with exit 0-3 and a JSON report, a PHPCBF run only with exit 0-7. Anything else is the blocking
  `phpcs.runFailed`. **Behaviour change:** a module CI whose PHPCS run used to pass without a report now fails with
  that finding; read the quoted output. The limits are unchanged (PHPUnit 300 s, or 600 s with coverage; PHPStan and
  PHPMD 300 s; PHPCS and PHPCBF 120 s). Merged into all four lines.

- **The gate writes nothing into your module any more** (#644). QT started PHPUnit without a cache option, so every
  `--test`, `--coverage` or full gate run left `.phpunit.cache/` (or `.phpunit.result.cache`) in the checked module,
  where it could be committed by accident; and with `executionOrder="defects"` the cached results reordered the next
  gate run by earlier failures, so the order depended on the run history. Both PHPUnit runs now pass
  `--do-not-cache-result` and keep the coverage cache below the system temp directory. You may delete the
  `.phpunit.cache/` or `.phpunit.result.cache` that earlier QT runs left in your module (your own PHPUnit runs may
  still write there, as your phpunit.xml says). **No finding changes.** Merged into all four lines.

- **`vendor/bin/freya --setup` writes the shop's own URL** (#639). It took the port from a docker-compose service
  named `web` and otherwise wrote `http://localhost:80/…`, so on OXID docker-eshop-sdk shops (own `PORT_SHOP`) and
  whenever it ran inside the container, the written `freya` entry did not answer. It now uses `--url=http://<host>:<port>/`
  or the shop's `sShopURL` from `source/config.inc.php` (only that value is read), says where the URL came from and
  whether Freya answered, and prints the matching `claude mcp add --scope user --transport http freya …` command.
  **Behaviour changes:** without `--url` or a usable `sShopURL` it writes nothing and exits 1 (it used to write :80 and
  exit 0); an existing `.mcp.json` that is no JSON object (e.g. a trailing comma) is left as it is with exit 1 (it used
  to be rewritten with only the `freya` entry, losing every other server); an unreadable or unwritable `.mcp.json` ends
  with a message and exit 1. If your `.mcp.json` points at `:80` and Freya does not connect, run
  `vendor/bin/freya --setup` again (or pass `--url`). Merged into all four lines.

- **Freya runs over HTTP only** (#583). The old stdio server had refused to serve since 2.9.0, so a `freya` entry that
  `vendor/bin/freya --setup --stdio` wrote into `.mcp.json` pointed at a server that exited at once. That server and
  the `--stdio` option are removed; `--setup --stdio` now stops with a message instead of writing a dead entry. If your
  `.mcp.json` still has a stdio `freya` entry, run `vendor/bin/freya --setup` in the shop root, or register
  `http://localhost:<port>/index.php?cl=freya_mcp` with `claude mcp add --transport http freya …`. Merged into
  all four lines.

- **The module list shows the right version** (#630). `metadata.php` still named 3.2.0 after the 4.0.0 release, so
  every shop's module list showed 3.2.0. It now names 4.0.0; a shop shows it after `composer update` of the quality
  tools (or `oe:module:install`). A new test fails the gate when the version, the CHANGELOG and the composer alias
  disagree. Merged into all four lines.

- **Advice and documentation corrected, verdicts unchanged** (DOC-1, #623). The advice of `tests.riskyTest` now says
  to assert what the test exercises (`#[DoesNotPerformAssertions]` only for a test whose whole purpose is that the code
  does not throw), and `coverage.belowThreshold` says to test the behaviour of the uncovered code, not to chase the
  number. `structure.missingModuleOptionsKeys` no longer calls a key "missing" when another admin language file holds
  it, and `runtime.phpVersionTooOld` says which OXID version the shop is checked for. AI agents now get the permitted
  migrate command for OXID 7.2–7.5 shops from the `state-changing-commands` workflow, and what view generation does
  to module columns. The MANUAL's PHPStan level, JSON report example and PHPMD comparisons are corrected. Merged into
  all four lines.

- **The cross-module rules measure the module and the modules it declares** (#569). `structure.crossModuleTagActivationGate`
  and `phpstan.providerContractMissingAvailabilityGate` used to count a tag as consumed across modules whenever any module
  installed next to the analysed one consumed it, declared or not. Now only the module and the OXID modules it declares
  in its composer.json `require` (directly or transitively) count, wherever they are installed. A finding can therefore
  **vanish** (only an undeclared neighbour consumed the tag) or **appear** (a declared module installed in another vendor
  directory consumes it). Both rules' finding texts now come from the rule data; the wording is unchanged. Measured
  with the OPAL modules on OXID 7.6, 7.5, 7.4 and 7.3 (on 7.5 opalerpodoo with its declared modules; migrations run):
  **no finding changes**, since none of them has a cross-module finding before or after. Merged into all four lines.

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
