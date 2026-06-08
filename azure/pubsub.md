# azure — pubsub

## [azure-web-pubsub-vs-signalr]
created: 2026-06-04
tags: azure, web-pubsub, signalr, websocket, pricing, python
symptom/context: Choosing a managed WebSocket push service for Python Azure Functions
  (Consumption/Flex) targeting 50-200 concurrent users, 20-100 messages/day.
finding: Azure Web PubSub wins over SignalR for polyglot/Python use cases.
  Cost: Web PubSub ~$49/month (1 unit, 1M messages/day free quota) vs SignalR
  ~$60-90/month. Web PubSub has a cleaner Python SDK (azure-messaging-webpubsubservice),
  no custom JWT construction needed. Negotiate pattern: HTTP Function returns
  wss://...?access_token=... via client.get_client_access_token(). Backend publishes
  via client.send_to_user() REST call (no persistent WebSocket server-side).
  WebSocket protocol bypasses browser CORS (Origin validated server-side), making
  it iframe-friendly. SignalR free tier is dev-only (20 connections).
recommendation: Use Azure Web PubSub for new Python-based real-time notification
  systems on Azure Functions. Keep HTTP polling as a fallback (graceful degradation
  on WebSocket disconnect).
