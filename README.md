# KORE Wireless (kore-wireless)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

KORE Wireless (KORE Group Holdings) is an Atlanta, Georgia headquartered global IoT connectivity provider operating as a mobile virtual network operator (MVNO) across more than 190 countries, with over 20 million IoT connections under management. KORE sits in the aggregator half of the telecom value chain: it does not own spectrum or radio access network, it resells and orchestrates multi-carrier cellular connectivity, eSIM/iSIM provisioning, device management, and IoT security as a service to enterprises in healthcare, fleet, logistics, utilities, and industrial automation. In 2024 KORE acquired the Twilio IoT business — Super SIM, Programmable Wireless, and Microvisor — inheriting a genuinely developer-first API surface, and it has kept that posture. In 2026 KORE was taken private by Searchlight Capital Partners and Abry Partners and delisted from the NYSE.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/kore-wireless/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/kore-wireless/refs/heads/main/apis.yml)

## Tags

- Telecommunications
- United States
- IoT
- eSIM
- Connectivity
- MVNO
- SIM Management
- Roaming
- Messaging
- SMS
- Device Management
- Network APIs

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## API Posture

KORE lands in the API-native half of the telecom split. Documentation at
[docs.korewireless.com](https://docs.korewireless.com/) is fully open — no login, published as GitBook with
`llms.txt`, `llms-full.txt`, and per-page `.md` content negotiation for agents. Account registration at
[console.korewireless.com](https://console.korewireless.com/) is self-serve. Most importantly, KORE publishes
**eight OpenAPI 3.0 specifications** in a public GitHub repository —
[korewireless/kore-openapi](https://github.com/korewireless/kore-openapi) — in both YAML and JSON, with a
machine-readable `specs/catalog.json` and a `make`-based openapi-generator SDK workflow. All eight were
harvested verbatim into [`openapi/`](openapi/) and all eight parse as valid OpenAPI.

Authorization is **OAuth 2.0 client credentials** (RFC 6749 §4.4) against
`https://api.korewireless.com/api-services/v1/auth/token`, with an `x-api-key` header option on ConnectivityPro
and SMS and legacy Twilio-style HTTP basic on Super SIM. No OAuth scopes are published. No OIDC discovery
document is served.

### CAMARA posture

**KORE exposes no CAMARA APIs, and there is not even a press release claiming otherwise.** A grep of the
complete published documentation index (358 entries) for CAMARA, Open Gateway, TM Forum, TMF, NEF, SCEF, and
network slicing returned zero matches, and none of the eight specs contains a CAMARA-shaped operation. KORE is
not a GSMA Open Gateway operator participant and has no Aduna channel relationship. This is expected and
correct: Open Gateway is an MNO commitment layer, and as an MVNO KORE *consumes* carrier network capability
rather than exposing it. Consistently, **CIBA does not appear anywhere** in KORE's auth surface. KORE holds no
TM Forum Open API conformance certification and publishes no NEF/SCEF, 5G network-slicing, or edge/MEC API.

## APIs

### KORE Connectivity Pro API

The largest of KORE's published surfaces — 49 paths and 55 operations covering SIM and eSIM provisioning, eSIM profile management, activation profiles, subscriptions, accounts, usage and session data, diagnostics, eligibility checks, testing, alerting, and reporting for KORE's global multi-carrier IoT connectivity platform.

- **Human URL:** [https://developer.korewireless.com/](https://developer.korewireless.com/)
- **Base URL:** `https://api.korewireless.com/connectivity`
- [OpenAPI](openapi/kore-wireless-connectivity-pro.yml)
- [Documentation](https://docs.korewireless.com/omnisim/kore-omnisim-r)

### KORE Super SIM API

The multi-IMSI global cellular connectivity platform KORE acquired from Twilio IoT. 20 paths and 31 operations across Sim, Fleet, NetworkAccessProfile, Network, eSimProfile, SettingsUpdate, BillingPeriod, UsageRecord, SmsCommand, IpCommand, and OTA resources.

- **Human URL:** [https://docs.korewireless.com/api/products/supersim](https://docs.korewireless.com/api/products/supersim)
- **Base URL:** `https://supersim.api.korewireless.com`
- [OpenAPI](openapi/kore-wireless-supersim.yml)
- [Documentation](https://docs.korewireless.com/supersim/supersim)

### KORE Programmable Wireless API

The legacy Twilio IoT cellular product now operated by KORE. 9 paths and 16 operations covering Wireless Sim, RatePlan, Command, DataSession, and account- and SIM-level UsageRecord resources.

- **Human URL:** [https://docs.korewireless.com/api/products/programmable-wireless-r](https://docs.korewireless.com/api/products/programmable-wireless-r)
- **Base URL:** `https://programmable-wireless.api.korewireless.com`
- [OpenAPI](openapi/kore-wireless-programmable-wireless.yml)

### KORE SMS API

Programmatically exchange short messages with IoT devices on KORE connectivity — send, list, and retrieve message history. Device-directed machine-to-machine SMS, not an A2P/CPaaS messaging product.

- **Human URL:** [https://developer.korewireless.com/](https://developer.korewireless.com/)
- **Base URL:** `https://api.korewireless.com/sms`
- [OpenAPI](openapi/kore-wireless-sms.yml)

### KORE Webhook API

Creates, retrieves, and modifies the signing secrets used to verify callbacks KORE sends to customer endpoints. KORE webhooks are signature-verified and idempotent; the companion Event Streams product delivers CloudEvents 1.0 events with versioned JSON Schema (draft-07) payloads to webhook or AWS Kinesis destinations.

- **Human URL:** [https://docs.korewireless.com/api/global-resources/webhook](https://docs.korewireless.com/api/global-resources/webhook)
- **Base URL:** `https://webhook.api.korewireless.com`
- [OpenAPI](openapi/kore-wireless-webhook.yml)
- [Documentation](https://docs.korewireless.com/developers/webhooks)

### KORE Identity and Access Management API

Manages customer accounts and their relationships — creating accounts, retrieving account hierarchies, and managing platform-specific mappings between parent and child accounts.

- **Human URL:** [https://docs.korewireless.com/api/global-resources/iam/account](https://docs.korewireless.com/api/global-resources/iam/account)
- **Base URL:** `https://iam.api.korewireless.com`
- [OpenAPI](openapi/kore-wireless-iam.yml)

### KORE API Clients API

Creates, retrieves, and lists the API Clients that hold the credentials and settings for an integration with KORE. An API Client is the required gateway to every other KORE API.

- **Human URL:** [https://docs.korewireless.com/api/global-resources/api-clients/clients](https://docs.korewireless.com/api/global-resources/api-clients/clients)
- **Base URL:** `https://client.api.korewireless.com`
- [OpenAPI](openapi/kore-wireless-api-clients.yml)

### KORE Token API

The OAuth 2.0 authorization endpoint for the whole platform. A single POST to `/v1/auth/token` exchanges Client Credentials for a bearer access token per RFC 6749 §4.4.

- **Human URL:** [https://docs.korewireless.com/developers/api-management/auth](https://docs.korewireless.com/developers/api-management/auth)
- **Base URL:** `https://api.korewireless.com/api-services`
- [OpenAPI](openapi/kore-wireless-token.yml)

## Links

- [Website](https://www.korewireless.com/)
- [Documentation](https://docs.korewireless.com/)
- [API Reference](https://docs.korewireless.com/api/api-reference)
- [Developer Console](https://console.korewireless.com/)
- [OpenAPI Repository](https://github.com/korewireless/kore-openapi)
- [GitHub Organization](https://github.com/korewireless)
- [Status](https://korewireless.service-now.com/csm?id=services_status)
- [Blog](https://www.korewireless.com/blog/)
- [News](https://www.korewireless.com/news/)
