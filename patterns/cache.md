# patterns — cache

## [fingerprint-cache-traps-failed-result]
created: 2026-06-22
tags: cache, idempotency, change-detection, llm, fingerprint
symptom/context: After fixing the root cause of a failed computation (e.g. an LLM
  enrichment step that had been falling back to a placeholder), the bad results
  never refreshed — they stayed wrong on every subsequent run.
root-cause: The pipeline cached results keyed by a content FINGERPRINT and stored
  the fingerprint EVEN WHEN the computation failed and produced a fallback value.
  On the next run the fingerprint matched -> the entry was treated as "already
  done" -> it was never retried, even though the original cause was now fixed.
fix: Do NOT store the fingerprint / mark success when a computation falls back or
  fails — cache only genuine successes, so a failed entry naturally re-processes
  next run. If you cannot avoid storing it, provide explicit invalidation (delete
  the fingerprints of failed/fallback entries) and run it after fixing the cause
  to force a recompute.
