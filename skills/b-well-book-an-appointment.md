---
name: Find a provider and book an appointment
description: Search the b.well provider directory, find open slots for a visit type, book the appointment as a FHIR Appointment resource, and reschedule or cancel it correctly.
api: mcp/b-well-mcp.yml
surface: MCP
server: https://api.{ENVIRONMENT}.icanbwell.com/mcp/
operations:
  - convert_zipcode_to_coordinates
  - provider_search
  - register_patient
  - get_visit_types
  - get_scheduling_slots
  - schedule_appointment
  - get_appointment_detail
  - get_upcoming_appointments
  - reschedule_appointment
  - get_cancel_reasons
  - cancel_appointment
---

# Find a provider and book an appointment

Scheduling is the one flow in b.well's tool catalogue where identifiers must be carried
forward unchanged from step to step. `visit_type_id`, `provider_id` and `department_id`
passed to `schedule_appointment` must be the *same values* used in
`get_scheduling_slots`, or the booking will not match the slot.

## Steps

1. **Resolve location.** If the user gave a ZIP code, call
   `convert_zipcode_to_coordinates` to get `lat`/`lon`. `provider_search` also accepts a
   raw `zipcode`, but coordinates give you distance sorting.
2. **Find the provider.** Call `provider_search` with `search_text` for a specialty
   ("cardiologist"), a facility ("Quest Diagnostics") or a name ("Dr. Smith"). Omit
   `search_text` for a general "providers near me". Tune with `distance` (miles,
   default 50), `page_size` (default 10) and `page_number` (0-based).
   `provider_search` is **read-only** — it does not establish a data connection.
   From the results keep the provider NPI (`provider_id`) and the `facility_id`, which
   becomes `department_id`.
3. **Register if required.** Call `register_patient` to register the patient with the
   scheduling-enabled organization before their first booking there.
4. **Get visit types.** Call `get_visit_types` for the provider/department and pick the
   `visit_type_id`.
5. **Get slots.** Call `get_scheduling_slots` with `visit_type_id`, `provider_id` and
   `department_id`. Keep the returned `timezone` value.
6. **Book.** Call `schedule_appointment` with:
   - `start` — ISO 8601 with a timezone offset, e.g. `2026-08-06T08:30:00-05:00`, built
     from the slot's `start_date` + `start_time`
   - `minutes_duration` — integer minutes
   - `visit_type_id`, `provider_id`, `department_id` — unchanged from step 5
   - `timezone_title` — the `timezone` from step 5. If omitted it defaults to
     `Central Standard Time`, which will be wrong for most patients. Always pass it.
   - `comment` — optional reason for the visit
   It creates a FHIR `Appointment` and returns a confirmation with an id.
7. **Confirm.** Call `get_appointment_detail` or `get_upcoming_appointments` to read
   back what was booked before telling the user it is done.

## Changing an appointment

- **Reschedule:** `reschedule_appointment`. Re-run `get_scheduling_slots` first — do not
  guess a new time.
- **Cancel:** call `get_cancel_reasons` for the valid reason set, then
  `cancel_appointment` with a reason from that set.

## Cautions

- Booking, rescheduling and cancelling are **write operations against a real clinical
  system**. Confirm the specific slot with the user before calling
  `schedule_appointment`.
- b.well documents **no idempotency key** on any of these tools. A retried
  `schedule_appointment` may create a second appointment. On a timeout, call
  `get_upcoming_appointments` to check whether the first attempt landed before retrying.
- Errors are FHIR `OperationOutcome`; see `errors/b-well-problem-types.yml`.
