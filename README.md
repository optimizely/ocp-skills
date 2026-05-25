# OCP Skills

Agent Skill Documents for building [Optimizely Connect Platform](https://docs.developers.optimizely.com/optimizely-connect-platform/docs) (OCP) apps. Follows the [agentskills.io](https://agentskills.io) open standard — auto-discovered by Claude Code, GitHub Copilot, and other compatible AI coding assistants.

## What's a Skill?

A skill is a set of reference documents that an AI coding assistant reads on demand during your session. AI models are trained on public data with a knowledge cutoff — OCP APIs evolve faster than that. Skills bridge the gap: when you ask an OCP question, the assistant reads the relevant file and answers from current, accurate documentation rather than guessing from stale training data.

## Skills

| Skill | Description |
| --- | --- |
| [ocp-app-development](skills/ocp-app-development) | Building, modifying, and debugging OCP apps |

## Installation

### Claude Code

```bash
git clone https://github.com/ZaiusInc/ocp-skills ~/.claude/ocp-skills && mkdir -p ~/.claude/skills && ln -s ~/.claude/ocp-skills/skills/* ~/.claude/skills/
```

### GitHub Copilot

```bash
git clone https://github.com/ZaiusInc/ocp-skills ~/.copilot/ocp-skills && mkdir -p ~/.copilot/skills && ln -s ~/.copilot/ocp-skills/skills/* ~/.copilot/skills/
```

### Verify

Start a session inside your project directory and ask:

```
I want to build an OCP app. Start with a webhook function that receives events from an external service and tracks them in ODP.
```

You should get a grounded answer with a correct `App.Function` class, `app.yml` entry, and `z.event()` call — not a generic guess.

### Update

**Claude Code:**
```bash
cd ~/.claude/ocp-skills && git pull
```

**GitHub Copilot:**
```bash
cd ~/.copilot/ocp-skills && git pull
```

## Project Structure

```
ocp-skills/
└── skills/
    └── ocp-app-development/
        ├── SKILL.md                      # Overview, app types, building blocks, developer journey
        └── references/                   # Loaded on demand — one file per topic
            ├── app-yml.md                # app.yml structure and all configuration options
            ├── function.md               # App.Function, App.GlobalFunction
            ├── job.md                    # App.Job, prepare/perform loop, cron, pagination
            ├── odp-schema.md             # ODP schema extensions — custom fields on ODP objects
            ├── lifecycle/
            │   ├── install.md            # onInstall — setup, secrets, external webhook registration
            │   ├── settings-form.md      # onSettingsForm — saving settings, button actions
            │   ├── oauth.md              # onAuthorizationRequest + onAuthorizationGrant
            │   ├── uninstall.md          # onUninstall + canUninstall — cleanup
            │   └── upgrade.md            # onUpgrade, onFinalizeUpgrade, onAfterUpgrade
            ├── settings-forms/
            │   ├── elements.md           # Element types — text, select, toggle, button, oauth_button
            │   └── conditional-logic.md  # Visibility, required fields, validation rules
            ├── data-sync/
            │   ├── source.md             # Sources, static/dynamic schema, sources.emit()
            │   └── destination.md        # App.Destination<T>, ready(), deliver(), deletes
            ├── opal-tools/
            │   ├── tool-functions.md     # ToolFunction, GlobalToolFunction
            │   ├── tools.md              # @tool, @interaction, ParameterType, OptiID auth
            │   └── islands-interactions.md # Island UI components and interaction handlers
            ├── app-sdk/
            │   ├── storage.md            # settings, secrets, kvStore, sharedKvStore
            │   ├── notifications.md      # notifications.info/success/warn/error
            │   └── logging.md            # logger API
            ├── node-sdk/
            │   ├── events.md             # z.event()
            │   ├── customers.md          # z.customer(), identity resolution
            │   ├── objects.md            # z.object(), custom object operations
            │   ├── graphql.md            # z.graphql() queries and mutations
            │   ├── schema.md             # ODP schema inspection — read schema, fields, objects
            │   ├── identifiers.md        # Identity resolution and merging
            │   └── lists.md              # List membership management
            └── cli-commands/
                ├── scaffolding.md        # ocp app init, ocp add
                ├── validation.md         # ocp app validate, tsc
                ├── deployment.md         # ocp app prepare, ocp directory publish/install
                └── logging.md            # ocp app logs, get-log-level, set-log-level
```

## Contributing

Skill documents must stay accurate and under 500 lines per file. When OCP APIs change, update the relevant reference file and open a PR.
