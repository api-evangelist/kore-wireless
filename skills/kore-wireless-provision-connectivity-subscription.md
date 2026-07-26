---
name: Provision and manage a KORE ConnectivityPro subscription
description: Check plan eligibility, activate a SIM, poll the provisioning request to
  completion, then suspend, change plan, reactivate or terminate it on KORE's
  ConnectivityPro/OmniSIM platform.
api: openapi/kore-wireless-connectivity-pro.yml
operations:
  - getPlanByAccountId
  - getPlanByPlanId
  - getSubscriptionsByAccountId
  - getSubscriptionDetailsBySubscriptionId
  - ActivateASim
  - CreateProvisioningRequest
  - getRequestStatusByProvisioningRequestId
  - getRequestStatusByAccountId
  - SuspendASim
  - ReactivateASim
  - PlanChangeForASim
  - DeactivateASim
  - TerminateASim
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-connectivity-pro.yml
---

# Provision and manage a KORE ConnectivityPro subscription

ConnectivityPro is KORE's own multi-carrier IoT connectivity platform (the OmniSIM
product family). Every operation is account-scoped: the `{account-id}` path
parameter is the KORE account (id prefix `CO`) your API Client targets.

Authorize first — see `skills/kore-wireless-authorize-and-manage-api-clients.md`.

## Step 1 — find what the account is eligible for

- `getPlanByAccountId` — `GET /v1/accounts/{account-id}/plans` lists every eligible
  plan. Use `getPlanByPlanId` for one plan's detail.
- `getServiceTypeByAccountId` and `getFeatureByAccountId` list the service types and
  features the account can buy.

Never assume a plan id — it is account-specific and must come from this call.

## Step 2 — locate the subscription

`getSubscriptionsByAccountId` — `GET /v1/accounts/{account-id}/subscriptions` lists
subscriptions (a subscription is one SIM's connectivity contract).
`getSubscriptionDetailsBySubscriptionId` fetches one.

## Step 3 — issue the state change

Each lifecycle action is its own POST under `/v1/accounts/{account-id}/provisioning-requests`:

| Intent | Operation |
|---|---|
| Bring a SIM into service | `ActivateASim` |
| Pause billing/traffic | `SuspendASim` |
| Resume after suspension | `ReactivateASim` |
| Move to a different plan or feature set | `PlanChangeForASim` |
| Take out of service (recoverable) | `DeactivateASim` |
| Permanently end the subscription | `TerminateASim` |
| Anything else the platform models | `CreateProvisioningRequest` |

`TerminateASim` is irreversible — treat it as a destructive action requiring explicit
human confirmation. `agentic-access/kore-wireless-agentic-access.yml` carries the
recommended execution contract for each of these operations.

## Step 4 — poll to completion

Provisioning is asynchronous. Take the provisioning request id from the response and
poll `getRequestStatusByProvisioningRequestId`
(`GET /v1/accounts/{account-id}/provisioning-requests/{provisioning-request-id}`)
until it reaches a terminal state. `getRequestStatusByAccountId` lists all outstanding
requests for the account.

Back off between polls; KORE publishes no rate-limit headers, and one operation in
this spec declares a 429 response.

## Step 5 — confirm

Re-read `getSubscriptionDetailsBySubscriptionId` and confirm the SIM state matches the
intent before reporting success. Do not infer success from the provisioning-request
acceptance alone.

## Rules

- **Pagination**: responses carry a `meta-data` object with `count`, `page_size`,
  `page_number`, `previous_page_url` and `next_page_url`. Follow `next_page_url`
  rather than incrementing offsets by hand.
- **Dates** are GMT.
- **Errors**: `{status, message, code, more_info}`. Codes are documented per-code at
  `https://docs.korewireless.com/errors/{code}` and catalogued in
  `errors/kore-wireless-error-codes.yml`.
- **Idempotency**: KORE publishes no request-side idempotency key. A retried
  provisioning POST can create a second request — always re-read the request list
  before retrying a write that may have landed.

## Related

- `conventions/kore-wireless-conventions.yml`
- `errors/kore-wireless-error-codes.yml`
- `data-model/kore-wireless-data-model.yml`
