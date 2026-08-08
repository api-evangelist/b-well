# b.well

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

b.well Connected Health is a digital health platform that unifies a person's fragmented
medical, pharmacy, claims, wearable and lab data into a single FHIR-native longitudinal
health record, then licenses that record to health systems, payers, employers and retail
health brands who embed it in their own applications.

- Website: https://www.icanbwell.com/
- Developer portal: https://developer.bwell.com/
- GitHub: https://github.com/icanbwell
- Status: https://status.bwell.com

## Public API surface profiled here

| Surface | Where |
| --- | --- |
| Application APIs (federated GraphQL + REST) | `https://api.client-sandbox.icanbwell.com/v1` |
| User Data Operations API (OpenAPI 3.0.3) | `openapi/b-well-user-data-operations-openapi.json` |
| Client Webhook API (OpenAPI 3.0.3, FHIR messaging) | `openapi/b-well-client-webhook-api-openapi.json` |
| FHIR R4 Server (open-source Helix FHIR Server) | `https://fhir.icanbwell.com` |
| Health SDK for AI — hosted MCP server, 31 agents | `mcp/b-well-mcp.yml` |

Both OpenAPI documents were extracted verbatim from the payload of b.well's own
ReadMe-hosted reference pages; b.well does not serve either from a downloadable spec URL.
