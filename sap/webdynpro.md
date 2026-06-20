# sap — webdynpro

## [sap-webdynpro-playwright-locator-patterns]
created: 2026-06-17
tags: sap, webdynpro, playwright, locator, msal, iframe
symptom/context: Automating SAP NWBC Webdynpro ABAP apps with Playwright —
  element IDs are dynamic (WD01, WD021E prefix) and change every reload.
  MSAL tokens not captured by Playwright storageState. Timing failures
  on interactive steps.
finding:
  - Dynamic IDs: avoid CSS/ID selectors. Use getByRole()/getByLabel()/
    getByPlaceholder()/getByText() — these survive reloads.
  - MSAL stores tokens in sessionStorage by default; Playwright's
    storageState only captures localStorage. Session reuse requires
    configuring MSAL to use localStorage OR re-running login each test.
  - NWBC embeds Webdynpro inside iframes. Must search self.page.frames
    for form fields — top-level page locators miss them.
  - Each button click triggers a full SSR POST → full page reload.
    Use wait_for_load_state("load") after clicks; networkidle for AJAX.
  - SAP uses <label f='ID'> (not <label for='ID'>) — requires custom
    label resolution (query f attribute, not for).
recommendation:
  - Primary selector hierarchy: getByRole(name=) > getByLabel() >
    getByPlaceholder() > getByText(). Never hard-code WD* IDs.
  - For frame-aware field search: iterate page.frames, try each frame's
    label/placeholder/text locators.
  - For session reuse: save storageState after login; only works if MSAL
    configured for localStorage.
  - Add wait_for_load_state("networkidle") after actions that fetch data;
    "load" after navigation actions.

## [sap-webdynpro-file-input-hidden]
created: 2026-06-18
tags: sap, webdynpro, playwright, file-upload, filechooser
symptom/context: SAP Webdynpro FileUploadElement: clicking the Upload button
  does not open a detectable file input. locator('input[type="file"]').count()
  returns 0 across all frames. set_input_files() raises "element not found."
root-cause:
  SAP renders <input type="file"> hidden (display:none). Playwright's
  locator.count() returns 0 for display:none elements (no bounding box).
  The element IS in the DOM; the visibility check is the wrong gate.
  Additionally, SAP triggers the file chooser event asynchronously via
  JavaScript — the hidden input fires a native filechooser event on click,
  not a direct DOM interaction.
fix:
  Use page.expect_file_chooser() context manager BEFORE the click that
  triggers the upload button. Playwright intercepts the filechooser event
  regardless of whether the underlying input is hidden:

    with page.expect_file_chooser() as fc_info:
        upload_button.click()
    file_chooser = fc_info.value
    file_chooser.set_files(filepath)

  Do NOT rely on locator.count() > 0 to gate file inputs — hidden inputs
  return 0. set_input_files() works on hidden inputs if the element exists,
  but the filechooser context manager is more reliable for SAP.
failed-attempts:
  - locator('input[type="file"]').count() > 0 gate → 0 for hidden inputs.
  - fr.locator().first.set_input_files() → raises if element absent.
  - Checking all frames iteratively → input absent from self.page.frames
    (not a popup; SAP fires filechooser from hidden input via JS).
