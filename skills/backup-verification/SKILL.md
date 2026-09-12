---
name: backup-verification
description: "Use when verifying backups: checksums, gzip/tar integrity."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [backup, verification, checksum, md5, tar, gzip, usb, flash-drive, restore]
---

# Backup / Archive Verification

Use when the user asks to check whether a copy/extraction/backup succeeded — e.g. "ตรวจว่าคลายไฟล์สำเร็จไหม", "check the flash drive", "did the backup copy over OK". Goal: prove files are complete and uncorrupted, and tell the user whether they are still compressed vs already extracted.

## Workflow

### Distinguish project completeness from application-state completeness

A disk can have all Git repositories intact while still missing application state, caches, sessions, databases, or backup bundles. Check both layers:

- **Projects:** compare repository presence, branch/HEAD, tracked-file count, and `git status --porcelain` across source and copy.
- **Application state:** compare the *actual data roots*, not drive roots. For this Windows Hermes setup, compare `D:\hermes` with `C:\Users\Acer\AppData\Local\hermes`; do not assume they are the same logical location. At minimum compare top-level names and sizes of databases/configs, then inspect only identified differences.
- **Large trees:** avoid an unbounded recursive file scan as the first pass. It can exceed several minutes on HDDs and gives no useful answer if it times out. Use top-level inventories and Git metadata first; reserve hashes or recursive comparison for a named subset.
- Report separately: `project files complete`, `application state differs`, and `backup bundle missing/unverified`. Never collapse these into a single "all data complete" verdict.

1. **Find the target drive.** On Windows git-bash, drives mount as `/c`, `/d`, `/e`, ... Probe them directly:
   ```bash
   for d in c d e f g h i; do [ -d /$d ] && echo "/$d"; done
   ```
   `System Volume Information` at the drive root is a tell-tale of a removable/USB drive. Avoid `wmic logicaldisk` from git-bash — it can drop into an empty `cmd` shell and return nothing useful.

2. **Read the docs first.** If the folder has a README / restore guide / `checksums.md5`, read them before verifying — they define the expected layout and hashes. They may also reveal that the files are a *backup bundle* (still compressed) rather than extracted output.

3. **Verify checksums.** Checksum files often embed absolute paths from the machine/mount where they were written (e.g. `*/e/hermes-backup-.../x.tar.gz` while the drive is now `/d/`). Do NOT run `md5sum -c` blindly — it will fail on path mismatch. Instead compute and compare manually:
   ```bash
   cd /d/<backup-folder> && md5sum *.tar.gz
   ```
   Multi-GB files take minutes on USB — use a generous timeout or `background=true, notify_on_complete=true`.

4. **Verify archive validity** (format + readable):
   ```bash
   gzip -t file.tar.gz                 # gzip stream OK?
   tar -tzf file.tar.gz | head         # integrity + peek contents
   ```
   Note `tar -tzf` decompresses the whole archive, so it is slow for multi-GB files — a matching md5 is usually sufficient proof for the big ones; spot-check the small ones.

5. **Test-extract when asked to prove extraction works.** Extract into a TEMP dir, never the real destination:
   ```bash
   mkdir -p ~/backup-test && tar -xzf X.tar.gz -C ~/backup-test 2> ~/backup-test/errors.log
   ```
   Start with the smallest archive; run large (>200MB) or many-small-file archives with `background=true, notify_on_complete=true` and report progress rather than polling in a loop. Confirm completeness by comparing counts:
   ```bash
   tar -tzf X.tar.gz | wc -l          # files in archive
   find ~/backup-test -type f | wc -l # files extracted
   ```
   Run heavy scans (find/du) only AFTER extraction finishes — they contend with the writer and both stall.

6. **Report a table**: file, size, checksum match (✅/❌), integrity, extraction result. Then state clearly: *checksums pass = copy OK*, and distinguish "copy verified" from "extraction still pending" — and offer to delete the secret-bearing test dir when done.

## Restoring for real (after verification)

When the user wants the backup brought onto the machine, not just verified:

1. **Survey destinations BEFORE extracting.** Check whether each target already holds content, so nothing gets clobbered silently:
   ```bash
   ls -la ~/.ssh ~/.railway ~/.gitconfig 2>&1; ls ~/Documents/GitHub 2>&1; ls ~/Desktop 2>&1
   ```
   Extract only where targets are missing/empty; explicitly flag anything that exists and would be overwritten (e.g. a default `.gitconfig` vs the backup's one with real identity).

2. **Detect "already restored" before overwriting a live app home.** If the destination looks populated, compare live files against the archive:
   ```bash
   ls -la ~/AppData/Local/hermes/config.yaml ~/AppData/Local/hermes/.env   # live files
   tar -tvzf X.tar.gz | grep -E "hermes/(config\.yaml|\.env|auth\.json)$"   # archived versions
   ```
   Live file identical in size AND timestamp = the archive was already extracted onto this machine (same file on disk) — nothing to restore. Newer live timestamps = changes made since the backup (setup runs, fresh logins) — **do not overwrite those**; extracting the archive would revert the new machine's config and can break a running service (e.g. the gateway's `config.yaml`/`auth.json`). Skip that archive or extract with `--exclude`, and tell the user why instead of force-overwriting.

3. **Extract each archive to its real destination** per the restore README (background + notify for the big ones), then verify: `wc -l errors.log` is empty, file counts match, top-level structure looks right.

4. **Clean up temp artifacts** you created in the user's home (error logs, test dirs).

## Pitfalls

- **Deleting extracted trees on Windows (git-bash)** has three distinct failure modes — see `references/restore-and-cleanup-windows.md` for the proven recipe:
  - `rm -rf` dies on `node_modules` ("Directory not empty" — long paths).
  - `shutil.rmtree` dies on read-only `.git` pack files (WinError 5) — fix with an `onerror` handler that `os.chmod(path, stat.S_IWRITE)` then retries.
  - A directory that is a process's CWD is locked (WinError 32) — the Hermes terminal tool persists `cd` across calls, so if a previous command `cd`'d into the tree being deleted, run `cd ~` first, then delete. Killed background extractions also leave orphan bash/python children holding handles — find via `ps -W`, kill via PowerShell `Stop-Process -Id <pid> -Force` (git-bash `taskkill //PID` escaping is flaky), then `rmdir` the empty leftovers.
- **`cmd //c "..."` one-liners from git-bash are unreliable** (MSYS path translation can drop into an interactive cmd; exit 0 but nothing runs). Use Python or plain `rmdir` for empty dirs instead.
- **Archive present ≠ extracted.** A `.tar.gz` on the flash drive means the restore/extraction step still has to happen on the target machine. Say so explicitly instead of answering "yes, extraction succeeded".
- **Backups contain secrets.** SSH keys, API keys, DB passwords, tokens. Remind the user to keep the medium safe and never upload it.
- **Drive letters change.** Checksums written for `/e/` may now live on `/d/` — always recompute rather than trusting `-c`.
- **USB read speed** dominates runtime; don't set tight foreground timeouts on big archives.
- **Foreground timeout kills mid-extraction.** A 193MB archive with ~72k small files exceeded 300s from USB; a 2.3GB archive took ~13 min. Use background + notify for anything non-trivial.
- **`rm -rf` fails on partially-extracted `node_modules`** — Windows "Directory not empty" (long paths / locked files). Don't delete: **re-extract over the partial tree** — tar overwrites existing files and fills in the rest.
- **`tar` EXIT=2 with "Cannot create symlink ... No such file or directory"** = dangling absolute-path symlinks inside the archive (transient runtime dirs, paths outside the backup root). Cosmetic — judge by error count vs total files, not the exit code. Observed case: 2 symlink errors out of 247,144 files = success.

## References

- `references/hermes-backup-layout.md` — the Hermes backup bundle convention (what each `*.tar.gz` contains and where it restores to), as used on the user's machine.
- `references/restore-and-cleanup-windows.md` — proven Windows (git-bash) recipes: deleting extracted test trees (read-only `.git` files, cwd-locked dirs, orphaned processes), detecting an already-restored home, and safe extraction to real destinations.
