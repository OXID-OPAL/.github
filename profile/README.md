# OXID-OPAL

Internal tools and modules for OXID eSales development.

---

## Announcements

### OXID Quality Tools v1.1.0 Released (February 2026)

New features in `oxid-quality-tools`:

- **Markdown report format** - Use `--report-format=markdown` for PR-friendly reports
- **Generate all formats** - Use `--report-format=all` for HTML, JSON, and Markdown
- **Fixed "Action Required" bug** - HTML reports now correctly show "All checks passed"
- **Updated documentation** - MANUAL.md and README.md fully revised
- **Fixed wrong exit code summarizing** - eliminated some false negatives
- **Fixed Suggestions** - not promoting Legacy Patterns anymore

See [quality-tools-module](https://github.com/OXID-eSales/quality-tools-module) for details.

---

## Repositories

- **quality-tools-module** - Unified quality checking for OXID modules (PHPStan, PHPMD, PHPCS, PHPUnit, coverage)
