# azure — storage

## [public-blob-no-cache-control-stale-frontend]
created: 2026-06-22
tags: blob, storage, cache-control, browser-cache, frontend
symptom/context: A frontend that fetches a public-read blob directly (e.g.
  catalog.json served from a public container) shows STALE content for hours after
  the blob is updated.
root-cause: The blob was written with no Cache-Control header and
  Content-Type=application/octet-stream. Browsers HEURISTICALLY cache responses
  that lack explicit caching headers (especially large files), so updates stay
  invisible until the heuristic TTL expires.
fix: Write browser-fetched public blobs with the correct Content-Type
  (e.g. application/json) AND Cache-Control=no-cache — this forces ETag
  revalidation (304 when unchanged, fresh bytes when changed) instead of blind
  heuristic caching. Note: clients holding the old heuristically-cached copy need
  one hard refresh; thereafter no-cache keeps them current.
