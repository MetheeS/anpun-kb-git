# devops — shell

## [bash-dotenv-special-char-password]
created: 2026-06-08
tags: bash, dotenv, env-file, special-chars, E2E, ROPC, password
symptom/context: ROPC / API authentication fails with "invalid username or
  password" in E2E tests despite correct credentials in .env.test. Password
  contains ! or other shell-special characters ($, *, &, etc.).
root-cause: Sourcing .env files into bash (set -a; . ./.env.test or
  source .env.test) causes bash to expand shell-special characters in
  values: ! triggers history expansion, $ expands variables, etc. The
  exported value is silently corrupted. Additionally, set -u (nounset)
  in strict scripts can cause "unbound variable" if the file contains
  lines with unquoted special characters. If the corrupted value lands in
  process.env first, a test setup that checks "if NOT already in env"
  will use the corrupted value instead of re-reading the file.
fix: Do NOT source .env.test before invoking the test runner. Instead:
  1. Let the test setup (e.g. global-setup.ts) parse the file line-by-line
     itself (split on first '=', take raw value — no shell expansion).
  2. Run the test runner in a clean environment:
       env -i HOME=$HOME E2E_STACK_EXTERNAL=1 npx playwright test
     so existing (potentially corrupted) shell env vars cannot override
     file-parsed values.
  3. In shell scripts that must source .env, use single-quotes around
     values containing special characters, or use a parser that does not
     invoke bash expansion.
