# patterns — reconciler

## [reconciler-detector-time-ordered-store-duplicate]
created: 2026-06-08
tags: reconciler, detector, time-ordered-store, duplicate, dedup, azure-table
symptom: Users see duplicate notifications for the same logical event. Both records
  share the same (system_id, request_id, approver) but differ in occurred_at,
  old_status, or trigger_type (one from the reconciler, one from the event detector).
root-cause: A reconciler (ground-truth poller) and an event-detector can both write
  rows for the same logical key when they run in close succession. Different
  occurred_at timestamps → different RowKeys in a time-ordered store → two distinct
  visible rows. If the reconciler's dedup index stores only ONE row per logical key
  (dict[key] = entity, overwrites on collision), duplicate rows in the table are
  invisible to stale-cleanup and to subsequent dedup passes.
fix: Change the reconciler's existing-row index from dict[key→single_entity] to
  dict[key→list[entity]]. Stale cleanup must delete ALL rows per stale key (not
  just one). Add a dedup pass for still-pending keys: sort rows by RowKey, keep
  the newest (smallest RowKey in an inverted-ticks scheme), delete the rest.
  The reconciler self-heals on its next scheduled cycle with no manual intervention.
failed-attempts: n/a — root cause found on first trace.
