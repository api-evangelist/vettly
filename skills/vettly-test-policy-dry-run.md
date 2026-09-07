---
name: vettly-test-policy-dry-run
description: Rehearse a Vettly moderation policy with mock category scores before sending real traffic, without calling any AI provider.
api: Vettly REST API
operations:
  - dryRunCheck
generated: 2026-09-07
method: generated
source: openapi/vettly-content-moderation-openapi.json
---

# Test a moderation policy with a dry run

Verify what a policy would decide before wiring it into production — no AI provider
calls, no cost, no decisions written against real content.

## Steps

1. Authenticate with `Authorization: Bearer vettly_test_...` (test-mode key prefix)
   or a live key.
2. Call `dryRunCheck` — `POST https://api.vettly.dev/v1/check/dry-run` with:
   - `policyId` (required): the policy to rehearse (e.g. `strict`).
   - `mockScores` (optional): a map of category → score, e.g.
     `{"violence": 0.8, "sexual": 0.3, "hate": 0.95}`.
3. Read the response the same way as a real check: `action`
   (`allow`/`warn`/`flag`/`block`) and `categories[]` showing which thresholds the
   mock scores would trigger.
4. Iterate on threshold edge cases (scores just above and below each category
   threshold) until the policy behaves as intended, then switch the integration to
   `checkContent` (`POST /v1/check`) with real content.
