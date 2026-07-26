---
name: Report connectivity usage across KORE products
description: Pull data-usage and session records for SIMs and accounts from
  ConnectivityPro, Super SIM and Programmable Wireless, respecting KORE's documented
  query-window limits.
api: openapi/kore-wireless-connectivity-pro.yml
also_uses:
  - openapi/kore-wireless-supersim.yml
  - openapi/kore-wireless-programmable-wireless.yml
operations:
  - getUsageRecordBySimNumberandDateRange
  - getUsageRecordByPlanId
  - getScheduledReportDownloadUrl
  - ListUsageRecord
  - ListBillingPeriod
  - listDataUsage
  - listDataUsageForSim
  - getDataSession
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-connectivity-pro.yml, openapi/kore-wireless-supersim.yml,
  openapi/kore-wireless-programmable-wireless.yml
---

# Report connectivity usage across KORE products

Usage lives in three places because KORE runs three connectivity platforms. Identify
the product first, then use its reader.

Authorize first — see `skills/kore-wireless-authorize-and-manage-api-clients.md`.

## ConnectivityPro / OmniSIM

- `getUsageRecordBySimNumberandDateRange` —
  `GET /v1/accounts/{account-id}/subscriptions/{subscription-id}/usage-records`
  returns usage for one subscription over a date range.
- `getUsageRecordByPlanId` — `GET /v1/accounts/{account-id}/plans/{plan-id}/usage-records`
  aggregates by plan.
- `getScheduledReportDownloadUrl` —
  `GET /v1/accounts/{account-id}/scheduled-reports/daily-subscriptions` returns the
  download URL for the scheduled daily subscription report. Prefer this over paging
  the API when you want a whole-account daily snapshot.
- Session state: `GET /v1/accounts/{account-id}/sessions` and
  `GET /v1/accounts/{account-id}/in-session` (neither declares an operationId).

## Super SIM

- `ListUsageRecord` — `GET /v1/UsageRecords` with grouping and granularity
  parameters.
- `ListBillingPeriod` — `GET /v1/Sims/{SimSid}/BillingPeriods` for the billing
  windows a SIM has been through.

**Respect the documented query limits** — these are enforced, not advisory:

| Constraint | Error |
|---|---|
| `StartTime`/`EndTime` must align to UTC day boundaries | 83601 |
| `StartTime`/`EndTime` must align to UTC hour boundaries | 83602 |
| Max 31 days when grouping by SIM | 83603 |
| Query period exceeds the max for the requested Granularity | 83604 |
| `StartTime` must be within the last 18 months | 83605 |

Chunk long ranges into compliant windows and stitch the results yourself.

## Programmable Wireless

- `listDataUsage` — `GET /v1/UsageRecords` for the whole account.
- `listDataUsageForSim` — `GET /v1/Sims/{Sid}/UsageRecords`.
- `getDataSession` — `GET /v1/Sims/{Sid}/DataSessions` for session-level detail.

## Rules

- **All times are GMT/UTC.** Convert before you build a window, not after.
- **Pagination**: ConnectivityPro returns the `meta-data` envelope
  (`count`, `page_size`, `page_number`, `next_page_url`); Super SIM and Programmable
  Wireless return the Twilio-style `meta` envelope. Follow the URLs in both cases.
- **Usage is not real-time billing.** Records settle; a recent window can change.
  Re-read rather than caching aggressively.
- **Errors**: `{status, message, code, more_info}` — see
  `errors/kore-wireless-error-codes.yml`.

## Related

- `conventions/kore-wireless-conventions.yml`
- `data-model/kore-wireless-data-model.yml`
