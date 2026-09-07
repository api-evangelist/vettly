---
name: vettly-moderate-content
description: Check user-generated text, image, or video content against a Vettly moderation policy and act on the allow/warn/flag/block decision.
api: Vettly REST API
operations:
  - checkContent
generated: 2026-09-07
method: generated
source: openapi/vettly-content-moderation-openapi.json
---

# Moderate content with Vettly

Check content against a moderation policy and receive an auditable decision.

## Steps

1. Authenticate every request with `Authorization: Bearer vettly_live_...` (use a
   `vettly_test_` key while developing — both prefixes are documented in the API's
   own security scheme).
2. Call `checkContent` — `POST https://api.vettly.dev/v1/check` with a JSON body:
   - `content` (required): the text, or base64 image data.
   - `policyId` (required): the policy to apply (e.g. `moderate`).
   - `contentType` (optional): `text`, `image`, or `video`; defaults to `text`.
   - `requestId` (optional but recommended): an idempotency key — retrying with the
     same `requestId` returns the cached decision instead of re-processing.
   - `metadata` (optional): `userId`, `ip`, or custom keys for the audit trail.
3. Read the response: `action` is `allow`, `warn`, `flag`, or `block`;
   `categories[]` carries per-category `score`, `threshold`, and `triggered`;
   `decisionId` (`dec_*`) is the permanent audit-trail reference.
4. Act on the decision: publish on `allow`, route `flag` to human review, reject on
   `block`. Store `decisionId` with the content so appeals and replays can reference it.
5. Handle errors from the documented envelope `{"error": {"code", "message"}}`:
   retry with exponential backoff on `429` (respect `Retry-After` and the
   `X-RateLimit-*` headers) and on `5xx`; a `402 QUOTA_EXCEEDED` means the monthly
   plan quota is exhausted, not a rate limit.
