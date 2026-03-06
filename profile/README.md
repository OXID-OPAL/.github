# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## 🔧 Quality Tools v1.4.1 — Bug Fixes & Output Parity

> `b-7.4.x` &bull; 6 March 2026

Patch release fixing **output discrepancies between AI and human mode**, **version-aware documentation links**, and **bootstrap warnings**.

### Fixed

```
 BUG FIXES              What changed
 ───────────────────────────────────────────────────────────────────────
 Version-aware URLs      fix_suggestion links now point to the detected
                         OXID version (was: hardcoded 7.0 / 7.3)
 AI mode errors          --caller=ai now detects tool execution errors
                         (non-zero exit + 0 violations → status: failed)
 Exit code               Top-level exit code is non-zero when any tool
                         reports failed status
 Bootstrap warnings      INSTALLATION_ROOT_PATH redefinition + session
                         ini_set warnings eliminated
 Human mode output       Docblock and template issues now shown inline
                         (was: hidden behind -v verbose flag)
```

### Changed

```
 BOOTSTRAP RULE          Enforces correct OXID bootstrap loading pattern
 ───────────────────────────────────────────────────────────────────────
 Guard required          if (!defined('INSTALLATION_ROOT_PATH'))
                         must wrap the require
 No pre-definition       define('INSTALLATION_ROOT_PATH', ...) before
                         loading source/bootstrap.php is now flagged
```

---

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
