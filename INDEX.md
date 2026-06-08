# anpunkit-kb — index

2026-06-03 | azure/functions | azure-functions-flex-vnet-constraint | Standard Consumption plan does not support VNet integration; must upgrade to Flex or Premium
2026-06-04 | azure/functions | azure-functions-flex-no-odbc | ODBC Driver 18 cannot be installed on Flex Consumption; use pymssql instead
2026-06-04 | azure/sdk | azure-sdk-resource-not-found-status-none | ResourceNotFoundError.status_code is None; catch it explicitly before HttpResponseError
2026-06-04 | azure/pubsub | azure-web-pubsub-vs-signalr | Web PubSub preferred over SignalR for Python; ~$49/month vs $60-90, cleaner SDK
2026-06-06 | azure/auth | azure-ad-ropc-graph-direct-scope | ROPC token for direct Graph /me call must use graph.microsoft.com/.default, not app GUID scope
2026-06-02 | frontend/react | react-postmessage-useeffect-race | postMessage events arrive before useEffect registers; buffer at module-eval time
2026-06-02 | frontend/auth | postmessage-wildcard-origin-not-auto-match | Array.includes("*") does not match origins; must check includes("*") explicitly
2026-05-31 | sdp/api | sdp-v3-fields-required-whitelist | SDP v3 list view omits fields unless whitelisted in list_info.fields_required; approver causes 400
