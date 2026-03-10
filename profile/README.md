# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## 🔮 Quality Tools v2.1.0 — Freya MCP Server

> `b-7.4.x` &bull; 10 March 2026

AI agents can now **browse the full quality rule catalog before writing code**,
eliminating trial-and-error cycles with the quality gate.

### Freya — Rule Browsing for AI Agents

**Freya** is a read-only [MCP](https://modelcontextprotocol.io/) server that
exposes all quality rules over JSON-RPC 2.0 stdio transport. Two tools:

| Tool | Description |
|------|-------------|
| `list_rules` | Browse all rules with ID, severity, title. Filter by tool (`phpstan`, `phpmd`, `phpcs`, `structure`, `templates`, etc.) |
| `get_rule` | Full details: what the rule detects, what to do instead, severity, documentation link |

### Setup (one command)

```bash
# Native PHP
vendor/bin/freya --setup

# Docker (auto-detected)
docker compose exec web php vendor/oxid-quality-tools/standards/bin/freya --setup
```

Writes `.mcp.json` to the shop root. Auto-detects Docker environments.
Restart Claude Code to connect.

### Workflow

1. `list_rules` → see what rules apply to your area
2. `get_rule` → understand detection pattern + fix suggestion
3. Write compliant code on the first attempt
4. `qualitytools:check` → verify

---

## ⚠️ Quality Tools v2.0.0 — BREAKING: Architecture Rebuild

> `b-7.4.x` &bull; 10 March 2026 &bull; **BREAKING CHANGE**

> [!CAUTION]
> **v2.0 is a complete internal rebuild.** All v1.4.x CLI options and detection
> rules are preserved, but the internal architecture is new. If you depend on
> internal classes (parsers, output generators, rule classes), your code **will
> break**. The previous v1.4.x codebase is archived on branch `b-7.4.x-v1.4`.

### What breaks

```
 REMOVED CLASS                          REPLACEMENT
 ──────────────────────────────────────────────────────────────────────────
 AiOutputGenerator                      AiConsolePlugin (OutputPluginInterface)
 OutputPresenter                        HumanConsolePlugin (OutputPluginInterface)
 OutputHelper / OutputHelperInterface    Output plugins handle this internally
 AiActionBuilder / AiActionBuilderInterface    Removed — no longer needed
 14 individual PHPStan rule classes      6 generic ConfigDriven*Rule classes
 8 individual PHPMD rule classes         1 generic ConfigDrivenPhpMdRule class
 2 individual PHPCS sniff classes        2 generic ConfigDriven*Sniff classes
```

### What stays the same

- All CLI options (`--phpstan`, `--phpmd`, `--phpcs`, `--test`, `--coverage`, etc.)
- All 24 detection rules (same violations, same rule IDs, same fix suggestions)
- TOON output format (`--caller=ai`)
- Report formats (HTML, JSON, Markdown)
- Exit codes and quality gate thresholds

### What's new

```
 FEATURE                         DESCRIPTION
 ──────────────────────────────────────────────────────────────────────────
 Data-driven rule engine          All rules defined in config/rules.php —
                                  add a rule = one config entry, zero PHP
 Crash diagnostics                Actionable PHPUnit crash messages instead
                                  of generic "PHPUnit crashed" (5 patterns)
 Output plugin architecture       5 plugins: Human, AI, HTML, JSON, Markdown
 Check/Result pattern             Typed results per check, QualityOrchestrator
 --test-structure flag             Independent exclusive purpose flag
```

### Migration

If you only use the CLI (`qualitytools:check`), **no changes needed**.

If you import internal classes, check the
[CHANGELOG](https://github.com/OXID-eSales/quality-tools-module/blob/b-7.4.x/CHANGELOG.md)
for the full list of removed and replaced classes.

**Archived branch:** `b-7.4.x-v1.4` preserves the v1.4.x codebase for reference.

---

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
