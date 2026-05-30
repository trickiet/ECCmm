---
name: publish-content
description: Orchestrates the 5-stage content publish pipeline for Veneris AI. Runs draft → validate → deploy → verify (HARD GATE) → distribute. Any stage failure stops the pipeline with an exact error. Never claims success without HTTP 200 confirmation.
origin: taylor
---

# Publish Content Pipeline

Deterministic, gate-enforced content publishing for Veneris AI. Each stage must pass before the next begins.

## When to Use

- Publishing blog posts to veneris.ai
- Publishing LinkedIn content
- Any content that requires deploy + live verification

## Pipeline Stages

```
1. DRAFT      → content-drafter agent
2. VALIDATE   → brand-voice-validator agent  ← blocks on FAIL
3. DEPLOY     → deploy-verifier agent        ← blocks on non-200
4. VERIFY     ← HARD GATE (no bypass allowed)
5. DISTRIBUTE → cross-poster agent
```

## How to Invoke

```
/publish-content topic="<topic>" format="blog|linkedin" voice="<department>" output="<path>"
```

Or interactively — I'll ask for missing fields.

## Stage Details

### Stage 1: Draft
Invoke `content-drafter` with topic, format, voice, and output path.
Wait for `DRAFT_READY: <path>` before continuing.

### Stage 2: Validate
Invoke `brand-voice-validator` on the draft file.
- `DECISION: PROCEED` → continue
- `DECISION: REVISE` → stop, show violations, ask Taylor to revise or regenerate

### Stage 3: Deploy
Invoke `deploy-verifier` with the content file and target URL.
- Commits + pushes to git
- Polls for Vercel deploy

### Stage 4: Verify (HARD GATE)
`deploy-verifier` must confirm HTTP 200 + content assertion.
**No exceptions. No "it should be live shortly."**
If deploy times out (120s) → STOP with exact error.

### Stage 5: Distribute
Invoke `cross-poster` to schedule LinkedIn post (if format includes social).
Confirm API success before reporting done.

## Failure Policy

- ANY stage failure = STOP immediately
- Report exact stage, exact error, exact action needed
- Never claim success without verified 200
- Never retry a failed deploy without user confirmation

## Success Output

```
PUBLISH COMPLETE
Stage 1 DRAFT:    ✓ <file path>
Stage 2 VALIDATE: ✓ PASS
Stage 3 DEPLOY:   ✓ pushed to main
Stage 4 VERIFY:   ✓ https://veneris.ai/<slug> → 200
Stage 5 DISTRIBUTE: ✓ LinkedIn queued
```
