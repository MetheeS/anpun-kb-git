# azure — sdk

## [azure-sdk-resource-not-found-status-none]
created: 2026-06-04
tags: azure-sdk, azure-table, python, exception-handling, http-error
symptom: DELETE/GET on a missing Azure Table row returned HTTP 500 instead of 404.
  Code had `except HttpResponseError as e: if e.status_code == 404: return 404`.
root-cause: ResourceNotFoundError is a subclass of HttpResponseError, but when
  bare-constructed its status_code attribute is None (not 404). The status_code==404
  branch was skipped, execution fell through to the 500 handler.
fix: Add an explicit `except ResourceNotFoundError` clause BEFORE the
  `except HttpResponseError` clause. Python exception matching is ordered —
  the more specific subclass must come first.
failed-attempts: Checking status_code==404 inside the HttpResponseError branch —
  fails because status_code is None for ResourceNotFoundError instances.
