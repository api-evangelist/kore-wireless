---
name: Authorize against KORE and manage API Clients
description: Exchange KORE API Client credentials for an OAuth 2.0 access token, verify
  gateway reachability, and create or inspect the API Clients that hold those credentials.
api: openapi/kore-wireless-api-clients.yml
also_uses:
  - openapi/kore-wireless-token.yml
operations:
  - getPingStatus
  - listClient
  - createClient
  - getClient
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-api-clients.yml, openapi/kore-wireless-token.yml, https://docs.korewireless.com/developers/api-management/auth
---

# Authorize against KORE and manage API Clients

Every other KORE skill depends on this one. An **API Client** is the gateway to all
KORE APIs: it holds the Client ID and Client Secret, and its selected scopes decide
which products and global resources the resulting token can reach.

## Preconditions

- A KORE account (self-serve at <https://console.korewireless.com/>).
- An API Client created in <https://build.korewireless.com/clients/list>, with the
  Products and Global Resources this workflow needs selected. The Client Secret is
  shown **once** at creation and cannot be retrieved afterwards.

## Step 1 — mint an access token

`POST https://api.korewireless.com/api-services/v1/auth/token`
(spec: `openapi/kore-wireless-token.yml`, path `/v1/auth/token`, no operationId is
declared on this operation).

```
curl -X POST https://api.korewireless.com/api-services/v1/auth/token \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data "grant_type=client_credentials" \
  --data "client_id=$KORE_CLIENT_ID" \
  --data "client_secret=$KORE_CLIENT_SECRET"
```

The response is `{access_token, expires_in, token_type: "Bearer", scope}`.
`expires_in` is in seconds and reflects the expiry the API Client was created with
(1 hour, 24 hours, 30 days, or 24 months). There is **no refresh token** — when the
token expires, repeat this call.

Cache the token for its lifetime rather than minting one per request, but re-mint
immediately after any scope change on the client: existing tokens keep their old
scopes until they expire.

## Step 2 — send the token on every call

Add `Authorization: Bearer {access_token}` to every request against every KORE host
(`api.korewireless.com`, `supersim.api.korewireless.com`,
`programmable-wireless.api.korewireless.com`, `client.api.korewireless.com`,
`iam.api.korewireless.com`, `webhook.api.korewireless.com`).

## Step 3 — confirm reachability

Call `getPingStatus` (`GET /v1/ping` on `https://client.api.korewireless.com`) to
confirm the gateway is up and the token is accepted before running a larger flow.
ConnectivityPro has the equivalent `getConnectionStatus` (`GET /v1/ping` on
`https://api.korewireless.com/connectivity`).

## Step 4 — inspect or create clients (Admin clients only)

- `listClient` — `GET /v1/clients` lists the API Clients on an account. Page it with
  `page_size` and `page_number`; read `meta-data.next_page_url` to walk forward.
- `getClient` — `GET /v1/clients/{client_id}` fetches one client.
- `createClient` — `POST /v1/clients` creates a client. This requires a token whose
  Global Resources include **API Clients** with Write access; a Standard client
  cannot do this.

## Rules

- **Errors**: failures return `{status, message, code, more_info}` as
  `application/json`. `401 unauthorized_client` means bad credentials at the token
  endpoint; `403 "Invalid or expired token"` means the token expired or the client
  lacks the scope. Look the numeric `code` up in
  `errors/kore-wireless-error-codes.yml`.
- **Never log the Client Secret or the access token.**
- **Child accounts** need their own API Client — a client targets exactly one account.

## Related

- `authentication/kore-wireless-authentication.yml`
- `scopes/kore-wireless-scopes.yml`
- `conventions/kore-wireless-conventions.yml`
