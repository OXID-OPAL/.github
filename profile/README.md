# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## 🩹 Quality Tools v2.1.1 — Bugfixes

> `b-7.4.x` &bull; 16 March 2026

Four corrections to rule definitions, path resolution, and detection coverage.

### Fixed

- **`admin.languageFiles` rule** — description and fix_suggestion now reference
  the correct path (`views/admin_twig/de/` and `views/admin_twig/en/`) and
  document all four accepted filename patterns
- **`--report-dir` / `--coverage-html` relative paths** — relative paths are
  now resolved relative to the target module directory, not the container's CWD
- **`admin.unknown` false positives** — the admin menu.xml wiring validator now
  recognizes list-only and simple admin patterns instead of enforcing the classic
  three-controller pattern
- **`phpmd.rawSqlDetection` detection gap** — now also scans class constants for
  raw SQL keywords (previously only method bodies were inspected)

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
[CHANGELOG](https://github.com/OXID-eSales/quality-tools-module/blob/b-7.4.x/CHANGELOG.md).

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

See [quality-tools-module](https://github.com/OXID-eSales/quality-tools-module) for full changelog and source.

---

## Repositories

- **quality-tools-module** — Unified quality checking for OXID modules (PHPStan, PHPMD, PHPCS, PHPUnit, coverage)
