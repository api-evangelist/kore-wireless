---
name: Register Super SIMs and control which networks they use
description: Build a Network Access Profile, attach networks to it, create a Fleet
  bound to that profile, register Super SIMs, move them into the Fleet and activate
  them.
api: openapi/kore-wireless-supersim.yml
operations:
  - CreateNetworkAccessProfile
  - ListNetworkAccessProfile
  - FetchNetworkAccessProfile
  - UpdateNetworkAccessProfile
  - ListNetwork
  - FetchNetwork
  - CreateNetworkAccessProfileNetwork
  - ListNetworkAccessProfileNetwork
  - DeleteNetworkAccessProfileNetwork
  - CreateFleet
  - ListFleet
  - FetchFleet
  - UpdateFleet
  - CreateSim
  - ListSim
  - FetchSim
  - UpdateSim
  - ListSettingsUpdate
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-supersim.yml
---

# Register Super SIMs and control which networks they use

Super SIM is the multi-IMSI global connectivity platform KORE acquired from Twilio
IoT. The resource names and `Sid` identifiers are the Twilio-era ones; the host is
`https://supersim.api.korewireless.com`.

Authorize first — see `skills/kore-wireless-authorize-and-manage-api-clients.md`.

## Step 1 — choose the networks

`ListNetwork` (`GET /v1/Networks`) enumerates the mobile networks Super SIMs can
attach to; `FetchNetwork` fetches one. Note the `Sid` of each network you want to
allow.

## Step 2 — create a Network Access Profile

`CreateNetworkAccessProfile` (`POST /v1/NetworkAccessProfiles`) creates the policy
object. Then attach each allowed network with
`CreateNetworkAccessProfileNetwork`
(`POST /v1/NetworkAccessProfiles/{NetworkAccessProfileSid}/Networks`).

Verify with `ListNetworkAccessProfileNetwork`. Remove one with
`DeleteNetworkAccessProfileNetwork` — this is a live traffic-steering change: a SIM
attached to a removed network will be rejected with error **83702 Attachment Rejected
Due To Network Not Allowed**.

## Step 3 — create a Fleet

`CreateFleet` (`POST /v1/Fleets`) creates the group that carries the data/SMS/IP
settings and binds the Network Access Profile. `UpdateFleet` changes them later.
A Fleet is where per-SIM behaviour is actually configured — do not expect per-SIM
settings.

## Step 4 — register and place the SIMs

- `CreateSim` (`POST /v1/Sims`) registers a Super SIM against your account. Error
  **83003** means it is already yours; **83002** means it cannot be registered.
- `UpdateSim` (`POST /v1/Sims/{Sid}`) sets the Fleet and the status.
- A SIM **must belong to a Fleet before it can be activated** — activating a
  fleetless SIM returns error **83007**.
- You cannot change the Fleet of a SIM whose status is `scheduled` (**83009**).

Confirm with `FetchSim` / `ListSim`.

## Step 5 — watch the settings propagate

`ListSettingsUpdate` (`GET /v1/SettingsUpdates`) reports the over-the-air settings
updates queued or applied to SIMs after a Fleet or profile change. Settings changes
are not instantaneous — do not assert success until the update lands.

## Rules

- **Pagination** on this surface uses the Twilio-style `meta` envelope
  (`page`, `page_size`, `first_page_url`, `next_page_url`, `previous_page_url`, `key`)
  rather than the KORE `meta-data` envelope used by ConnectivityPro. Follow the URLs.
- **Errors** in the 83xxx block are Super SIM specific and fully catalogued in
  `errors/kore-wireless-error-codes.yml`.
- **Updates use POST, not PUT/PATCH** on this API (Twilio inheritance).
- Traffic steering is a production control action — changing a Network Access Profile
  can disconnect a fleet in the field. Treat it as high-consequence.

## Related

- `conventions/kore-wireless-conventions.yml`
- `data-model/kore-wireless-data-model.yml`
- `agentic-access/kore-wireless-agentic-access.yml`
