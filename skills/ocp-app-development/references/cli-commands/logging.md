# CLI — Logs

- [View logs](#view-logs)
- [Log level](#log-level)

Availability zones used by the `-a` flag: `us` (default), `eu`, `au`.

> **Apps with CMS installation scope** are targeted with `--scope=<instanceId>`; the tracker ID can then be omitted — see [per-instance-installation.md](per-instance-installation.md).

## View logs

Stream or query logs produced by your app. Logs are accessible for functions, jobs, and lifecycle hooks — not for global functions.

```bash
# Logs for an installation (last 24h by default)
ocp app logs --appId=<appId> --trackerId=<trackerId>

# Tail live
ocp app logs --appId=<appId> --trackerId=<trackerId> --tail

# Filter by level, time range, or search string
ocp app logs --appId=<appId> --trackerId=<trackerId> --level=ERROR --from=1h --search='failed'

# App with CMS installation scope — logs for one CMS instance (works with --tail, --level, --search, etc.)
ocp app logs --appId=<appId> --scope=<instanceId>
ocp app logs --appId=<appId> --scope=<instanceId> --level=ERROR --from=1h --search='failed'

# Logs for a specific job execution
ocp app logs --jobId=<jobId>

# Build logs from ocp app prepare
ocp app logs --buildId=<buildId>
```

One of `--appId`, `--buildId`, or `--jobId` is required. They are mutually exclusive — specify only one.

With `--appId` developers must narrow the query to one installation: `--trackerId` for a regular app, `--scope` for an app with CMS installation scope. Only admins can query an app's logs across all installations.

| Flag | Description |
| --- | --- |
| `--appId` | App ID |
| `--buildId` | Build logs from `ocp app prepare` |
| `--jobId` | Logs for a specific job execution — use `ocp jobs list` to get the job ID |
| `--trackerId` | OCP account tracker ID — required for developers when using `--appId`; not required with `--jobId` or `--buildId` |
| `--scope` | CMS instance ID — required for developers when the app has CMS installation scope (`--trackerId` alone is not enough); the tracker ID can then be omitted |
| `--appVersion` | Filter by version |
| `--level` | Filter by level: `DEBUG`, `INFO`, `WARN`, `ERROR`. Default: `DEBUG` (all logs shown) |
| `--from` | Start time — ISO string, epoch, or relative (e.g. `"1h"`, `"30m"`). Default: `24h` |
| `--to` | End time — same formats as `--from` |
| `--search` | Search string — can be specified multiple times |
| `--tail` | Stream logs live |
| `-a` | Availability zone (default: `us`) |

## Log level

Default log level is `DEBUG` for `-dev` versions and `INFO` for release and `beta` versions.

```bash
# Check log level for an installation
ocp app get-log-level <appId@version> --trackerId=<trackerId>

# Set log level for an installation (resets after 2h by default)
ocp app set-log-level <appId@version> DEBUG --trackerId=<trackerId>

# Override the default 2h expiration
ocp app set-log-level <appId@version> DEBUG --trackerId=<trackerId> --ttl=30m

# App with CMS installation scope — target one CMS instance
ocp app get-log-level <appId@version> --scope=<instanceId>
ocp app set-log-level <appId@version> DEBUG --scope=<instanceId>
```

Accepted levels: `DEBUG`, `INFO`, `WARN`, `ERROR`, `NEVER` (no logs) — upper case only; lower case is rejected.

Developers must target one installation: `--trackerId` for a regular app, `--scope` for an app with CMS installation scope. Only admins can omit both to apply the command to the app version itself.

Both commands accept `-a` for availability zone (default: `us`).
