# Triage Notes — Microsoft Sentinel KQL Detection Lab

Analyst: Vedhan Lad. Date: 23 Nov 2026. Lab, not a real incident.

## Environment — what was actually used

- Azure Data Explorer, https://dataexplorer.azure.com, database
  `help/SampleLogs`, table `RawSysLogs`.
- No Azure subscription, no credit card, no live Sentinel workspace.
- `RawSysLogs` rows are SQL Server monitoring metrics, e.g.
  `name: sqlserver_schedulers`, `tags.host: adx-vm`,
  `tags.sql_instance: adx-sql-server`, timestamp 2023-02-21.

## Results

### Query 1 — count by metric name

```kql
RawSysLogs
| summarize Count = count() by name
| sort by Count desc
| take 10
```

| name | Count |
|---|---:|
| sqlserver_performance | 753,187,980 |
| sqlserver_waitstats | 59,071,116 |
| sqlserver_memory_clerks | 31,235,165 |
| sqlserver_schedulers | 15,024,121 |
| sqlserver_azuredb_waitstats | 13,959,224 |
| sqlserver_database_io | 6,546,811 |
| sqlserver_requests | 2,452,438 |
| sqlserver_db_resource_governance | 1,190,316 |
| sqlserver_azure_db_resource_stats | 1,190,305 |
| sqlserver_server_properties | 1,190,292 |

Run: 2026-11-23 20:36 UTC, done in 2.377 s, 10 records.
Screenshot: `screenshots/kql1-rawsyslogs-by-name.png`.

Finding: performance counters dominate — `sqlserver_performance` alone
is about 85% of the table. That is normal for SQL monitoring data, not
an attack indicator.

### Query 2 — count by host

```kql
RawSysLogs
| summarize Count = count() by tostring(tags.host)
| sort by Count desc
| take 10
```

| tags_host | Count |
|---|---:|
| adx-vm | 885,047,768 |

Run: 2026-11-23 20:38 UTC, done in 9.545 s, 1 record.
Screenshot: `screenshots/kql2-rawsyslogs-by-host.png`.

Finding: a single host, `adx-vm`, produced all 885,047,768 records.
No multi-host comparison is possible in this sample.

### Query 3 — total count

```kql
RawSysLogs
| count
```

Result: **885,047,768**.
Run: 2026-11-23 20:38 UTC, done in 0.528 s, 1 record.
Screenshot: `screenshots/kql3-rawsyslogs-count.png`.

Cross-check: the top-10 counts in Query 1 sum to 885,047,768, matching
Query 3 exactly.

## Dead ends, recorded honestly

1. Azure Portal direct sign-in with vedhan88@gmail.com — error
   AADSTS50020, account does not exist in tenant 'Microsoft Services'.
   A personal Microsoft account with no Azure subscription has no tenant.
2. Azure free account — requires a Visa/Mastercard in the holder's
   name for verification. None available.
3. Azure for Students — "you don't have access", student verification
   did not pass.
4. Log Analytics demo (aka.ms/LADemo) — `SecurityEvent | take 5`,
   `Syslog | take 5`, `AzureActivity | take 5`, `SigninLogs` queries —
   all no results. Tables empty at the time of the run.
5. ADX sample, `TransformedSysLogs` — `| count` returned 1. The
   transformed table is effectively empty; the raw table is the one
   with data. The first three queries were tried on
   `TransformedSysLogs` (errors by app, failed messages, errors by
   severity) and all returned no rows for this reason.

## Verdict

KQL proven working on real data (885M rows, 3 queries, screenshots).
No live Sentinel workspace, no Sentinel incidents, no security-log
detections tested live — the Sentinel rules in
`queries/sentinel-rules.kql` are written rules, not live-tested ones.
That limit is stated in the README and here, so the project is not
oversold.
