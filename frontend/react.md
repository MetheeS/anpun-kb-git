# frontend — react

## [react-postmessage-useeffect-race]
created: 2026-06-02
tags: react, postmessage, useeffect, race-condition, iframe
symptom: React SPA embedded in an iframe receives a window.postMessage auth token,
  but the handler never fires. App hangs at "Signing in…" then shows auth-error.
  Negative/error cases pass; valid tokens are silently dropped.
root-cause: window.addEventListener("message", handler) is registered inside a React
  useEffect, which runs after component mount. The parent posts the token immediately
  after page.goto() / iframe load — before React mounts — so the event is dropped
  with no listener registered.
fix: Register a buffer listener at module-eval time (before createRoot), e.g.
  installEarlyAuthCapture() called in main.tsx before render(). On mount, drain the
  buffer via consumeBufferedAuthMessage(). Apply the same validation (aud, origin)
  to both buffered and live messages.
failed-attempts: Adding a readiness flag — rejected; page.goto() already waits for
  load so module scripts execute before goto returns, making a flag redundant. The
  race is at the message-listener level, not the page-load level.
