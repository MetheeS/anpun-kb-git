# azure — swa

## [swa-routes-json-deprecated-silently-ignored]
created: 2026-06-08
tags: swa, routing, config, deprecated, staticwebapp, nextjs
symptom/context: Direct navigation to SPA sub-routes (/ask, /schedules,
  /data-dictionary) returns 404 on Azure Static Web Apps despite a
  web/public/routes.json defining rewrite rules for each route. The rules
  appear correct and the file is present in the deployed output.
root-cause: routes.json is the DEPRECATED legacy SWA configuration format.
  @azure/static-web-apps-cli >= 2.x emits a WARNING during deployment —
  "Functionality defined in the routes.json file is now deprecated. File
  will be ignored!" — and then silently ignores ALL routing rules in it.
  With no routing config active, SWA serves the raw filesystem; a request
  for /ask finds no file at that path and returns 404.
fix: Replace routes.json with staticwebapp.config.json (same location:
  web/public/, same JSON schema). For a Next.js static export with
  trailingSlash: true add explicit per-route rewrites PLUS a
  navigationFallback. Brace-globs ({json,css}) in the exclude array are
  NOT supported — list each extension or use separate wildcard entries.

  {
    "routes": [
      { "route": "/ask",               "rewrite": "/ask/index.html" },
      { "route": "/ask/*",             "rewrite": "/ask/index.html" },
      { "route": "/schedules",         "rewrite": "/schedules/index.html" },
      { "route": "/schedules/*",       "rewrite": "/schedules/index.html" },
      { "route": "/data-dictionary",   "rewrite": "/data-dictionary/index.html" },
      { "route": "/data-dictionary/*", "rewrite": "/data-dictionary/index.html" }
    ],
    "navigationFallback": {
      "rewrite": "/index.html",
      "exclude": ["/_next/*", "/catalog.json"]
    },
    "responseOverrides": {
      "404": { "rewrite": "/404.html" }
    }
  }

## [swa-cli-binary-direct-invocation-windows]
created: 2026-06-08
tags: swa, deploy, windows, cli, workaround, binary
symptom/context: npx --yes @azure/static-web-apps-cli deploy <dir>
  exits with code 1 on Windows: "The deployment binary exited with
  code 1. If you are running in a minimal container image, ensure
  native dependencies are installed." No useful error in the wrapper
  output.
root-cause: The SWA CLI npm wrapper fails to correctly invoke the
  bundled StaticSitesClient.exe on some Windows environments (encoding
  issues, path handling, or runtime context).
fix: Invoke StaticSitesClient.exe directly. The binary accepts different
  argument names than the npm wrapper. Correct invocation:

    "C:\Users\<user>\.swa\deploy\<hash>\StaticSitesClient.exe" upload \
      --app "<output-dir>" \
      --apiToken "<deployment-token>" \
      --skipAppBuild true \
      --skipApiBuild true

  The hash is the subfolder under %USERPROFILE%\.swa\deploy\ (only one
  version is cached at a time). The binary looks for staticwebapp.config.json
  automatically in the --app directory. NOTE: --configFileLocation takes
  a DIRECTORY path (not a file path); the binary appends the filename itself.
  Flags that do NOT work: --rootPath, --deploymentToken, --outputLocation
  (when used without --app).
