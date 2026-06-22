# frontend — search

## [fuzzy-search-whole-text-overmatch]
created: 2026-06-22
tags: search, fuzzy, subsequence, ux, frontend
symptom/context: A client-side fuzzy search (subsequence + typo matching) over
  catalog names + descriptions matched nearly EVERY record for short queries —
  e.g. typing "MFG" matched 44/44 databases; the search appeared to "not filter".
root-cause: The in-order subsequence (and typo) matcher ran against the WHOLE
  multi-word text, including long descriptions. Any 2-3 char query is trivially a
  subsequence scattered across the words of almost any sentence; short queries
  also fuzzy-expand to common words (mfg -> manufacturing) that appear everywhere.
fix: (1) Scope subsequence/typo matching to individual WORD TOKENS (split on
  non-alphanumeric, unicode-aware), NOT the whole string — keep plain substring
  matching against the whole text so multi-word phrases still match. (2) Gate
  fuzzy expansion behind query.length >= 4; 1-3 char queries match by
  exact/substring only (an acronym means the literal token). This keeps
  custmr->customer and invc->invoice while eliminating the short-query blowup.
