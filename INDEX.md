# anpunkit-kb — index

2026-06-03 | azure/functions | azure-functions-flex-vnet-constraint | Standard Consumption plan does not support VNet integration; must upgrade to Flex or Premium
2026-06-04 | azure/functions | azure-functions-flex-no-odbc | ODBC Driver 18 cannot be installed on Flex Consumption; use pymssql instead
2026-06-04 | azure/sdk | azure-sdk-resource-not-found-status-none | ResourceNotFoundError.status_code is None; catch it explicitly before HttpResponseError
2026-06-04 | azure/pubsub | azure-web-pubsub-vs-signalr | Web PubSub preferred over SignalR for Python; ~$49/month vs $60-90, cleaner SDK
2026-06-06 | azure/auth | azure-ad-ropc-graph-direct-scope | ROPC token for direct Graph /me call must use graph.microsoft.com/.default, not app GUID scope
2026-06-03 | azure/auth | msal-browser-cache-inject-realm | MSAL-browser v3 cache injection fails without `realm` field in account entry; `tenantId` alone is insufficient
2026-05-31 | azure/auth | entra-v1-v2-token-audience-issuer | v1 tokens use api://<guid> aud + sts.windows.net iss; v2 use bare guid + login.microsoftonline.com/v2.0; validate both
2026-06-02 | azure/auth | msal-python-browser-cache-schema | msal-python ROPC token requires manual localStorage key construction for msal-browser v3 injection to work
2026-05-30 | azure/aca | aca-vnet-config-immutable | ACA managed environment vnetConfiguration is immutable; adding VNet integration requires full env recreate
2026-05-30 | azure/aca | aca-fqdn-changes-on-env-recreate | ACA FQDN changes on env recreate; update CORS, Entra redirect URIs, and re-set container secrets
2026-06-03 | azure/aca | aca-in-process-cache-multi-instance | ACA load-balances across replicas; in-process cache is per-replica, not fleet-wide
2026-06-04 | azure/sql | pyodbc-odbc18-mars-off-cursor-leak | ODBC Driver 18 MARS OFF + unclosed cursors silently starves subsequent queries; always call cursor.close()
2026-05-31 | azure/sql | azure-sql-obo-utf16le-token | Azure SQL token auth via pyodbc requires UTF-16-LE encoding + 4-byte length header via struct.pack
2026-06-01 | azure/openai | openai-sdk-httpx-proxies-pin | openai >= 1.51 + httpx >= 0.28 breaks with proxies kwarg error; pin httpx==0.27.2
2026-06-01 | python/sql | sqlglot-tsql-dateadd-zero-anchor | sqlglot <= 25.x misparses DATEADD(...,0) in tsql dialect; upgrade to >= 30.8.0 and add fail-closed guard verification
2026-06-02 | frontend/react | react-postmessage-useeffect-race | postMessage events arrive before useEffect registers; buffer at module-eval time
2026-06-02 | frontend/auth | postmessage-wildcard-origin-not-auto-match | Array.includes("*") does not match origins; must check includes("*") explicitly
2026-05-31 | sdp/api | sdp-v3-fields-required-whitelist | SDP v3 list view omits fields unless whitelisted in list_info.fields_required; approver causes 400
2026-06-08 | azure/swa | swa-routes-json-deprecated-silently-ignored | SWA CLI >= 2.x silently ignores routes.json (deprecated); use staticwebapp.config.json with explicit per-route rewrites + navigationFallback
2026-06-08 | azure/swa | swa-cli-binary-direct-invocation-windows | When SWA CLI npm wrapper exits code 1 on Windows, invoke StaticSitesClient.exe directly with --app --apiToken --skipAppBuild true
2026-06-08 | azure/networking | azure-sql-mi-fqdn-private-peered-vnet | Default Azure DNS resolves SQL MI VNet-local FQDN to private IP across peered VNets without a private DNS zone (validated in production)
2026-06-08 | devops/shell | bash-dotenv-special-char-password | Sourcing .env in bash silently corrupts special-char passwords (!,$,*); parse the file line-by-line in the test setup instead
2026-06-08 | patterns/reconciler | reconciler-detector-time-ordered-store-duplicate | Reconciler+detector both writing same logical key with different timestamps creates invisible duplicates; index must group all rows per key
