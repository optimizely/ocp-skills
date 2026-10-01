# CLI — Deployment & Job Management

- [Prepare](#prepare) — package, upload, and build the app
- [Publish](#publish) — make a version available in the App Directory
- [App info](#app-info) — list versions and their state (which are published)
- [Install](#install) — install a published version into an account
- [Upgrade](#upgrade) — move an installation to another version
- [List installations](#list-installations) — find tracker IDs for installed accounts
- [List functions](#list-functions) — retrieve deployed webhook URLs
- [Unpublish](#unpublish) — remove a version from the App Directory
- [Uninstall](#uninstall) — remove an installation from an account
- [Job management](#job-management) — trigger, monitor, and stop jobs

Availability zones used by the `-a` flag: `us` (default), `eu`, `au`.

> **Apps with CMS installation scope** are targeted with `--scope <instanceId>`; the tracker ID can then be omitted — see [per-instance-installation.md](per-instance-installation.md).

## Prepare

Packages, uploads, and builds the app. The project directory must be inside an initialized Git repository — the command will fail with an error if it is not.

> **Always get explicit user confirmation before running this command.**

```bash
ocp app prepare
```

| Flag                            | Description                                                             |
|---------------------------------|-------------------------------------------------------------------------|
| `--publish`                     | Automatically publish after a successful prepare                        |
| `--use-previous-app-env-values` | Reuse `.env` values from the previous version — local `.env` is ignored |
| `--bump-dev-version`            | Bump the dev version before building                                    |
| `--upgrade-deps`                | Automatically update OCP dependencies                                   |

## Publish

Makes a prepared version available in the OCP App Directory.

> **Always get explicit user confirmation before running this command.**

```bash
ocp directory publish <appId@version>
```

## App info

Shows the app's general details and every version with its state, newest first. Use it to find which versions are published — and therefore installable — before running `install` or `upgrade`.

```bash
ocp directory info <appId>
```

| Flag | Description                       |
|------|-----------------------------------|
| `-a` | Availability zone (default: `us`) |

Version states shown in the **Versions** list:

| State | Meaning |
| --- | --- |
| `PUBLISHED` | Published to the App Directory — can be installed |
| `BUILT` | Prepared successfully but not published yet; the review status is shown next to it (`NOT_STARTED`, `IN_REVIEW`, `APPROVED`, `NOT_REQUIRED`) |
| `NEW`, `PUBLISHING`, `UNPUBLISHING`, `UPGRADING_RUNTIME` | In progress |
| `BUILD_FAILED`, `PUBLISHING_FAILED`, `UNPUBLISHING_FAILED` | Failed |
| `UNPUBLISHED` | Removed from the App Directory |
| `ABANDONED` | Discarded |

## Install

Installs a published version into a specific account — or, for an app with CMS installation scope, into a specific CMS instance. Each installation gets its own webhook URLs for the app's functions. To find the tracker ID or CMS instance ID, see [accounts.md](accounts.md).

> **Always get explicit user confirmation before running this command.**

```bash
ocp directory install <appId@version> <trackerId>
```

| Flag      | Description                                                                                   |
|-----------|-----------------------------------------------------------------------------------------------|
| `-a`      | Availability zone (default: `us`)                                                             |
| `--scope` | CMS instance ID for an app with CMS installation scope; the tracker ID can then be omitted (`-s` is short form) |

## Upgrade

Moves an existing installation to another published version.

> **Always get explicit user confirmation before running this command.**

```bash
ocp directory upgrade <appId> <trackerId> --toVersion=<version>
```

| Flag          | Description                                                                              |
|---------------|------------------------------------------------------------------------------------------|
| `--toVersion` | Version to upgrade to (e.g. `1.0.0`); `-v` is short form                                 |
| `-a`          | Availability zone (default: `us`)                                                        |
| `--scope`     | CMS instance ID for an app with CMS installation scope; the tracker ID can then be omitted (`-s` is short form) |

## List installations

Lists all installations of an app, optionally filtered by version.

```bash
ocp directory listInstalls <appId>
ocp directory listInstalls <appId@version>
```

| Flag | Description                       |
|------|-----------------------------------|
| `-a` | Availability zone (default: `us`) |

Use this to find the tracker ID of each installation, needed by commands that target an installation (e.g. `listFunctions`, `jobs trigger`). For an app with CMS installation scope, the output adds a **Scope** column with the CMS instance name of each installation. Commands take the CMS instance ID, not the name — to get the ID from the name, see [accounts.md](accounts.md#find-a-cms-instance-id).

## List functions

Lists the deployed webhook URLs for an installation.

```bash
ocp directory listFunctions <appId> <trackerId>
```

| Flag      | Description                                                        |
|-----------|--------------------------------------------------------------------|
| `-a`      | Availability zone (default: `us`)                                  |
| `--scope` | CMS instance ID for an app with CMS installation scope; the tracker ID can then be omitted |

## Unpublish

Removes a version from the OCP App Directory.

> **Always get explicit user confirmation before running this command.**

```bash
ocp directory unpublish <appId@version> --no-prompt
```

| Flag | Description                       |
|------|-----------------------------------|
| `-a` | Availability zone (default: `us`) |

## Uninstall

Removes an installation from a specific account.

> **Always get explicit user confirmation before running this command.**

```bash
ocp directory uninstall <appId> <trackerId> --no-prompt
```

| Flag      | Description                                                        |
|-----------|--------------------------------------------------------------------|
| `-a`      | Availability zone (default: `us`)                                  |
| `--scope` | CMS instance ID for an app with CMS installation scope; the tracker ID can then be omitted |

## Job management

**Trigger a job manually:**

```bash
ocp jobs trigger <appId> <jobName> <trackerId>
```

| Flag           | Description                                                             |
|----------------|-------------------------------------------------------------------------|
| `--parameters` | JSON string of parameters to pass to the job (e.g. `'{"mode":"full"}'`) |
| `--scope`      | CMS instance ID for an app with CMS installation scope; the tracker ID can then be omitted     |
| `-a`           | Availability zone (default: `us`)                                       |

**List job execution history:**

```bash
ocp jobs list <appId>
```

| Flag          | Description                                                                            |
|---------------|----------------------------------------------------------------------------------------|
| `--trackerId` | Filter by tracker ID                                                                   |
| `--scope`     | Filter by CMS instance ID (apps with CMS installation scope)                                            |
| `--function`  | Filter by job name                                                                     |
| `--status`    | Filter by status: `PENDING`, `SCHEDULED`, `RUNNING`, `COMPLETE`, `ERROR`, `TERMINATED` |
| `--limit`     | Number of results (default: `50`)                                                      |
| `--from`      | Start time — ISO string, epoch, or relative (e.g. `"5m"`, `"7d"`)                      |
| `-a`          | Availability zone (default: `us`)                                                      |

**Show runtime status of a running job:**

```bash
ocp jobs runtimeStatus <jobId>
```

| Flag | Description                       |
|------|-----------------------------------|
| `-a` | Availability zone (default: `us`) |

**Terminate a running job:**

```bash
ocp jobs terminate <jobId>
```

| Flag | Description                       |
|------|-----------------------------------|
| `-a` | Availability zone (default: `us`) |
