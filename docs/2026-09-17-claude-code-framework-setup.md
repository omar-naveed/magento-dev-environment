# Claude Code framework setup — 2026-09-17

## Decisions made
- New dedicated repo `magento-dev-environment` (GitHub:
  omar-naveed/magento-dev-environment) instead of reusing
  `ci-binary-trust`, which is a separate CI/binary-trust initiative and
  shouldn't be mixed with an unrelated Magento project.
- Installed Matt Pocock's `mattpocock-skills` plugin (user scope) from the
  `mattpocock` marketplace (github.com/mattpocock/skills) for TDD,
  code-review, diagnosing-bugs, domain-modeling, implement, etc.
- This repo is public-facing portfolio work — see the sanitization rules in
  `CLAUDE.md`. The real employer Magento backup in `~/Magento Backup`
  (DB dump, media, `auth 4.json`, `service-config.json`) will **not** be
  restored into this repo. This environment uses Magento's official sample
  data instead.

## Open items / next steps
1. GitHub repo `magento-dev-environment` needs to actually exist — confirm
   it's created (public or private, your call) and get the remote wired up.
2. SSH key `~/.ssh/id_ed25519` is not yet registered with GitHub
   (`Permission denied (publickey)` on `ssh -T git@github.com`) — add
   `~/.ssh/id_ed25519.pub` under GitHub → Settings → SSH keys before pushing.
3. `mattpocock-skills` plugin was installed mid-session — restart Claude
   Code in this project directory to pick up its skills
   (`/tdd`, `/code-review`, `/implement`, etc.).
4. Once the local Magento install exists (composer create-project + DDEV),
   run `/setup-matt-pocock-skills` to wire up issue tracking/triage
   conventions for this repo.
5. Local environment (DDEV, composer install, sample data deploy) not yet
   done — this session only scaffolded the Claude Code framework
   (CLAUDE.md, .gitignore, README, plugin) and git init.
