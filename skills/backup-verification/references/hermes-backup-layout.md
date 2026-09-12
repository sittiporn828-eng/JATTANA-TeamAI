# Hermes Backup Bundle Layout (user's machine)

Backup convention found on the user's machine (backup folder: `hermes-backup-YYYYMMDD`, e.g. `D:\hermes-backup-20260806`). Created via a backup script; verified intact by md5 checksums and full test-extraction (2026-08-07): all 4 archives extracted cleanly into a temp dir; only 2 dangling symlinks in `hermes-home` (`hermes/hermes-agent/.hermes-runtime/...`, `hermes/skills/hallmark`) — cosmetic.

## Bundle contents

| Archive | Restores to | Contains |
|---|---|---|
| `hermes-home.tar.gz` (~2.3 GB) | `C:\Users\<user>\AppData\Local\` | full `hermes/` folder: config, skills, memories, sessions |
| `github-projects.tar.gz` (~428 MB) | `C:\Users\<user>\Documents\` | git project checkouts |
| `jatana-group.tar.gz` (~193 MB) | `C:\Users\<user>\Desktop\` | work folder |
| `dotfiles.tar.gz` (~3 KB) | `C:\Users\<user>\` | `.ssh/` (3 GitHub keys: mediline, meekamrai, work1), `.railway/` token, `.gitconfig` |

Plus `checksums.md5` (md5 of each archive, written with `/e/...` absolute paths) and `README-restore.md` (Thai-language restore guide).

## Restore procedure (per README-restore.md)

1. Install Hermes on the new machine: `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
2. Stop Hermes on the new machine.
3. Extract `hermes-home.tar.gz` over `C:\Users\<user>\AppData\Local\`
4. Extract `github-projects.tar.gz` → `Documents\`, `jatana-group.tar.gz` → `Desktop\`, `dotfiles.tar.gz` → home.
5. Run `hermes doctor` — all checks pass = ready.
6. If the new username differs from `Acer`: edit `config.yaml` paths that reference `C:\Users\Acer\...` (e.g. Obsidian vault).

## Security note

Bundle contains secrets (API keys, SSH private keys, DB passwords, Railway token) — keep the medium offline/safe, never upload.
