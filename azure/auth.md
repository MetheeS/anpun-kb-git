# azure — auth

## [azure-ad-ropc-graph-direct-scope]
created: 2026-06-06
tags: azure-ad, ropc, graph, token-scope, obo, e2e-testing
symptom: E2E test acquires ROPC token with app GUID .default scope (e.g.
  2e91669e-.../.default). Backend POST /auth/token returns 401 {"error":"graph_unauthorized"}.
root-cause: Backend calls Graph /me DIRECTLY (not via OBO). Token audience is the
  app's own GUID. Graph rejects a token not scoped to graph.microsoft.com — returns 401.
  OBO would exchange the token for a Graph-scoped one; direct-call does not.
fix: When the backend calls Graph /me directly (no OBO), the ROPC token MUST use
  scope https://graph.microsoft.com/.default. With OBO, use the app GUID .default
  scope (OBO handles the exchange).
failed-attempts: Using app GUID .default scope with a direct Graph call — Graph
  rejects the token (aud mismatch); 401 graph_unauthorized.
