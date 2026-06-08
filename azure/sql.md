# azure — sql

## [pyodbc-odbc18-mars-off-cursor-leak]
created: 2026-06-04
tags: pyodbc, azure-sql, odbc-driver-18, mars, cursor-leak, silent-failure
symptom/context: A loop that opens a cursor per iteration (e.g. COUNT(DISTINCT) +
  DISTINCT TOP 50 sampling across every column in every table) returns zero results
  silently. No exception propagates; the calling code sees an empty or un-enriched
  output. Warm cache serves the empty result for the full TTL.
root-cause: ODBC Driver 18 for SQL Server defaults to MARS OFF (Multiple Active
  Result Sets disabled). With MARS OFF a connection permits only ONE active statement
  at a time. Calling cursor.execute() without cursor.close() leaves the prior
  statement active; the next cursor.execute() on the same connection raises a pyodbc
  error. If that error is swallowed by a bare `except: continue`, every subsequent
  iteration silently produces nothing.
fix: Wrap every cursor in try/finally: cursor.close(). Never suppress exceptions
  silently — add logging.warning() in every except block so failures surface in
  telemetry (App Insights). Pattern:
    cursor = conn.cursor()
    try:
        cursor.execute(sql)
        rows = cursor.fetchall()
    finally:
        cursor.close()
failed-attempts: Checking cursor.timeout (AttributeError — cursor has no .timeout;
  only conn has .timeout). This was a red herring.

## [azure-sql-obo-utf16le-token]
created: 2026-05-31
tags: azure-sql, pyodbc, obo, managed-identity, token-auth, utf-16-le, struct-pack
symptom/context: Connecting to Azure SQL MI via pyodbc using an OBO or managed-identity
  access token fails with authentication errors when the token is passed as a plain
  Python string or bytes without the required encoding.
root-cause: pyodbc's SQL_COPT_SS_ACCESS_TOKEN attribute (numeric value 1256) expects
  the token as UTF-16-LE encoded bytes prepended with a 4-byte little-endian length
  header. Passing the raw token string causes the driver to reject the credential.
fix: Encode with:
    token_bytes = access_token.encode("utf-16-le")
    header = struct.pack("<I", len(token_bytes))
    attrs_before = {1256: header + token_bytes}
  Pass via pyodbc.connect(conn_str, attrs_before=attrs_before).
  Do NOT include UID or PWD in the connection string.
  Connection string: "DRIVER={ODBC Driver 18 for SQL Server};SERVER=<host>,<port>;
  DATABASE=<db>;Encrypt=yes;TrustServerCertificate=no"
  For managed identity: acquire token for resource "https://database.windows.net/.default"
  via ManagedIdentityCredential(client_id=<uami-client-id>).get_token(...).token
