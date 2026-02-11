# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## Announcements

### :rocket: Quality Tools v1.2.0 — Major Rule Expansion (February 2026)

> **11 new detection capabilities** added to `oxid-quality-tools` on branch `b-7.4.x`.
> These rules catch real-world anti-patterns found across OPAL modules during validation.

#### New PHPStan Rules
| Rule | What it catches |
|------|----------------|
| **RegistryInExtensionsRule** | `Registry::` calls in Extension classes — use `ContainerFacade::get()` |
| **QueryBuilderFromMisuseRule** | String concatenation in `->from()` (safe `getSQL()` subquery pattern excluded) |
| **DatabaseValueTypeComparisonRule** | Strict `===` comparison with `fetchOne()`/`fetchColumn()` results |

#### New PHPMD Rules
| Rule | What it catches |
|------|----------------|
| **MagicPropertyAccess** | Legacy `->oxtable__oxfield->value` — use `getFieldData()` |
| **AddslashesDetection** | `addslashes()` for SQL escaping — use parameterized queries |
| **RawSQLDetection** *(enhanced)* | Now also detects `sprintf()`-composed SQL and `from()` concatenation |

#### New Structure Validators
| Validator | What it checks |
|-----------|---------------|
| **services.yaml exclusion** | Extension classes excluded from autowiring |
| **Twig blocks metadata** | Empty `blocks` array when using path-based Twig extensions |
| **Migration documentation** | README.md documents database migrations when `migration/` exists |

#### Other Improvements
- **Template `data-testid` check** — warns when interactive elements lack test IDs
- **ExtensionMethodPrefix fix** — no longer false-positives on test classes in `\Tests\Extension\`
- **Path fixes** — `HtmlTemplateRenderer` and `ReportGenerator` use `dirname(__DIR__)` instead of hardcoded Registry paths
- **Docs updated** — README.md and MANUAL.md revised for OXID 7.4

---

<details>
<summary><b>Previous: v1.1.0 (February 2026)</b></summary>

- **Markdown report format** - Use `--report-format=markdown` for PR-friendly reports
- **Generate all formats** - Use `--report-format=all` for HTML, JSON, and Markdown
- **Fixed "Action Required" bug** - HTML reports now correctly show "All checks passed"
- **Updated documentation** - MANUAL.md and README.md fully revised
- **Fixed wrong exit code summarizing** - eliminated some false negatives
- **Fixed Suggestions** - not promoting Legacy Patterns anymore

</details>

See [quality-tools-module](https://github.com/OXID-eSales/quality-tools-module) for details.

---

## Repositories

- **quality-tools-module** - Unified quality checking for OXID modules (PHPStan, PHPMD, PHPCS, PHPUnit, coverage)
