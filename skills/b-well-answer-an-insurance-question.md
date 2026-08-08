---
name: Answer a patient's insurance question
description: Establish which benefit plan a patient has and whether it is active before answering any coverage, copay, deductible or benefits question, then search the policy documents for the answer.
api: mcp/b-well-mcp.yml
surface: MCP
server: https://api.{ENVIRONMENT}.icanbwell.com/mcp/
operations:
  - validate_insurance_coverage
  - list_available_insurance_docs
  - explore_policy_context
  - search_insurance_policy
---

# Answer a patient's insurance question

b.well's insurance tools have a mandatory ordering: a plan question cannot be answered
correctly without first naming the plan and its coverage period. Do not skip step 1.

## Before you start

Every call runs in an authorized end-user context. You need a b.well user access token
obtained through OAuth 2.0 token exchange. Attach it as the MCP server's
`authorization_token`. `person_id` is optional on every tool below — when omitted it is
resolved from the `clientFhirPersonId` claim on the token.

## Steps

1. **Establish coverage first.** Call `validate_insurance_coverage`. Read
   `coverage_state` from the result — it is the routing signal for everything that
   follows:
   - `no_coverage_no_docs` — **stop**. Tell the user no coverage or documents were
     found. Do not call `list_available_insurance_docs` or `search_insurance_policy`.
   - `verified` — proceed; no disclaimer needed.
   - `unverified_docs_exist` or `expired_docs_exist` — proceed, but surface the
     `warning` string to the user before answering.
2. **Name the plan.** Use `benefit_plan_name`, `period_start` and `period_end` from the
   returned `coverages[]`. A `period_end` of `None` means ongoing coverage. `order: 1`
   is primary coverage, `2` is secondary.
3. **Find the documents.** Call `list_available_insurance_docs` to identify which policy
   documents exist for this patient.
4. **Get the context.** Call `explore_policy_context` to understand the structure of the
   relevant policy before searching it.
5. **Search for the answer.** Call `search_insurance_policy` with the user's question.
6. **Answer with the plan named.** Always state which plan and which coverage period the
   answer applies to.

## Error handling

Failures arrive as FHIR `OperationOutcome` resources. Check `issue[].code`:
- `SECURITY` or `EXCEPTION` with a null access token — the token has expired; re-run
  token exchange (see `authentication/b-well-authentication.yml`).
- `NOT_FOUND` — the person id could not be resolved.
See `errors/b-well-problem-types.yml` for the full issue-type catalogue.

## Cautions

- The `warning` field exists because coverage data can be stale. Never present an
  unverified plan answer as authoritative.
- b.well documents no rate limits and no idempotency contract. These are read-only
  tools, so retries are safe, but do not assume a published retry budget.
