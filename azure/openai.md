# azure — openai

## [openai-sdk-httpx-proxies-pin]
created: 2026-06-01
tags: openai-sdk, httpx, proxies, version-pin, dependency, python
symptom/context: Every request through the openai Python SDK returns HTTP 500 with
  "Client.__init__() got an unexpected keyword argument 'proxies'". Fails immediately
  on first use; no requests reach the API.
root-cause: openai >= 1.51.0 passes proxies= to httpx.Client internally. httpx >= 0.28
  removed the proxies parameter. If httpx is unpinned (or pip resolves to >= 0.28),
  the openai client instantiation raises a TypeError on every call.
fix: Pin httpx==0.27.2 in requirements.txt. No resolver conflict with openai 1.51.x;
  no other common dependency forces httpx to >= 0.28.
  Check for resolution: openai==1.51.* + httpx==0.27.* must co-exist in pip freeze.

## [azure-openai-regional-endpoint-no-token-auth]
created: 2026-06-10
tags: azure-openai, managed-identity, entra-id, authentication
symptom/context: POST to Azure OpenAI images/generate returns 400 BadRequest — "Please provide a custom subdomain for token authentication, otherwise API key is required."
root-cause: Azure OpenAI regional endpoints (https://<region>.api.cognitive.microsoft.com/) do NOT support Entra ID / managed identity token auth. Only resources created with a custom subdomain (https://<name>.openai.azure.com/) accept Bearer token auth.
fix: Either (a) recreate the Azure OpenAI resource WITH a custom subdomain and update the endpoint env var, or (b) fall back to API key auth by setting the key as an env var and switching the client to key auth when the var is non-empty.
failed-attempts: Updating managed identity role assignments (Cognitive Services OpenAI User) did not help — the endpoint format is the root issue, not RBAC.

## [openai-sdk-gpt-image-1-min-version]
created: 2026-06-10
tags: azure-openai, openai-sdk, gpt-image-1, python
symptom/context: POST to Azure OpenAI images/generate with gpt-image-1 returns 400 or TypeError on output_format / quality params.
root-cause: openai SDK < 1.74.0 predates gpt-image-1 support. output_format is not a recognized parameter in older versions, and quality="low" is rejected by the Azure endpoint.
fix: Pin openai >= 1.74.0 in requirements.txt.
failed-attempts: Passing unsupported params via extra_body workaround — partially works but not stable.
