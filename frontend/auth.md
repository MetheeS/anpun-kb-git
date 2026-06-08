# frontend — auth

## [postmessage-wildcard-origin-not-auto-match]
created: 2026-06-02
tags: postmessage, origin-validation, wildcard, iframe-auth
symptom: SPA with VITE_PARENT_ORIGINS=* silently drops all postMessage auth tokens
  even after the useEffect race was fixed. 10s timeout fires, auth-error shown.
root-cause: parseCsvEnv('*') produces ['*'] (length 1). isAllowedOrigin(origin, ['*'])
  calls ['*'].includes(origin) which is false for any real origin string. The literal
  string "*" in an array does not act as a wildcard — it only matches if the event
  origin is literally the string "*" (which it never is).
fix: Explicitly check if allowlist.includes("*") before the standard includes(origin):
    if (allowlist.includes("*")) return true   // explicit wildcard opt-in
    return allowlist.includes(origin)
failed-attempts: Adding a readiness flag for page.goto timing — unrelated; race was
  already closed. This was a separate origin-gate bug masked by the same symptom.
