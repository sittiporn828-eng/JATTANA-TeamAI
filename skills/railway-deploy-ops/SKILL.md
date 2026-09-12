---
name: railway-deploy-ops
description: "Railway deploy/verify: CLI, secrets, deploy status, health."
---

# Railway Deploy Ops

Workflow for the user's Railway-hosted apps (MediLINE: project `mediline2027`, service `MediLINE`, production env). Deploy flow: **push main → Railway auto-deploy + GitHub Actions CI** — no manual upload needed.

## 1. CLI install (if missing)

```bash
npm install -g --allow-scripts=@railway/cli @railway/cli --foreground-scripts
export PATH="$(npm root -g)/@railway/cli/bin:$PATH"
```

- PITFALL: without `--allow-scripts=@railway/cli` the postinstall that downloads the binary is blocked → "could not find the CLI binary at ...node_modules\@railway\cli\bin\railway.exe".
- On this machine npm lives under the hermes-bundled node (`C:\Users\Acer\AppData\Local\hermes\node`) — global installs land there.

## 2. Auth

- Login persists in `~/.railway/config.json`: `user.accessToken`, `user.refreshToken`, and per-project links (`projects.<path>` → project/service/environment IDs).
- `railway whoami` refreshes an expired token and rewrites the file. Run it first if API calls 403.
- Linked project: `cd <repo>` then `railway status` shows workspace/project/environment.

## 3. Fetch prod secrets (REDIS_URL etc.)

- `railway variables` (plain) renders a table that **truncates long values** (`redis://` only).
- `railway variables --json` shape is unreliable — do not depend on it.
- Working method — GraphQL directly with the token from config.json:

```bash
TOKEN=$(node -e "console.log(require('$HOME/.railway/config.json').user.accessToken)")
curl -s --max-time 15 -X POST https://backboard.railway.app/graphql/v2 \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"query":"query($pid:String!,$eid:String!,$sid:String!){variables(projectId:$pid,environmentId:$eid,serviceId:$sid)}","variables":{"pid":"<projectId>","eid":"<environmentId>","sid":"<serviceId>"}}'
```

- Returns a name→value map (`data.variables.REDIS_URL`). Use the IDs from `~/.railway/config.json`.
- PITFALL: the args are **String!** — declaring `$pid: ID!` fails with "used in position expecting type String!". A bare `{ variables(...) }` with subfields fails too ("type EnvironmentVariables! has no subfields") — query it as a plain map.

## 4. Deploy status

```bash
railway deployment list   # SUCCESS / REMOVED rows with timestamps
gh run list --repo <owner>/<repo> --limit 4
gh run watch <runId> --repo <owner>/<repo> --exit-status   # blocks until CI done
```

- CI jobs on push: Quality Gates + Secret Scan. Verify BOTH are green, then confirm the newest `deployment list` row is SUCCESS and its timestamp matches the push time.

## 5. Health check

```bash
curl -s https://<app>.up.railway.app/health
```

- Expect `{"status":"ok","startup":"ready","database":"up","redis":"up"}` + per-queue `waiting/failed/delayed` counts. Failed > 0 → investigate (see `bullmq-redis-inspection` skill).
- `/healthz` may 404 and `/api/health` may 401 — `/health` is the one.

## Pitfalls

- After a local `build:ci`, Vite rewrites `public/index.html` asset hashes — **revert it** (`git checkout -- public/index.html`) before push; never commit build artifacts. Railway builds fresh on deploy.
- Verify suite for MediLINE: clear `*.tsbuildinfo` first (stale cache hides tsc errors), then `npm run verify` (tsc + vitest + oxlint) + `npm run build:ci`.
