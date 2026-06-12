# frontend — fetch

## [fastapi-streaming-response-client-timeout]
created: 2026-06-10
tags: fastapi, fetch, streaming, timeout, abortcontroller, react
symptom/context: Frontend calls a FastAPI endpoint that returns StreamingResponse
  (e.g., a long-running NL→SQL pipeline). Under certain conditions (cold cache,
  slow backend, stalled future) the backend stalls mid-stream but keeps the HTTP
  connection alive by sending CRLF keep-alive bytes. The frontend awaits res.json()
  with no timeout — it blocks indefinitely, leaving the UI in a permanent loading
  state with no error surfaced.
finding: Browser fetch has no built-in timeout. FastAPI StreamingResponse holds the
  connection open as long as the generator yields — even if it only yields keep-alive
  bytes. await res.json() resolves only when the stream closes. If the backend stalls
  before the final JSON chunk, res.json() never resolves and no exception is thrown
  unless the user navigates away.
fix: Wrap the fetch in an AbortController and set a timeout matching the server-side
  gateway cap (Azure Container Apps = 240s):
    const controller = new AbortController();
    const tid = setTimeout(() => controller.abort(), 240_000);
    try {
      const res = await fetch(url, { signal: controller.signal, ... });
      const data = await res.json();
    } catch (e) { /* AbortError or network error — surface to user */ }
    finally { clearTimeout(tid); }
  Always clear the timeout in finally to avoid phantom aborts after success.
  240s is the ACA gateway cap — tune to match your own infra's gateway timeout.
recommendation: Any fetch to a streaming endpoint must have an AbortController timeout.
  Match it to the infrastructure timeout cap (ACA: 240s, ALB: varies). Never rely on
  the browser to surface a stalled stream as an error — it won't.

## [axios-formdata-array-detail-object-object]
created: 2026-06-12
tags: axios, formdata, fastapi, error-handling, interceptor
symptom/context: File upload via axios FormData POST fails and the error
  displayed to the user is literally "[object Object]" — no useful message.
root-cause: Two compounding bugs:
  (1) FastAPI validation errors (422) return detail as an array of objects
      [{loc, msg, type}]. An axios response interceptor doing
      String(error.response?.data?.detail) on an array produces "[object Object]".
  (2) An axios instance with a default Content-Type: application/json header
      may send that header on FormData POSTs, causing FastAPI to reject the
      multipart body with a 422 (triggering bug 1).
fix:
  In the response interceptor, handle array detail:
    const raw = error.response?.data?.detail;
    const message = Array.isArray(raw)
      ? raw.map((d) => d.msg ?? JSON.stringify(d)).join("; ")
      : (raw != null ? String(raw) : null) ?? error.message ?? "Unknown error";
    return Promise.reject(new Error(message));

  On FormData POST requests, clear the Content-Type so the browser sets
  multipart/form-data with the correct boundary:
    apiClient.post("/upload", formData, { headers: { "Content-Type": undefined } });
recommendation: Any axios client that defaults Content-Type: application/json
  must pass { headers: { "Content-Type": undefined } } on FormData uploads.
  Any FastAPI + axios integration must handle array-shaped detail in the
  error interceptor.
