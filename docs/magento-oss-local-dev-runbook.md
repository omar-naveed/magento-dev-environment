# Magento Open Source Local Dev — Runbook

Living reference for this project. Update as things change — don't recreate, edit in place.

## Scope
Personal Magento Open Source install, no Adobe Commerce license. Fully separate
from the VOLT/client Adobe Commerce Cloud site (`~/Magento Backup` restore effort) —
different Composer auth, different repo, different purpose. Don't cross-wire the two.

## Stack
| Component | Choice | Why |
|---|---|---|
| Local env tool | DDEV | Easiest Docker-based Magento setup, good PHPStorm integration |
| Docker runtime | OrbStack | Lighter/faster than Docker Desktop, free for personal use |
| PHP | 8.4 | Magento 2.4.9 supports 8.4 or 8.5 |
| Database | MySQL 8.4 | Magento 2.4.9's stated requirement — DDEV's `magento2` type defaults to MariaDB, had to override |
| Magento | Open Source 2.4.9 | Latest as of 2026-09 (released 2026-05-12) |
| Project path | `~/Sites/magento-oss` | |

## One-time prerequisite steps (manual, GUI/interactive — can't be scripted)
1. Open **OrbStack.app** once to finish first-run setup and grant permissions.
2. Run `mkcert -install` in a real terminal (not sandboxed) — needs sudo password
   to add the local CA to the macOS keychain. Without this, DDEV's HTTPS certs
   show as untrusted in the browser.
3. Free Adobe Commerce Marketplace account → My Profile → Access Keys → generate
   a key pair (labeled e.g. `magento-oss-local-macbook`). Needed because Magento
   Open Source packages are still pulled from `repo.magento.com`, which requires
   auth even for the free edition.
   - **Do NOT reuse the VOLT/client `auth.json`** from the Commerce Cloud backup —
     that's a different account/license entirely.

## Completed setup log
- Installed via Homebrew: `php@8.4`, `composer`, `ddev/ddev/ddev`, `orbstack` (cask)
- `brew link --overwrite --force php@8.4` (a php 8.5 was already linked as a
  dependency of something else — had to force-relink 8.4 as default)
- DDEV project scaffolded: `.ddev/config.yaml` → type `magento2`, docroot `pub`,
  php `8.4`, database overridden to `mysql:8.4` (default was `mariadb:12.3`)
- Composer global auth configured for `repo.magento.com`:
  `composer global config http-basic.repo.magento.com <public-key> <private-key>`
  → written to `~/.config/composer/auth.json`, permissions locked to `600`

## Status: INSTALLED ✅ (2026-09-17)
- Storefront: https://magento-oss.ddev.site/ (HTTP 200)
- Admin: https://magento-oss.ddev.site/admin (HTTP 200)
- Admin username: `admin` — password generated and shared with Omar directly,
  not stored in this repo. Stored in 1Password.
- Magento version installed: 2.4.9
- Search engine: OpenSearch (via `ddev/ddev-opensearch` add-on — not included
  by default in DDEV's `magento2` project type, had to add separately)

## Full install steps (for rebuilding from scratch)
1. Confirm Docker is up: `docker info` should succeed (not error) once OrbStack is running.
2. Confirm mkcert CA is trusted: `mkcert -CAROOT` should show a rootCA.pem that's
   also present in the login keychain (or just trust the first `ddev start` — it
   will warn if certs aren't trusted).
3. `cd ~/Sites/magento-oss && ddev start`
4. Add OpenSearch service (not in `magento2` type by default):
   `ddev add-on get ddev/ddev-opensearch && ddev restart`
5. `ddev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition:2.4.9 .`
   — directory must be genuinely empty (see gotcha below); pulls the codebase using
   the global Composer auth already configured.
6. `ddev magento setup:install` with base-url, db-*, admin-*, search-engine=opensearch
   flags (see gotcha below re: stale `env.php`/`config.php` if retrying after a
   failed attempt).
7. Verify: `ddev launch` should open the storefront in a browser over HTTPS with a trusted cert.

## Gotchas encountered
- **`ddev composer create-project` requires a genuinely empty/clean directory** —
  it only tolerates `app app/etc app/etc/.gitignore app/etc/env.php pub pub/media`.
  Our `docs/` folder (created before the Magento install) blocked it. Fix: move
  `docs/` out of the project dir, run create-project, move it back afterward.
- **Global Composer auth on the HOST is NOT visible inside DDEV containers.**
  `composer global config http-basic.repo.magento.com ...` writes to
  `~/.config/composer/auth.json` on the Mac, but `ddev composer` runs Composer
  *inside* the web container, which has its own home directory. Fix: also copy
  the auth.json into DDEV's global home-additions dir so it's mounted into every
  project's container home:
  ```
  mkdir -p ~/.ddev/homeadditions/.composer
  cp ~/.config/composer/auth.json ~/.ddev/homeadditions/.composer/auth.json
  chmod 600 ~/.ddev/homeadditions/.composer/auth.json
  ddev restart   # required to pick up homeadditions changes
  ```
  Verify with `ddev exec 'cat $HOME/.composer/auth.json'` (container `$HOME` is
  `/home/<user>`, not the Mac's `/Users/<user>`).
- **PHP version conflict**: a newer PHP (8.5) was already brew-linked as a
  dependency of another package before we explicitly installed `php@8.4`. Had to
  `brew link --overwrite --force php@8.4` to make it the active `php` on PATH.
- **DDEV's `magento2` type defaults to MariaDB**, but Magento 2.4.9's docs
  specifically call out MySQL 8.4 — reconfigured with
  `ddev config --database=mysql:8.4` after the initial scaffold.
- **`mkcert -install` can't run from a sandboxed/non-interactive shell** — it
  needs a real TTY to prompt for the sudo password for keychain access. Must be
  run by the user directly in a terminal.
- **OrbStack needs a GUI first-run** — can't be silently initialized from the
  command line the first time.
- **Composer keys for Magento Open Source are still required** even though
  there's no paid license involved — `repo.magento.com` gates on auth regardless
  of edition. Free to generate, free to rotate/revoke anytime from the
  Marketplace account if ever a concern.
- **DDEV's `magento2` project type does NOT include OpenSearch/Elasticsearch**
  despite Magento 2.4.9 requiring one for catalog search. Fix:
  `ddev add-on get ddev/ddev-opensearch` then `ddev restart`, then pass
  `--search-engine=opensearch --opensearch-host=opensearch --opensearch-port=9200`
  to `setup:install`.
- **`setup:install` failures leave stale state that breaks the next attempt** —
  this bit us twice:
  1. First run failed ~1/10th of the way through (`Cannot instantiate interface
     ...DefaultStockProviderInterface`, during the `Magento_InventoryCatalog`
     schema patch — looked like a transient module-dependency-order issue).
  2. Retried after only dropping the DB and clearing `generated/`/`var/cache` —
     failed differently and much earlier (`WebsiteRepository: The default
     website isn't defined`), because **`app/etc/env.php` and `app/etc/config.php`
     survive a failed install** and get reused on the next run, causing bad state.
  - **Full reset recipe** before retrying any failed `setup:install`:
    ```
    rm -f app/etc/env.php app/etc/config.php
    rm -rf generated/code generated/metadata var/cache var/page_cache var/di var/generation var/log
    ddev mysql -e "DROP DATABASE IF EXISTS db; CREATE DATABASE db;"
    ```
    Third attempt with this full reset succeeded end-to-end (all 1480 modules,
    admin user created, indexers built).

## Troubleshooting
- `ddev start` hangs or fails → check `docker info` first; if Docker/OrbStack
  isn't running, DDEV can't create containers.
- Browser shows cert warning on `*.ddev.site` → mkcert CA isn't trusted yet, rerun
  `mkcert -install`, then `ddev restart`.
- Composer auth errors pulling Magento packages → check
  `~/.config/composer/auth.json` has the `http-basic.repo.magento.com` entry, keys
  haven't been revoked/rotated on the Marketplace side.
