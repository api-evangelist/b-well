---
name: Honor a user's data export or deletion request
description: Initiate a personal health data export or an account deletion for a b.well user over the HMAC-signed User Data Operations REST API, and handle the asynchronous completion webhook.
api: openapi/b-well-user-data-operations-openapi.json
surface: REST
servers:
  - https://user-data-ops.client-sandbox.icanbwell.com
  - https://user-data-ops.prod.icanbwell.com
operations:
  - 'POST /users/{id}/data-exports'
  - 'DELETE /users/{id}'
  - 'POST /webhook (client-implemented callback)'
---

# Honor a user's data export or deletion request

This is **not an agent-callable flow**. Both operations are irreversible, run in system
context, and require an HMAC secret that must never reach a client or an LLM. Run them
from your backend on an explicit, verified user request.

## Signing the request

Every call is signed. Build the `Authorization` header as:

```
HMAC-SHA512 SignedHeaders=x-bwell-date;host;x-bwell-client-user-token;x-bwell-client-key;x-bwell-content-sha512&Signature=<hmac-sha512-signature>
```

The signed headers are:

| Header | Value |
| --- | --- |
| `x-bwell-date` | UTC timestamp in RFC 1123 format |
| `Host` | the DNS host of the request |
| `x-bwell-client-user-token` | your client-specific user authentication token |
| `x-bwell-client-key` | the client key b.well issued you |
| `x-bwell-content-sha512` | Base64-encoded SHA-512 hash of the request content |

See `authentication/b-well-authentication.yml`.

## Export a user's health data

`POST /users/{id}/data-exports`

`{id}` is the **client-specific** user identifier, which varies by client — confirm with
b.well support which identifier your integration sends. It is not necessarily the FHIR
person id used by the SDK and MCP surfaces.

- `202` — export request accepted, processing has started
- `401` — invalid HMAC signature; rebuild the canonical signed-header string and re-sign
- `404` — user not found
- `500` — server error

## Delete a user's account

`DELETE /users/{id}`

Same identifier space, same response set. This initiates deletion of the user's profile
and associated information from b.well's system. b.well documents configurable waiting
periods or immediate execution
(https://developer.bwell.com/docs/user-account-deletion-1) — confirm which your tenant
is configured for before calling.

## Handle completion

Both operations return `202` immediately and report completion asynchronously to the
webhook endpoint you registered with b.well. b.well POSTs a FHIR `Bundle` of
`type: message`; the `MessageHeader.eventCoding` identifies the event and
`focus[].reference` points at the subject. Identify your user by the
`urn:client:identifier` identifier on the `Person` resource in the bundle.

Return `200` on success. Return `400` for an invalid payload and `401` if the inbound
HMAC signature does not validate — **verify it before acting on the event**. See
`asyncapi/b-well-webhooks.yml`.

## Cautions

- There is **no documented idempotency contract**. Both operations return `202` and
  enqueue asynchronous work, so a blind retry may enqueue a second job. Record your own
  request key and do not retry a `202`.
- Deletion is irreversible. Require an explicit, authenticated user action; never let an
  agent trigger it.
