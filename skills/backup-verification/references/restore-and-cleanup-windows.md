# Restore + cleanup on Windows (git-bash) — proven recipe

Observed on Windows 10, Hermes terminal (git-bash backend), restoring a 4-archive
Hermes backup bundle from a USB flash drive. Total restored: ~122k files.

## Deleting an extracted test tree (three failure modes, one recipe)

```bash
# 1) release any cwd lock FIRST — the Hermes terminal tool persists `cd` across
#    calls, and Windows locks a directory that is a process's CWD.
cd ~
# 2) find stray orphans from killed background processes (they hold handles too)
ps -W | grep -iE "bash|python|tar"    # look for PIDs started at the same minute as a killed proc
# 3) kill them via PowerShell (git-bash `taskkill //PID ...` escaping is flaky;
#    `cmd //c "rmdir /s /q"` is also unreliable — avoid both)
powershell -Command "Stop-Process -Id 1234,5678 -Force"
# 4) delete with Python rmtree + onerror chmod-retry (handles read-only .git files)
python -c "
import shutil, os, stat
def onerr(func, path, exc_info):
    os.chmod(path, stat.S_IWRITE)
    func(path)
shutil.rmtree(r'C:\path\to\tree', onerror=onerr)
print('DELETED')
"
# 5) empty dirs left behind (e.g. locked 'hermes' dir): plain rmdir, not -r
rmdir /c/Users/<user>/tree/hermes && rmdir /c/Users/<user>/tree
```

Failure mode reference:
- `rm -rf`: "Directory not empty" on `node_modules` → don't fight it, re-extract over
  the partial tree instead (tar overwrites + fills in) or use the Python recipe.
- `shutil.rmtree`: `PermissionError: [WinError 5] Access is denied: ...\pack-*.idx`
  → git objects are read-only; the onerror chmod-retry fixes it.
- `shutil.rmtree`: `[WinError 32] ... being used by another process` on a directory
  → something has it as CWD (often the terminal session itself) or a stray process
  holds it; cd out, kill strays, then delete.

## Detecting "already restored" (don't revert a live install)

Comparing live vs archive before overwriting an app home:

```bash
# live
ls -la ~/AppData/Local/hermes/config.yaml ~/AppData/Local/hermes/.env ~/AppData/Local/hermes/auth.json
# archived (tar -tvzf lists metadata; decompresses whole archive — slow on USB, fine for one file grep)
tar -tvzf hermes-home.tar.gz | grep -E "hermes/(config\.yaml|\.env|auth\.json)$"
```

Verdict logic that worked:
- `.env` live == archive (same 23,964 bytes AND same mtime Aug 2 23:12) → the home
  was ALREADY restored from this backup; only a fresh install would have a new .env.
- `config.yaml` / `auth.json` live NEWER (today's `hermes setup` run, fresh Nous
  Portal login) → extracting the archive would revert them and can break the running
  gateway. Skip the archive (or `tar --exclude` those paths) and explain; don't
  force-overwrite just because the user said "put everything".
- `state.db` is written continuously by the live gateway — never overwrite a live
  session store with the backup's.

## Extraction to real destinations

- Verify each target is empty first (`ls` the parent); report which destinations
  already had content and what was overwritten (e.g. default `.gitconfig` replaced
  by backup's real identity — that's desired).
- Big/many-small-file archives: `background=true, notify_on_complete=true`; a
  193MB / 72k-file archive took ~6-10 min from USB; the 2.3GB home took ~13 min.
- After extract: `wc -l errors.log` (0 = clean), `find <dest> -type f | wc -l`,
  top-level `ls`. tar EXIT=2 with only dangling-symlink errors is a pass.
- `du -sh` on 100k+ file trees can stall minutes — skip it or run `timeout 90 du`.
- Delete temp error logs you created in the user's home afterwards.
