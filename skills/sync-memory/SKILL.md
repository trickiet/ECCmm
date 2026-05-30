---
name: sync-memory
description: Manually refresh ~/.claude/state/live.json and report what changed. Use when live data (OpenRouter balance, worker status, queue depths) may be stale mid-session. Also surfaces TTL-expired fields.
origin: taylor
---

# Sync Memory

Force-refreshes the live state file and shows what changed since last update.

## When to Use

- Before answering questions about OpenRouter balance, worker status, or queue depths
- When live.json is more than 10 minutes old
- After a known system event (deploy, worker restart, campaign launch)
- Anytime you want a current snapshot without waiting for the launchd interval

## How It Works

```bash
# 1. Run refresh script
bash ~/.claude/scripts/refresh-state.sh

# 2. Read new state
cat ~/.claude/state/live.json

# 3. Compare to previous state and report delta
```

## Output Format

```
SYNC COMPLETE — <timestamp>
Age before refresh: <N> minutes

Changes detected:
- openrouter.balance_usd: $X.XX → $Y.YY
- workers.scope_email.pm2_stopped: [] → [linkedin-poster-worker]
- queues.scope_email.depths.email-sender.failed: 0 → 23

No changes:
- deployments.veneris_website (last 200: <timestamp>)
```

If nothing changed: `No changes detected since last sync (<N> min ago).`

## TTL Enforcement

After syncing, flag any field that was stale (>10 min) before the sync:
```
⚠ Fields were stale for <N> min before this sync:
  - openrouter.balance_usd (was <X> min old)
```

## Manual invocation

```
/sync-memory
```

No arguments needed.
