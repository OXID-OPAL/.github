# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## 🆕 Quality Tools v1.4.0 — Admin Validation & Smarter Reporting

> `b-7.4.x` &bull; 2 March 2026

v1.4.0 adds a **full admin structure validator** (8 new checks) and makes reporting **sharper and more actionable**. The template linter now understands all OXID custom Twig tags natively.

### Admin Structure Validator *(new)*

```
 ADMIN CHECKS            Validates admin controller setup end-to-end
 ───────────────────────────────────────────────────────────────────────
 Menu.xml wiring         cl= and list= point to registered controllers
 Three-controller        Frameset / List / Main pattern completeness
 Template format         No .html.twig extension, @module-id/ prefix required,
                         no path separators after prefix
 Template placement      Admin templates must live in views/twig/
 Controller keys         snake_case with module prefix enforced
 Dual registration       Flags controllers in BOTH metadata.php AND services.yaml
 Language files           de/ and en/ lang files with required translation keys
```

### Template Linting

```
 OXID TWIG TAGS          Registered on the linter environment
 ───────────────────────────────────────────────────────────────────────
 Token parsers           capture, ifcontent, include_content,
                         include_dynamic, hasrights
 Functions               insert_tracker
                         → No more false-positive "unknown tag" errors
```

### Reporting Improvements

```
 SHARPER OUTPUT          More context, less guesswork
 ───────────────────────────────────────────────────────────────────────
 Per-file infra leaks    tests/Unit/FooTest.php — uses Registry (infrastructure)
                         (was: generic "Unit tests contain infrastructure code")
 errors in test summary  TOON now shows passed / failed / errors / skipped
 StaticAccess hints      Removable (→ inject ConfigInterface) vs justified
                         (→ suppress or use ContainerFacade)
 YAML snippets           services.yaml recommendation includes ready-to-use
                         exclude config for Extension/ and Entity/
 Exit code               Module structure ERRORs now produce exit code 1
```

### Other Changes

- Mock-context lines (`createMock`, `getMockBuilder`) excluded from infrastructure scanning
- Assertion-referencing lines excluded from infrastructure scanning
- `QueryBuilderFactory` pattern no longer matches `QueryBuilderFactoryInterface`
- Blanket `@SuppressWarnings(PHPMD)` narrowed to specific rules throughout

---

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
