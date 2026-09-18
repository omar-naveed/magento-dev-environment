# CLAUDE.md — magento-dev-environment

## What this repo is
A **public** portfolio project demonstrating Magento Open Source development
practices and workflow discipline. It is explicitly NOT a place to publish
employer or customer information — see the sanitization rules below before
touching git.

## Sanitization rules — check before every commit and push
This repo is meant to go public. Never commit any of the following:
- `auth.json`, `app/etc/env.php`, `.env`, or any credentials/API keys
- Database dumps, backups, or `pub/media` content from a real store
- Real customer names, emails, addresses, order data, or any PII
- The employer's name, internal domains/hostnames, internal ticket IDs,
  Slack/Jira links, or anything that identifies the company or its customers
- Anything sourced from `~/Magento Backup` (that is real employer/customer
  data and must never enter this repo)

Use Magento's official sample data (`bin/magento sampledata:deploy`) for any
demo content, product catalog, or fixtures. Before every commit, review the
diff for anything that looks like a real domain, employee name, or customer
identifier and stop if found — flag it to the user rather than committing.

## Stack
- Magento Open Source 2.4.9 (current stable as of 2026) — confirm the exact
  PHP version against Adobe's compatibility matrix for whatever Magento
  version is actually installed before assuming a PHP version
- Local environment: DDEV (preferred) or Warden
- DB inspection: TablePlus or DBeaver
- CLI helper: n98-magerun2 for day-to-day Magento CLI shortcuts
- IDE: PHPStorm

## Common commands
- `composer install`
- `bin/magento setup:upgrade`
- `bin/magento setup:di:compile`
- `bin/magento setup:static-content:deploy -f`
- `bin/magento cache:flush`
- `bin/magento module:enable <Vendor_Module>`

## Coding standards
- PSR-12 plus the Magento Coding Standard (`magento/magento-coding-standard`
  via phpcs)
- Prefer constructor dependency injection over `ObjectManager::getInstance()`
- New custom modules live under `app/code/<Vendor>/<Module>` unless the goal
  is a distributable extension package

## Working conventions
- Working docs/plans/session notes go in `docs/`, named
  `docs/YYYY-MM-DD-topic.md` — not in scratch/tmp directories
- This project has the `mattpocock-skills` plugin installed (Matt Pocock's
  engineering skills: https://github.com/mattpocock/skills) for workflow
  discipline:
  - `/tdd` — red-green-refactor for new features/fixes
  - `/code-review` — dual-axis review (standards + spec compliance) before
    merging
  - `/diagnosing-bugs` — disciplined debugging loop for any bug investigation
  - `/domain-modeling` — once real domain concepts exist in this project
  - `/implement` — feature work with integrated TDD + review
  - Run `/setup-matt-pocock-skills` once there is an actual issue
    tracker/backlog wired up for this repo, to configure triage labels and
    `CONTEXT.md` conventions. Not yet run — see docs/ for status.
