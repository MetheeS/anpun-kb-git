# azure — frontdoor

## [azure-frontdoor-origin-reachability-not-validated]
created: 2026-06-24
tags: azure, frontdoor, provisioning, testing
symptom/context: Tried to force an App into "Failed" in E2E by binding a route to an unresolvable backend host (*.invalid), expecting provisioning to fail.
finding: Azure Front Door Standard (Microsoft.Cdn AFD) does NOT validate origin reachability/DNS at provision time. An origin pointing at a non-resolving host still reaches provisioning_state=Succeeded (derives to Active). Failed only comes from ARM sub-step/control-plane errors — never from a well-formed-but-unreachable host.
recommendation: Don't induce "Failed" via a bad host — it won't fail. Test Failed/Retry UI paths via a mock/route-interception fixture (or a real ARM error); cover real Failed transitions in the backend regression corpus.
