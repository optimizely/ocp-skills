# CLI — Accounts

- [Who am I](#who-am-i) — your role, and the accounts and CMS instances you can reach
- [Find a tracker ID](#find-a-tracker-id)
- [Find a CMS instance ID](#find-a-cms-instance-id)

## Who am I

Shows your profile (including your role) and every OCP account and CMS instance you can reach. Use it to look up the **tracker ID** or **CMS instance ID** a command needs — users rarely know either.

```bash
ocp accounts whoami          # human-readable summary
ocp accounts whoami --json   # raw profile and entitlements — use this when parsing
```

The JSON groups accounts and CMS instances by Opti ID organization:

```json
{
  "profile": { "email": "jdoe@example.com", "role": "developer", "...": "..." },
  "entitlements": {
    "optiId": {
      "accountStatus": "ok",
      "scopeStatus": "ok",
      "organizations": [
        {
          "organizationId": "37f848911df94ff38f3f52a1b312fcbc",
          "organizationName": "Acme",
          "accounts": [
            { "trackerId": "TEWmF71qZwuZ8IrGEjODFw", "name": "Acme US", "shard": "us", "instanceId": "d1f02159278745389009a257f7deaacf", "accountType": "ODP" },
            { "trackerId": "T3qnEfW_ruC1PEuvL0uLJw", "name": "Acme EU", "shard": "eu", "instanceId": "d5035bcc0aab48a9b11c642bfbd572ef", "accountType": "OCP" }
          ],
          "cmsInstances": [
            {
              "projectName": "acmesaas",
              "instances": [
                { "id": "581f186882df494e9799b101a20b560b", "name": "acmesaas: Production1", "region": "" },
                { "id": "10d7e5093b9c4a68806cdae9142cc333", "name": "acmesaas: Test1", "region": "" }
              ]
            }
          ],
          "unresolvedInstances": [
            { "instanceId": "2cd278c4314046c58da3c2df2365ea6b", "instanceName": "Acme Training", "region": "US", "reason": "no_account" }
          ]
        }
      ]
    }
  }
}
```

- If `accountStatus` or `scopeStatus` is not `ok`, the account or CMS instance list may be incomplete — tell the user.
- `accounts` and `cmsInstances` can be empty for an organization.
- `profile.role` is the user's OCP role — `developer`, `deployer`, `metric_analyst`, `administrator`, or `app_administrator`. The `administrator` and `app_administrator` roles are **admins**: they can reach every OCP account, not only the ones listed.

## Find a tracker ID

The tracker ID is `entitlements.optiId.organizations[].accounts[].trackerId`.

```bash
ocp accounts whoami --json | jq -r '.entitlements.optiId.organizations[].accounts[] | "\(.trackerId)\t\(.shard)\t\(.accountType)\t\(.name)"'
```

- Match the user's description against `accounts[].name` (and `organizationName`). If more than one account could match, **ask the user which one** — never guess a tracker ID for install, uninstall, or upgrade.
- `shard` is the account's availability zone — pass it as `-a <shard>` to commands for that account (the default is `us`).
- `instanceId` is the account's OCP instance ID, not a tracker ID; CLI commands take the tracker ID.

To find accounts that **already have an app installed**, use `ocp directory listInstalls <appId>` instead (see [deployment.md](deployment.md#list-installations)).

## Find a CMS instance ID

For an app with CMS installation scope, commands target an installation with `--scope <cmsInstanceId>` (see [per-instance-installation.md](per-instance-installation.md)). The CMS instance ID is `entitlements.optiId.organizations[].cmsInstances[].instances[].id`.

```bash
ocp accounts whoami --json | jq -r '.entitlements.optiId.organizations[].cmsInstances[].instances[] | "\(.id)\t\(.name)"'
```

- Match the user's description (e.g. "Production", "Test1") against `instances[].name`. If more than one instance could match, **ask the user which one** — never guess a scope for install, uninstall, or upgrade.
- `unresolvedInstances` are CMS instances with no reachable OCP account (`reason`: `no_region`, `unmapped_region`, `no_account`, `lookup_failed`). They cannot be used as a scope.
