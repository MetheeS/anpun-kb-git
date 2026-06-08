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
