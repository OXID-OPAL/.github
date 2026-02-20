# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## Quality Tools v1.3.0 — Test Intelligence & Stricter CLI

> `b-7.4.x` &bull; 20 February 2026

v1.3.0 focuses on **test quality analysis** and **CLI correctness**. The tool now understands your test structure, catches misplaced tests, and rejects flag combinations that never worked correctly.

### What's new

```
 TEST STRUCTURE          Scans your tests/ tree for structural problems
 ───────────────────────────────────────────────────────────────────────
 Infrastructure leaks    Unit tests using Registry, oxNew(), DatabaseProvider,
                         ContainerFacade, IntegrationTestCase → suggests Integration/
 Suite config check      tests/Integration/ exists but phpunit.xml has no suite for it
 Bootstrap quality       Hardcoded paths, missing INSTALLATION_ROOT_PATH guard,
                         _parent stubs hiding coverage gaps
 Suppression report      Lists @codeCoverageIgnore, @SuppressWarnings, phpcs:ignore,
                         @phpstan-ignore in src/ across all output formats
```

```
 STATIC ANALYSIS         New and enhanced rules
 ───────────────────────────────────────────────────────────────────────
 ContainerFacadeIn-      Flags ContainerFacade::get() in modules targeting ^7.0
 ModulesRule (PHPStan)   → use constructor injection via services.yaml
 DeprecatedDatabase-     Now also catches DatabaseProvider::getDb() and getMaster()
 Provider (PHPMD)        (previously only Db::getDb())
```

```
 CLI CHANGES             Breaking behavior changes
 ───────────────────────────────────────────────────────────────────────
 Exclusive purpose flags --phpstan --phpmd now throws InvalidArgumentException
                         (use --no-phpcs to skip one tool instead)
 --stop-on-error +       Now throws error (was silently ignored)
 --report-format
 Auto-implication         --coverage shows feedback that it enables --test
 TOON output             --caller=ai now emits TOON (30-60% fewer tokens than JSON)
 PHPUnit exit 255        Differentiates crash-during-run vs suite-loading-failure
```

### Fixes

- **Twig modules false positive** — `blocks` warning no longer fires for Twig-only modules
- **Report data propagation** — suppression annotations and test structure data flow through to all report formats
- **MANUAL.md** — corrected 6 factual errors, added all missing v1.3.0 documentation

---

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
