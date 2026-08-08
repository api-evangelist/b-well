---
name: Connect a patient to a health data source
description: Discover a patient's providers, health plans, labs and pharmacies in the b.well network, capture consent, establish the data connection, and manage its lifecycle afterwards.
api: mcp/b-well-mcp.yml
surface: MCP + SDK
server: https://api.{ENVIRONMENT}.icanbwell.com/mcp/
operations:
  - provider_search
  - search_connections
  - connect_to_datasource
---

# Connect a patient to a health data source

Connections are the point of b.well: without them the longitudinal record is empty. This
flow is consumer-mediated, so **consent is a prerequisite, not a formality**.

## Steps

1. **Discover what exists.** Call `provider_search` to browse providers, practices,
   insurance entities, labs and pharmacies. Set `include_populated_proa_only: true` when
   you only want sources with PROA (patient right of access) connectivity — those are
   the ones that will actually return records.
2. **Check what is already connected.** Call `search_connections` before offering to
   connect anything, so you do not duplicate an existing connection.
3. **Capture consent.** Create the consent record before establishing the connection.
   Consents carry a `CategoryCode` — `PROA_ATTESTATION` requires an `organizationId`,
   and `DIRECT_IMPORT_RECORDS` (FHIR code `direct:import:records`, added in Kotlin SDK
   1.16.0) covers direct record import. See
   https://developer.bwell.com/docs/consents-typescript-sdk.
4. **Establish the connection.** Call `connect_to_datasource`. Many sources require an
   OAuth handoff — the SDK path returns an OAuth URL to open, and completion is signalled
   back by postMessage listener or URL polling
   (https://developer.bwell.com/docs/oauth-connection-flow). Where `integrationType` is
   `DIRECT`, credentials are not required and `username`/`password` may be omitted; for
   all other integration types they are required.
5. **Watch the state.** A connection moves through `DataConnectionStatus` /
   `SyncStatus` values and its data source reports an `endpointStatus`. The state
   machine is documented at
   https://developer.bwell.com/docs/connection-state-diagram.
6. **Retrieve records.** Once connected, use the record tools
   (`b-well-retrieve-health-records.md`) or `$everything` on the FHIR server.

## Managing connections afterwards

- **Disconnect** leaves the connection record; **delete** removes it. A disconnected
  connection can be reactivated.
- Data sources publish `consentPolicyUrl` and `consentValidityDuration` (an ISO 8601
  duration such as `P1Y`) as of Kotlin SDK 1.17.0 — re-consent before that duration
  lapses or the connection will stop returning data.
- Smart Connect / Individual Access Service locates records across networks (HIEs, HINs,
  TEFCA QHINs, CMS-aligned networks) rather than at a single named provider. See
  https://developer.bwell.com/docs/smart-connect-locating-managing-connections.

## Error handling

FHIR `OperationOutcome` issue types you will actually hit here:

- `SECURITY` — the user entered invalid credentials for the external source. Prompt
  again; do not retry the same credentials.
- `TIMEOUT` — the external connection service timed out. The connection may still be
  establishing; re-check with `search_connections` before retrying.
- `VALUE` — invalid `connectionId`, or an invalid `organizationId` on a
  `PROA_ATTESTATION` consent.
- `NOT_FOUND` — the connection does not exist (`getDataSource`), or the user is not
  connected to the connection being disconnected.

See `errors/b-well-problem-types.yml`.

## Cautions

- Connecting a data source moves a real person's medical records. Confirm the specific
  source with the user before calling `connect_to_datasource`.
- Test this against the b.well Sandbox PROA Server rather than a live health system —
  see `sandbox/b-well-sandbox.yml`.
