# testing — playwright

## [playwright-route-last-registered-wins]
created: 2026-06-12
tags: playwright, e2e, routing, intercept, mock
symptom/context: E2E tests mock specific API routes via page.route() but
  a broad catch-all handler (e.g. **/api/**) intercepts them first,
  returning wrong data and causing all tests to fail.
root-cause: Playwright evaluates page.route() handlers in
  LAST-registered-first order. A catch-all registered after specific
  handlers will shadow them.
fix: Register the broad catch-all FIRST, then register specific
  handlers. Specific routes registered last win over earlier catch-alls.
  Example:
    page.route('**/api/**', genericFallback);   // registered first = lowest priority
    page.route('**/api/my-roles**', rolesMock); // registered last = wins
recommendation: Always register catch-alls before specifics in Playwright
  test setup. Apply the same order in beforeEach/beforeAll blocks.

## [playwright-getbytext-descendant-text-concat]
created: 2026-06-12
tags: playwright, e2e, locator, getbytext, selector
symptom/context: getByText(regex) returns unexpected matches — a loose
  pattern like /shared/i matches a container element because the item
  name inside is "Shared Dish", causing a false-positive even when the
  intended badge is absent.
root-cause: getByText(regex) tests the regex against an element's
  NORMALIZED text content, which for a container is the concatenation
  of ALL descendant text nodes. A regex that matches any substring of
  the concatenated text will match the container, not just the target
  element with that specific text.
fix: Tighten the regex to a phrase that only appears in the target
  element and cannot span unrelated siblings. Prefer exact=true or
  a unique substring that no parent's concatenated text would produce
  accidentally.
  Bad:  getByText(/shared/i)          // matches any element containing "Shared"
  Good: getByText(/shared from \w+/i) // phrase unique to the badge element
recommendation: Treat getByText as a full-subtree text search, not a
  leaf-node match. When testing for a specific UI badge or label,
  use a phrase-level regex or combine with a role/test-id selector.

## [playwright-storagestate-not-reapplied-on-navigation]
created: 2026-06-24
tags: playwright, auth, storagestate, e2e
symptom: E2E injects an auth token into localStorage (storageState/global-setup), removes it to assert a signed-out view, then navigates expecting signed-in again — the second navigation stays signed-out.
root-cause: Playwright applies storageState once per browser CONTEXT (at creation), not on every page.goto. Removing the token in-page and re-navigating does not re-apply it.
fix: Re-seed the token in the run harness for the re-nav step — page.addInitScript to set the localStorage key on next load, or set it explicitly before the second goto. Harness fix; no assertion change.
