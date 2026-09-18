# Magento Open Source Local Setup — Notes

## Goal
Fresh Magento Open Source 2.4.9 install, personal machine, no Adobe Commerce license.
Separate from the VOLT/client Adobe Commerce Cloud site (that work stays in its own
`~/Magento Backup` restore effort — different auth.json, different repo, not to be
mixed with this project).

## Stack decisions
- Local env tool: DDEV (Docker-based)
- Docker runtime: OrbStack (lighter than Docker Desktop, free for personal use)
- PHP: 8.4 (Magento 2.4.9 supports 8.4 or 8.5)
- DB: MySQL 8.4 (overrode DDEV's mariadb default to match Magento 2.4.9's stated requirement)
- Magento version: 2.4.9 (latest as of 2026-09, released 2026-05-12)

## Installed so far (via Homebrew)
- php@8.4, composer, ddev/ddev/ddev, orbstack (cask)
- DDEV project scaffolded at `.ddev/config.yaml`: type=magento2, docroot=pub

## Pending manual steps (require interactive/GUI access)
- [ ] Open OrbStack.app once to finish first-run setup
- [ ] Run `mkcert -install` interactively (needs sudo password prompt)
- [ ] Create free account at commercemarketplace.adobe.com, generate Composer
      access keys (My Profile → Access Keys) — needed to pull Open Source
      packages from repo.magento.com. Do NOT reuse the VOLT/client auth.json.
- [ ] Provide the public/private key pair so Composer auth can be configured
- [ ] `ddev start`, then `ddev composer create-project --repository=https://repo.magento.com/ magento/project-community-edition .`
- [ ] `ddev magento setup:install` (or run the setup wizard)

## Related, separate track (not part of this project)
- SSH keys generated (`~/.ssh/id_ed25519`) and added to `~/.ssh/config` for both
  github.com and bitbucket.org — for the VOLT/client Bitbucket repo clone and
  general GitHub auth. Public key needs to be added in both web UIs.
- Portfolio/sanitized-commit publishing idea flagged as needing employer/legal
  sign-off before any execution — holding until handbook review.
