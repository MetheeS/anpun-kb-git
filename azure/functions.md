# azure — functions

## [azure-functions-flex-vnet-constraint]
created: 2026-06-03
tags: azure-functions, vnet, consumption-plan, flex-consumption
symptom/context: az functionapp vnet-integration add fails on a Standard Consumption
  Function App; needed VNet access to SQL Server VMs inside a private subnet.
finding: Standard Consumption plan does NOT support Regional VNet integration. Confirmed
  via Microsoft Learn (updated 2026-03-03). Must upgrade to Flex Consumption (GA May 2024)
  or Premium. Flex requires subnet delegation Microsoft.App/environments (min /27 subnet).
  Premium requires Microsoft.Web/serverFarms (min /28).
recommendation: Plan a Flex Consumption or Premium upgrade before any VNet-dependent
  work. Add a plan-upgrade step to Phase 0 if the project reaches for private endpoints.

## [azure-functions-flex-no-odbc]
created: 2026-06-04
tags: azure-functions, flex-consumption, odbc, mssql, pymssql
symptom/context: Tried to install ODBC Driver 18 on a Flex Consumption Function App
  to connect to SQL Server. Needed pyodbc for existing adapter code.
finding: Impossible. Flex Consumption runs in a read-only sandboxed environment —
  no root access, no WEBSITE_STARTUP_COMMAND, no post-deploy hooks that can install
  system packages. Azure Linux 3 has tdnf but the runtime cannot access it.
  GitHub issue #9234 (CBL-Mariner) documents an additional ODBC init failure even with
  root. Premium Plan + custom Dockerfile (apt-get) works but is costly and still risks
  the CBL-Mariner issue.
recommendation: Use pymssql (pure Python, no ODBC driver, pip-installable). Works on
  Flex Consumption. Slightly different API than pyodbc (pymssql.connect vs pyodbc.connect).
  If Premium Plan is acceptable, custom Dockerfile with apt-get msodbcsql18 works on
  a Debian-based image.
