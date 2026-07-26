---
name: Exchange machine-to-machine messages with devices
description: Send and track SMS and IP/UDP commands to IoT devices across KORE's Super
  SIM, Programmable Wireless and SMS APIs, and read back their delivery state.
api: openapi/kore-wireless-supersim.yml
also_uses:
  - openapi/kore-wireless-programmable-wireless.yml
  - openapi/kore-wireless-sms.yml
operations:
  - CreateSmsCommand
  - ListSmsCommand
  - FetchSmsCommand
  - CreateIpCommand
  - ListIpCommand
  - FetchIpCommand
  - ListSimIpAddress
  - sendCommand
  - listCommands
  - getCommand
  - deleteCommand
generated: '2026-07-25'
method: generated
source: openapi/kore-wireless-supersim.yml, openapi/kore-wireless-programmable-wireless.yml,
  openapi/kore-wireless-sms.yml
---

# Exchange machine-to-machine messages with devices

KORE has three distinct device-messaging surfaces. Pick the one that matches the
connectivity product the device is on — they are not interchangeable.

| Product | Host | Operations |
|---|---|---|
| Super SIM | `supersim.api.korewireless.com` | `CreateSmsCommand`, `CreateIpCommand` |
| Programmable Wireless | `programmable-wireless.api.korewireless.com` | `sendCommand` |
| KORE SMS | `api.korewireless.com/sms` | `POST /v1/messages/send` (no operationId declared) |

All of it is device-directed M2M messaging. None of it is an A2P/CPaaS messaging
product — do not use it to text people.

Authorize first — see `skills/kore-wireless-authorize-and-manage-api-clients.md`.

## Super SIM — SMS commands

1. `CreateSmsCommand` (`POST /v1/SmsCommands`) sends an SMS payload to a Super SIM.
2. `FetchSmsCommand` (`GET /v1/SmsCommands/{Sid}`) reads its delivery state.
3. `ListSmsCommand` filters the history by SIM and status.

Error **33203 Messaging not allowed** means the Fleet's settings forbid SMS.

## Super SIM — IP commands

1. `ListSimIpAddress` (`GET /v1/Sims/{SimSid}/IpAddresses`) tells you the IP the SIM
   currently holds.
2. `CreateIpCommand` (`POST /v1/IpCommands`) sends an IP/UDP payload to the device.
3. `FetchIpCommand` / `ListIpCommand` track it.

Expect **83401** if the device was not attached to a cellular network at send time,
and **83402** if your callback endpoint returned an error response to the command
callback. Neither is a client bug — both are real-world device/network conditions the
caller must handle.

## Programmable Wireless — commands

`sendCommand` (`POST /v1/Commands`) sends a command to a Programmable Wireless SIM;
`getCommand`, `listCommands` and `deleteCommand` manage the history. Watch for
**33111 Command exceeded max length** and **33118 Number of commands exceeded**.

## Rules

- **Asynchronous by nature.** A 2xx on the create call means KORE accepted the
  command, not that the device received it. Always fetch the command back, or
  subscribe to the event surface, before reporting delivery.
- **Callbacks**: if the command carries a callback URL, that endpoint must be HTTPS
  with a valid certificate, must verify `kore-signature`, and should deduplicate on
  `kore-idempotency-token` — see
  `asyncapi/kore-wireless-event-streams-webhooks.yml`.
- **No request-side idempotency key exists.** A blind retry of a send can deliver the
  message twice; list the command history first.
- **Errors**: `{status, message, code, more_info}` — see
  `errors/kore-wireless-error-codes.yml`.

## Related

- `conventions/kore-wireless-conventions.yml`
- `sandbox/kore-wireless-sandbox.yml`
