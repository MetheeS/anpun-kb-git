# sap — nwbc

## [sap-nwbc-window-open-popup-async]
created: 2026-06-19
tags: sap, nwbc, playwright, popup, async, window.open
symptom/context: SAP NWBC forms open popup windows (e.g. PDPA consent,
  service order entry) via window.open(). Popup arrives AFTER networkidle —
  clicking into the popup immediately after a button click hits the wrong page.
root-cause:
  SAP fires window.open() asynchronously via XHR callback, AFTER
  wait_for_load_state("networkidle") resolves. Checking for a pending popup
  only in the on-click handler misses the window — it has not opened yet.
fix:
  1. Register context.on("page", handler) at Runner init to capture ALL
     new pages into a _pending_popup slot.
  2. At the START of every DSL step execution (_exec()), call
     _switch_to_popup_if_pending() BEFORE dispatching the verb handler.
     This catches the popup that arrived asynchronously after the previous
     step's networkidle settled.
  3. Track whether page was switched; switch back to main page on popup close.

  NOT sufficient: checking for popup only after successful clicks, or only
  on specific verbs. SAP XHR timing means the popup may not exist yet when
  the current step completes.
failed-attempts:
  - Listening for popup only after click() → popup arrives too late (after
    networkidle), not captured.
  - Checking context.pages at end of step → race condition; popup not yet open.
