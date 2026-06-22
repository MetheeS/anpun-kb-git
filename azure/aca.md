# azure — aca

## [aca-vnet-config-immutable]
created: 2026-05-30
tags: aca, container-apps, vnet, immutable, recreate
symptom/context: Need to add VNet integration to an existing Azure Container Apps
  managed environment (e.g. to reach an Azure SQL MI private endpoint).
root-cause: vnetConfiguration on an ACA managed environment is IMMUTABLE — it can
  only be set at creation time. There is no in-place edit; any change requires
  deleting and recreating the entire environment and all container apps within it.
fix: Plan for VNet-integrated ACA from the start (Phase 0 / infra provisioning).
  If retrofitting: recreate the env with vnetConfiguration set, redeploy all
  Container Apps, and perform the post-recreate checklist (see
  [[aca-fqdn-changes-on-env-recreate]]). Minimum subnet: /27, delegated to
  Microsoft.App/environments. Peering: bidirectional (allowForwardedTraffic=true
  on both sides). Keep external ingress (internal=false) so users still reach
  the API publicly.

## [aca-fqdn-changes-on-env-recreate]
created: 2026-05-30
tags: aca, fqdn, cors, entra, env-recreate, secrets
symptom/context: After recreating an ACA managed environment (e.g. to add VNet
  integration), API calls from the frontend return CORS errors; Entra login fails;
  container secrets (e.g. OBO client secret) are missing.
root-cause: The ACA public FQDN (<app>.<random-suffix>.<region>.azurecontainerapps.io)
  is assigned per environment. Recreating the environment assigns a NEW random suffix
  even if the container app name is unchanged. Container secrets stored on the app
  are also lost on env recreate.
fix: After every ACA env recreate, update all of:
  1. ALLOWED_ORIGINS / CORS config on the container app env var.
  2. SWA CORS settings (if using Azure Static Web Apps).
  3. Redirect URIs on both frontend SPA and backend Entra app registrations.
  4. Re-set all container secrets (e.g. obo-client-secret) — they are lost.
  5. Update any hardcoded FQDN references in downstream config / tests.

## [aca-in-process-cache-multi-instance]
created: 2026-06-03
tags: aca, cache, multi-instance, load-balancing, in-process, scale
symptom/context: Warm-cache tests flap: a second request returns as slowly as the
  first (cold) request. Or an in-process cache is populated on one replica but
  subsequent requests see a cold cache.
root-cause: ACA auto-scales and load-balances across replicas. In-process data
  structures (Python dicts, module-level vars) are per-replica — not shared across
  the fleet. A "warm" hit is only guaranteed if both requests reach the same replica.
fix: For caches that must be fleet-wide (session state, rate-limit counters): use
  an external store (Redis, Azure Table Storage). For caches where eventual per-
  replica warm-up is acceptable (schema introspection, catalog): accept the behavior,
  document the cold-first-request cost per replica, and do NOT assert strict warm/cold
  speedup ratios in tests. Assert an absolute improvement instead (e.g. warm < cold
  - N seconds) or skip with INFRA marker if load-balanced to a different replica.

## [aca-job-stale-image-not-updated-by-containerapp-update]
created: 2026-06-22
tags: aca, container-apps-jobs, cicd, deploy, stale-image
symptom/context: A code fix was built and deployed (CI green, the container APP
  updated to the new image), but a scheduled ACA JOB kept running the OLD logic —
  the job failure persisted after the "fix" shipped.
root-cause: `az containerapp update` (and CI pipelines that only run it) update
  the container APP only. ACA JOBS (Microsoft.App/jobs) are SEPARATE resources
  that keep their own pinned image; an app update does not touch them.
fix: CI must update each job explicitly — `az containerapp job update --image
  <same image:tag>` — as its own deploy step, for every ACA Job that shares the
  app image. Add a dedicated "Deploy ACA Jobs" step to the pipeline so the jobs
  and the app ship together.

## [aca-job-replica-timeout-no-partial-progress]
created: 2026-06-22
tags: aca, container-apps-jobs, timeout, change-detection, performance
symptom/context: A nightly ACA Job is killed at replicaTimeout (e.g. 1800s); its
  output is never written, so every run restarts from scratch and dies at the same
  wall — a silent stall with frozen output.
root-cause: ACA Jobs have a HARD replicaTimeout and NO partial-progress
  checkpoint. A job whose total work exceeds the timeout never completes. In this
  case it ran expensive per-column sampling on ALL entities just to compute
  change-detection fingerprints.
fix: For change-detection, fingerprint with CHEAP metadata only (e.g. column
  name+type via INFORMATION_SCHEMA) and run expensive work (sampling/LLM) ONLY on
  the changed set. Seed the fingerprint store from existing output to avoid a
  first-run full rebuild. If work can still exceed the timeout, persist progress
  incrementally so reruns make forward progress.
