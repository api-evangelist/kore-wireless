---
name: Receive and verify KORE webhooks and event streams
description: Create and rotate the webhook signing secret through the Webhook API,
  then verify every inbound KORE event — signature, idempotency token and CloudEvents
  envelope — before acting on it.
api: openapi/kore-wireless-webhook.yml
operations:
  - createSecret
  - getSecrets
  - modifySecret
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-webhook.yml, https://docs.korewireless.com/developers/webhooks
---

# Receive and verify KORE webhooks and event streams

KORE pushes events three ways: a callback URL set on an API resource, a callback URL
configured in the console, or a destination attached to Event Streams. All three are
signed and idempotent the same way.

Authorize first — see `skills/kore-wireless-authorize-and-manage-api-clients.md`.

## Step 1 — provision the signing secret

Host: `https://webhook.api.korewireless.com`.

- `createSecret` — `POST /v1/secrets` creates a new signing secret.
- `getSecrets` — `GET /v1/secrets` lists the secrets currently in force.
- `modifySecret` — `PATCH /v1/secrets/{id}` modifies an existing secret (this is the
  rotation path; see
  <https://docs.korewireless.com/developers/how-to/webhooks/rotate-my-webhook-secret>).

Store the secret in a secret manager. Support **two live secrets during rotation** so
in-flight deliveries signed with the old secret still verify.

## Step 2 — stand up the endpoint

- Must be **HTTPS**. KORE will not connect to a self-signed certificate.
- Must return a `2XX` **fast** — KORE's read timeout is 15,000 ms and it retries once,
  on HTTP 408, 423 or 5xx. Acknowledge first, process asynchronously.
- Do not allowlist by source IP: KORE sends from a pool with no published range.

## Step 3 — verify every request

For each inbound request, in this order:

1. **Verify `kore-signature`** against the stored secret
   (<https://docs.korewireless.com/developers/how-to/webhooks/validate-webhook-signatures>).
   Reject anything that fails — an unverified event is untrusted input.
2. **Deduplicate on `kore-idempotency-token`.** The value is unique per event and
   **stable across redeliveries**, so a token you have already processed means drop
   the event. KORE explicitly warns it may send the same event more than once.
3. Read `date` (UTC, `%Y-%m-%dT%H:%M:%SZ`) and confirm `user-agent` is `KoreProxy/1.1`.
4. Parse the body as `application/json`, UTF-8.

## Step 4 — read the CloudEvents envelope

Event Streams payloads are **CloudEvents 1.0**:

```json
{
  "data": { },
  "id": "fff521c2-c7db-4b53-a5a0-c5d5d01f66ce",
  "time": "2024-09-25T19:09:48.5842087+00:00",
  "type": "com.kore.eventstreams.test.event",
  "source": "kore-events",
  "dataschema": "/schemas/test/1",
  "specversion": "1.0",
  "datacontenttype": "application/json"
}
```

Branch on `type`. Validate `data` against the JSON Schema (draft-07) named by
`dataschema` — schemas are versioned and the streaming rule pins the version, so a
consumer can rely on the shape it subscribed to.

## Step 5 — test before you trust it

Trigger a **Test Event** from the destination detail page in the console
(Destinations → destination → actions → Test Event). It sends the
`com.kore.eventstreams.test.event` payload above through the real signing and
idempotency path — see `sandbox/kore-wireless-sandbox.yml`.

## Rules

- Never act on an event that failed signature verification.
- Never treat idempotency as optional: KORE's at-least-once delivery is documented,
  not theoretical.
- Errors returned by the Webhook API use the standard KORE envelope
  `{status, message, code, more_info}`.

## Related

- `asyncapi/kore-wireless-event-streams-webhooks.yml`
- `conventions/kore-wireless-conventions.yml`
- `errors/kore-wireless-error-codes.yml`
