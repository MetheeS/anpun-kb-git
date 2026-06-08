# anpunkit-kb — guide

This repo accumulates resolved issues and research findings from anpunkit projects.
Populated by `/store-wisdom`. Read by the anpunkit `researcher` agent at session start.

## Entry format — issues
## [slug]
created: YYYY-MM-DD
tags: tag1, tag2
symptom: what was observed
root-cause: the real underlying cause
fix: exact solution
failed-attempts: what did not work

## Entry format — research
## [slug]
created: YYYY-MM-DD
tags: tag1, tag2
symptom/context: what prompted the research
finding: what was discovered
recommendation: what to do

## INDEX.md format
YYYY-MM-DD | domain/file | slug | one-sentence summary

## Staleness
Research entries older than 6 months are flagged [STALE] at session load.
Stale entries are re-researched locally and rewritten via /store-wisdom.
Issue entries never go stale.
