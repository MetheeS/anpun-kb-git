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

## [azure-table-list-entities-no-query-filter-kwarg]
created: 2026-06-10
tags: azure-data-tables, python-sdk, query, TypeError
symptom/context: Calling list_entities(query_filter="...", parameters={...}) raises TypeError: Session.request() got an unexpected keyword argument 'query_filter' — the exception propagates through the entire Azure pipeline to the HTTP transport layer.
root-cause: list_entities() passes unknown **kwargs straight through to the HTTP transport (requests.Session.request()). The query_filter + parameters API is specific to query_entities().
fix: Use query_entities(query_filter="...", parameters={...}) for any parameterized filtered scan. Reserve list_entities() for full-table unfiltered scans only.
failed-attempts: None — once the traceback was read the fix was immediate.

## [azure-table-upsert-on-nonexistent-table]
created: 2026-06-10
tags: azure-data-tables, python-sdk, ResourceNotFoundError, table-creation
symptom/context: First upsert_entity() call on a net-new table (never created) raises ResourceNotFoundError and crashes the service on startup.
root-cause: Azure Table Storage does not auto-create tables on upsert. The table must exist before any entity operation. On a fresh deployment the table is absent.
fix: Wrap the first write in try/except ResourceNotFoundError, call create_table() (or TableServiceClient.create_table_if_not_exists()), then retry. For reads (list, query), catch ResourceNotFoundError and return []. Pattern applies to any table that may not exist on first deploy.
failed-attempts: None.

## [azure-table-no-array-membership-filter]
created: 2026-06-12
tags: azure-tables, query, filter, array, odata
symptom/context: Need to query Azure Table Storage for rows where a
  stored JSON array field contains a specific value (e.g. visibleSites
  contains "site-a"). OData filter expressions on array fields either
  error or return no results.
root-cause: Azure Table Storage OData supports only flat-value comparisons
  (eq, ne, lt, gt, le, ge) on primitive entity properties. There is no
  array-membership predicate (IN, contains, any()). JSON arrays stored
  as strings cannot be queried by content — the filter sees only the raw
  serialized string.
fix: Fetch all candidate rows and filter in-memory after deserialization.
  If the dataset is large, partition by owner first (reducing rows fetched)
  then apply in-memory membership check.
recommendation: Do not rely on server-side filtering for array-valued
  properties in Azure Table Storage. Design the schema to support
  flat-value filters (e.g. one row per site-item pair) if server-side
  fan-out filtering is required at scale.
