---
name: Retrieve a patient's health records
description: Pull demographics, labs, vitals, clinical notes and typed FHIR resources for an authenticated patient, choosing the right tool per question and filtering by date, code and text.
api: mcp/b-well-mcp.yml
surface: MCP
server: https://api.{ENVIRONMENT}.icanbwell.com/mcp/
operations:
  - get_demographics
  - get_patient_summary
  - get_labs
  - get_vitals
  - get_fhir_record
  - search_clinical_notes
  - get_health_scores
  - get_wearables_summary
---

# Retrieve a patient's health records

## Before you start

All calls run in an authorized end-user context with a b.well user access token.
`person_id` is optional everywhere below; when omitted it is extracted from the
`clientFhirPersonId` claim on the token. If it cannot be resolved either way, the tool
returns a structured error rather than throwing.

## Choosing the right tool

| The user asks about | Call |
| --- | --- |
| who they are, their name, contact details | `get_demographics` |
| an overall picture of their health | `get_patient_summary` |
| blood tests, panels, CBC, A1c, cholesterol | `get_labs` |
| blood pressure, heart rate, weight, temperature | `get_vitals` |
| immunizations, conditions, medications, or any single FHIR resource type | `get_fhir_record` |
| something a clinician wrote | `search_clinical_notes` |
| their b.well health score | `get_health_scores` |
| wearable / device data | `get_wearables_summary` |

## Steps

1. **Set context.** Call `get_demographics` first when personalizing. Respect the
   PRIVACY / BEHAVIOR RULES block it returns: do not proactively reveal phone, email or
   full address — surface those only when the user explicitly asks or an identity
   verification task requires it.
2. **Query labs precisely.** `get_labs` takes exact lab names. Use full test names, not
   abbreviations (`'Complete Blood Count'` rather than "neutrophils"; `'CRP'` rather
   than "c-reactive protein"). Invalid names are silently ignored — and if *every* name
   is invalid, all labs come back instead, which looks like success. Verify the result
   matches what was asked.
3. **Bound the window.** `start_date` and `end_date` take `YYYY-MM-DD` and are
   inclusive. Omit for no bound.
4. **Raise `count` for history.** `count` is the number of raw FHIR records fetched
   *before* filtering, with a minimum of 300 enforced. For "have I ever…" questions,
   set 1000–5000 or the answer will be silently truncated.
5. **Pick output formats.** `get_labs` accepts `output_formats` of `table`, `timeline`,
   `statistics`, `csv`, `markdown`; the default is `["table", "statistics"]`.
6. **Use `text_filters` on `get_fhir_record`.** Pass several variations of a medical
   term (full name, abbreviation, synonyms) to prioritize matching records. Do not use
   it for vague or summary questions.
7. **Debug only when needed.** `debug: true` adds the FHIR request URLs, timing and a
   `request_id` — useful when reconciling with b.well support.

## Error handling

Errors are FHIR `OperationOutcome` resources. `NOT_FOUND` means the person id did not
resolve; `EXCEPTION` after authentication generally means a null or expired access
token. See `errors/b-well-problem-types.yml`.

## Cautions

- `get_demographics` performs a live FHIR fetch on every call — it is deliberately not
  cached. Do not call it in a loop.
- The other record tools accept `ignore_cache`; leave it false unless you have just
  written data.
