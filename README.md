# Microsoft Sentinel KQL Detection Lab

Home lab, not employment. KQL (Kusto Query Language) — the query language
Microsoft Sentinel uses — practised on real sample data, because a live
Sentinel workspace was not available for this run.

## Why this lab exists, honestly

A live Microsoft Sentinel workspace needs an Azure subscription. That
needs a Visa/Mastercard in the account holder's name for verification,
and Azure for Students did not verify this student account. The free
alternatives were also empty at the time of this run:

- Log Analytics demo (https://aka.ms/LADemo) — `SecurityEvent`,
  `Syslog`, `AzureActivity`, `SigninLogs` all returned no results.
- Azure Data Explorer sample, `TransformedSysLogs` table — count was 1
  row total.

So this lab was done in Azure Data Explorer (https://dataexplorer.azure.com,
no card, no Azure subscription) on the `help/SampleLogs` database, table
`RawSysLogs`, which does have live data. The KQL is the same language
Sentinel uses. The data is SQL Server monitoring logs, not Windows
security logs — that is stated here, not hidden.

## What was run

All queries run 23 Nov 2026 (UTC) in Azure Data Explorer, database
`help/SampleLogs`, table `RawSysLogs`.

### 1. Count by metric name

```kql
RawSysLogs
| summarize Count = count() by name
| sort by Count desc
| take 10
```

Top result: `sqlserver_performance` — 753,187,980 records, about 85% of
the table. Next: `sqlserver_waitstats` 59,071,116,
`sqlserver_memory_clerks` 31,235,165, `sqlserver_schedulers`
15,024,121, `sqlserver_azuredb_waitstats` 13,959,224.
See `screenshots/kql1-rawsyslogs-by-name.png`.

### 2. Count by host

```kql
RawSysLogs
| summarize Count = count() by tostring(tags.host)
| sort by Count desc
| take 10
```

Result: one host only — `adx-vm`, 885,047,768 records. All of the
sample data comes from a single host.
See `screenshots/kql2-rawsyslogs-by-host.png`.

### 3. Total count

```kql
RawSysLogs
| count
```

Result: **885,047,768** records. The top-10 by name add up to the same
total, which cross-checks queries 1 and 3.
See `screenshots/kql3-rawsyslogs-count.png`.

## Sentinel detection rules (written, KQL)

The 3 Sentinel-style detection rules from this lab are in
`queries/sentinel-rules.kql`, written for the tables Sentinel normally
uses (`SecurityEvent`, `SigninLogs`). They were **not** tested on live
Sentinel data, because no live workspace was available — the LA Demo
tables for those rules were empty. The KQL syntax in them is the same
`summarize`, `where`, `sort` and `count` pattern proven working on
`RawSysLogs` above. Do not present them as tested detections.

- Rule 1: Excessive failed logons — `SecurityEvent | where EventID == 4625`,
  count by Account, threshold > 10 in 5 minutes.
- Rule 2: Successful logon after failures — 4625 followed by 4624 for
  the same account.
- Rule 3: Failed sign-ins by user — `SigninLogs | where ResultType != 0`,
  count by UserPrincipalName.

## Structure

- `findings/triage-notes.md` — full notes, including the dead ends
- `queries/kql-queries.kql` — the 3 queries actually run on RawSysLogs
- `queries/sentinel-rules.kql` — the 3 written Sentinel rules (not
  live-tested, see above)
- `screenshots/` — 3 screenshots, one per query run

## Tools

Azure Data Explorer (free, no card), KQL. No Azure subscription, no
Sentinel workspace, no credit card used.
