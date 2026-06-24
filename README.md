# OCP Skills

Agent Skill Documents for building [Optimizely Connect Platform](https://docs.developers.optimizely.com/optimizely-connect-platform/docs) (OCP) apps. Follows the [agentskills.io](https://agentskills.io) open standard — auto-discovered by Claude Code, GitHub Copilot, and other compatible AI coding assistants.

## What's a Skill?

A skill is a set of reference documents that an AI coding assistant reads on demand during your session. AI models are trained on public data with a knowledge cutoff — OCP APIs evolve faster than that. Skills bridge the gap: when you ask an OCP question, the assistant reads the relevant file and answers from current, accurate documentation rather than guessing from stale training data.

## Skills

| Skill | Description |
| --- | --- |
| [ocp-app-development](skills/ocp-app-development) | Building, modifying, and debugging OCP apps |
| [ocp-local-testing](skills/ocp-local-testing) | Running and testing an OCP app locally before deploying |
| [ocp-node22-runtime-migration](skills/ocp-node22-runtime-migration) | Migrating an OCP app from node18 to node22 runtime (SDK 1.x→2.x), with optional path to SDK 3.x |
| [ocp-app-sdk-v3-migration](skills/ocp-app-sdk-v3-migration) | Modernizing an OCP app already on node22: app-sdk 2.x→3.x, native fetch, ESLint v9, Jest→vitest |

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

Each skill is a directory under `skills/` with a `SKILL.md` entry point and an optional `references/` directory of on-demand topic files.

```
ocp-skills/
└── skills/
    ├── ocp-app-development/              # Building, modifying, debugging OCP apps
    │   ├── SKILL.md
    │   └── references/                   # On-demand topic files (app.yml, lifecycle, SDKs, CLI, …)
    ├── ocp-local-testing/                # Running and testing an OCP app locally
    │   └── SKILL.md
    ├── ocp-node22-runtime-migration/     # node18 → node22 runtime + SDK 1→2 (optional → 3)
    │   └── SKILL.md
    └── ocp-app-sdk-v3-migration/         # app-sdk 2→3 modernization on node22
        └── SKILL.md
```

## Contributing

Skill documents must stay accurate and under 500 lines per file. When OCP APIs change, update the relevant reference file and open a PR.
