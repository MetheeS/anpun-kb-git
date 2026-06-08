# sdp — api

## [sdp-v3-fields-required-whitelist]
created: 2026-05-31
tags: sdp, sdp-v3, api, fields-required, watermark, 400-error
symptom: SDP v3 GET /requests list query returns rows but last_updated_time is always
  null. Watermark never advances. sort_field and search_criteria on last_updated_time
  appear to have no effect (results unsorted/unfiltered).
root-cause: SDP v3 list view returns a DEFAULT field set that omits last_updated_time
  (and other non-default fields). sort/search silently no-op on absent fields.
  To get a field, it must be whitelisted in list_info.fields_required.
fix: Add list_info.fields_required: ["id","status","subject","requester","last_updated_time"]
  (and any other fields needed). Do NOT include "approver" — SDP has no approver column
  on the list view and returns 400 {"status_code":4001,"message":"Invalid Input",
  "fields":["approver"]} for the entire request.
failed-attempts: Deriving fields_required from every field_map value — fails because
  approver field_map key maps to "approver.email_id" and SDP rejects "approver" as a
  list-view column.
