# Fix Strava Sport Type Mapping

**Date:** 2026-03-02

## Problem

Strava webhook activities were arriving and being processed, but `processStravaWebhook` was
completing with `processed_challenges: 0` for many activities. Root cause: the fallback
`SPORT_TYPE_MAPPING` in `packages/backend/lib/strava.ts` used the key `"Running"` to match
Strava Run/TrailRun/VirtualRun types, but the challenge's activity type is named
**"Outdoor Run"**. Since the matching used `name.includes("running")`, it failed because
`"outdoor run".includes("running")` is false.

The preview scoring code in `actions/strava.ts` already had the correct mapping keys
(`"Outdoor Run"`, `"Rowing"`, `"Hi-Intensity Cardio"`, etc.) but the actual webhook
processing code in `lib/strava.ts` was out of sync.

## Impact

- **97 webhook events** from **32 unique athletes** failed to create activities in
  March Fitness 2026 (over ~36 hours since challenge start)
- Most affected: running activities (the most common Strava type)
- Also missing: Rowing, Hi-Intensity Cardio, Lo-Intensity Cardio mappings

## Fix

- [x] Added `Run` key to `SPORT_TYPE_MAPPING` in `lib/strava.ts` (matches "Outdoor Run")
- [x] Added `Rowing`, `Hi-Intensity Cardio`, `Lo-Intensity Cardio` keys to `lib/strava.ts`
  (previously only in preview code)
- [x] Made matching bidirectional: checks both `name.includes(key)` AND `key.includes(name)`
- [x] Applied same bidirectional matching to preview code in `actions/strava.ts`
- [ ] Deploy fix to production
- [ ] Replay missed webhook events to backfill activities
