# azure — networking

## [azure-sql-mi-fqdn-private-peered-vnet]
created: 2026-06-08
tags: azure, sql-mi, vnet, peering, dns, private-endpoint
symptom/context: Deploying ACA (or other compute) in a VNet peered to the
  SQL MI's VNet; need to connect to the MI over its private endpoint
  (FQDN: <mi>.<dns-zone>.database.windows.net) from the compute VNet.
  Microsoft docs are ambiguous — some state a private DNS zone is REQUIRED
  for cross-peered-VNet resolution, others imply default Azure DNS handles it.
finding: Default Azure DNS (168.63.129.16) DOES automatically resolve the
  SQL MI VNet-local FQDN to its private IP from a correctly peered VNet,
  WITHOUT an explicit private DNS zone, provided BOTH peering directions are
  Connected + FullyInSync and allowForwardedTraffic=true on both sides.
  Validated in production: nslookup from ACA container (10.100.0.0/27, peered
  to MI VNet 172.29.218.128/26) returned 172.29.218.140 (private), not the
  public IP. The MI NSG rule VirtualNetwork->1433 automatically covers peered
  subnets — no NSG edit required.
recommendation: Validate in Phase 1 acceptance (nslookup from inside the
  container MUST return a private 172.29.x IP, not a public 52.x IP).
  Build a private DNS zone Bicep module as a safety net but gate it off
  (deployPrivateDnsZone=false parameter). Enable only if nslookup fails.
  Always use the FQDN, never a hard-coded IP (SQL MI does not support
  IP-address connections per Microsoft docs).
