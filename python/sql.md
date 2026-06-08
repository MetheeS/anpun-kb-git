# python — sql

## [sqlglot-tsql-dateadd-zero-anchor]
created: 2026-06-01
tags: sqlglot, tsql, t-sql, dateadd, parser-bug, guard, version-pin
symptom/context: A SQL guard using sqlglot with dialect="tsql" to parse, validate,
  and inject TOP N caps on T-SQL queries silently returns an uncapped SELECT (or
  incorrectly rejects a valid read-only SELECT) when the query uses the T-SQL
  "0-as-date-anchor" idiom: DATEADD(QUARTER, DATEDIFF(QUARTER, 0, GETDATE()), 0).
  This breaks a "never emit uncapped SQL" safety invariant.
root-cause: sqlglot <= 25.16.0 misparses the 3-argument DATEADD() form when the
  third argument is the integer literal 0. It disambiguates it as a 2-argument
  date_add() call, corrupting the AST. The TOP-injection round-trip then either
  rejects the query or re-emits it without the injected TOP cap. Fixed in sqlglot
  v30.8.0 (GitHub issue #4520: "disambiguate 2-arg date_add from 3-arg dateadd").
fix: Upgrade sqlglot to >= 30.8.0.
  Additionally, harden the guard with post-injection verification: after emitting
  the "safe" SQL, re-parse it and confirm (a) single statement, (b) outermost node
  is SELECT, (c) a TOP or LIMIT cap is present. If any check fails, REJECT the
  query (fail-closed) — never return the original uncapped SQL as "allowed".
  This defense-in-depth catches any future sqlglot regressions on edge-case T-SQL.
