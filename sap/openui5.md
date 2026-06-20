# sap — openui5

## [sap-openui5-tab-role]
created: 2026-06-19
tags: sap, openui5, playwright, tab, aria, role
symptom/context: Clicking SAP OpenUI5 tab bar items silently fails or
  selects wrong element. getByRole("button") and getByRole("link") find
  nothing; getByText() hits background elements instead.
root-cause:
  SAP OpenUI5 renders tab bar items with role="tab" (ARIA tablist pattern),
  not role="button" or role="link". Playwright's getByRole("button", name=)
  and getByRole("link", name=) both return no matches, causing fall-through
  to text-based locators that may match unintended background elements.
fix:
  Add getByRole("tab", name=text) as an explicit getter BEFORE the text
  fallback in click resolution:

    # In _find_clickable / _collect_clickable:
    loc = frame.get_by_role("tab", name=text, exact=True)
    if loc.count() > 0:
        return loc.first

  Place this after getByRole("button") and getByRole("link") checks,
  before getByText() fallback.
failed-attempts:
  - getByRole("button", name=tab_label) → 0 matches on OpenUI5 tab bars.
  - getByRole("link", name=tab_label) → 0 matches.
  - getByText(tab_label) → matches but may hit background page elements
    (e.g. filter checkboxes with same label text).
