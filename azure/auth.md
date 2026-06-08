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

## [msal-browser-cache-inject-realm]
created: 2026-06-03
tags: msal, entra, playwright, e2e, cache-injection
symptom/context: Playwright global-setup writes a full MSAL-browser v3 localStorage
  cache (account entry, token entries, msal.account.keys, msal.token.keys.<clientId>)
  via page.evaluate and saves storageState. Specs still land on the unauthenticated
  home; useIsAuthenticated() returns false; the app renders the Microsoft login button.
root-cause: AccountEntity.isAccountEntity() in @azure/msal-common requires the field
  `realm` (NOT `tenantId`) to validate an account entry. Without it,
  BrowserCacheManager.createKeyMaps() silently drops the account from the map;
  `accounts` stays empty; useIsAuthenticated() returns false.
  Also required: two active-account keys written before saving storageState —
  `msal.<clientId>.active-account-filters` (JSON {homeAccountId, localAccountId, tenantId,
  lastUpdatedAt}) and `msal.<clientId>.active-account` (= localAccountId).
fix: Add `realm: tid` to the injected account entry (keep `tenantId` too for the
  public AccountInfo shape the app reads via accounts[0]). Write both active-account
  keys before saving storageState. Account key format:
  `<homeAccountId>-<environment>-<tid>` (>= 3 hyphen-separated segments).
failed-attempts: Adding only the active-account keys — necessary but not sufficient;
  the `realm` field in the account VALUE was still missing.

## [entra-v1-v2-token-audience-issuer]
created: 2026-05-31
tags: entra, msal, jwt, obo, fastapi, v1-endpoint, v2-endpoint
symptom/context: FastAPI returns 401 "Invalid audience" on tokens that are valid.
  Tokens from az CLI / msal-python use the v1 Entra endpoint; tokens from
  msal-browser use the v2 endpoint.
root-cause: v1 tokens: aud = "api://<guid>", iss = "https://sts.windows.net/<tid>/".
  v2 tokens: aud = "<bare-guid>", iss = "https://login.microsoftonline.com/<tid>/v2.0".
  A validator that accepts only one form rejects valid tokens from the other.
fix: Accept BOTH audience forms — pass a list: [bare_guid, "api://" + bare_guid].
  Validate BOTH issuer patterns. In PyJWT: pass options={"verify_aud": False} and
  manually check aud and iss after decode.
failed-attempts: Accepting only the bare-GUID audience — az CLI issues api://<guid>
  audience (v1 endpoint); those tokens were rejected.

## [msal-python-browser-cache-schema]
created: 2026-06-02
tags: msal, playwright, e2e, localStorage, ropc, cache-schema
symptom/context: Acquiring a token via msal-python ROPC
  (acquire_token_by_username_password) and injecting it into browser localStorage
  for Playwright does not authenticate the msal-browser / React app.
root-cause: msal-python and @azure/msal-browser use different localStorage key
  schemas. msal-browser v3 requires specific key formats for account entries, token
  entries, and index keys. The `realm` field (not `tenantId`) must be present in the
  account value for AccountEntity.isAccountEntity() to accept the entry.
  See also [[msal-browser-cache-inject-realm]].
fix: In global-setup.ts, construct all localStorage keys manually from ROPC token
  claims (oid, tid, preferred_username, etc.). Required keys:
  - Account entry (with realm = tid, plus tenantId, homeAccountId, environment,
    localAccountId, username, authorityType)
  - Access-token entry keyed by clientId-authority-scope
  - Id-token entry
  - msal.account.keys (JSON array containing the account key)
  - msal.token.keys.<clientId> (JSON {accessToken:[...], idToken:[...], refreshToken:[]})
  - msal.<clientId>.active-account (= oid)
  - msal.<clientId>.active-account-filters (JSON {homeAccountId, localAccountId, tenantId})
