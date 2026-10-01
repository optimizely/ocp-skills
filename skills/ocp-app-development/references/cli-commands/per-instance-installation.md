# CLI — Per-instance installation (CMS apps)

- [What changes for an app with CMS installation scope](#what-changes-for-an-app-with-cms-installation-scope)
- [Check whether an app has CMS installation scope](#check-whether-an-app-has-cms-installation-scope)
- [Find the CMS instance ID](#find-the-cms-instance-id)
- [Commands that take `--scope`](#commands-that-take---scope)

CMS UI Extension apps can be classified with the **CMS installation scope**. Such an app is installed once **per CMS instance** instead of once per OCP account: each CMS instance gets its own installation, settings, and storage. The CMS instance ID is the installation's **scope**.

## What changes for an app with CMS installation scope

- **Target an installation with `--scope <instanceId>`** — the tracker ID can then be omitted. OCP resolves the owning OCP account from the CMS instance.
- **Without `--scope`, the CLI prompts** the user to pick a CMS instance. The prompt needs an interactive terminal: when stdin is not a TTY (e.g. when an agent runs the command) or with `--no-prompt`, the command fails with:

  ```
  <appId> is a CMS UI Extension app. Pass --scope to specify a CMS instance.
  ```

  **Always pass `--scope` when running these commands yourself.** Never rely on the prompt.
- **One OCP account can hold several installations** of the same app — one per CMS instance — so the tracker ID alone no longer identifies an installation.
- A CMS instance is managed by exactly one OCP account. Only the managing account can install the app on that CMS instance.

## Check whether an app has CMS installation scope

```bash
ocp directory info <appId>
```

An app with CMS installation scope shows `Installation Scope` set to `CMS` in the **General** section. If the field is absent, the app is installed per OCP account — use the tracker ID as usual.

## Find the CMS instance ID

Users rarely know CMS instance IDs — look them up with `ocp accounts whoami --json` as described in [accounts.md](accounts.md#find-a-cms-instance-id).

To see which CMS instances already have the app installed, run `ocp directory listInstalls <appId>` — for an app with CMS installation scope it adds a **Scope** column with the CMS instance name. `--scope` takes the instance ID, so look the name up with `ocp accounts whoami --json`.

## Commands that take `--scope`

Every command that targets an installation accepts `--scope <instanceId>` (short form `-s`); the tracker ID can then be omitted — see [deployment.md](deployment.md) and [logging.md](logging.md) for the exact syntax.
