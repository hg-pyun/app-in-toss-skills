# Apps In Toss Skills

Claude Code plugin marketplace for developing [Apps In Toss (앱인토스)](https://developers-apps-in-toss.toss.im) mini-apps.
Each plugin bundles skills that map to a slice of the mini-app lifecycle — scaffolding, knowledge lookup, sandbox testing, deployment, and so on.

## Install

```sh
# Add this repo as a plugin marketplace
/plugin marketplace add toss/apps-in-toss-skills

# Install a specific plugin (see the index below)
/plugin install scaffolding@apps-in-toss-skills
```

To iterate locally without publishing:

```sh
claude --plugin-dir ./plugins/scaffolding
```

## Plugins

| Plugin | Skills | Status |
| --- | --- | --- |
| [`scaffolding`](./plugins/scaffolding) | `/scaffolding:create-mini-app` | ✅ shipped |

New plugin categories land here as the marketplace grows (e.g. `knowledge-skills`, `testing`, `deploy`).

## Repository layout

```
apps-in-toss-skills/
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (one entry per plugin)
├── plugins/
│   └── <plugin>/
│       ├── .claude-plugin/
│       │   └── plugin.json       # plugin manifest
│       ├── README.md             # plugin scope + skill index
│       └── skills/
│           └── <skill>/
│               ├── SKILL.md      # required entry point
│               ├── references/   # optional — progressive disclosure docs
│               ├── scripts/      # optional — executables the skill runs
│               └── templates/    # optional — files the skill copies
└── README.md                     # this file
```

## Adding a new skill

1. **Pick a plugin.** Skills go inside the plugin whose scope matches the task.
   - Scaffolding / project setup / adding features → [`plugins/scaffolding`](./plugins/scaffolding).
   - Docs / API lookups → `plugins/knowledge-skills` (create if missing).
   - Anything new → propose a new plugin (see below).
2. **Create the skill directory** `plugins/<plugin>/skills/<skill-name>/` with a `SKILL.md`.
3. **Write the frontmatter.** At minimum set `description` — make the trigger keywords explicit (Korean + English when relevant). Use `allowed-tools` to limit tool access and `argument-hint` to advertise CLI arguments.
4. **Keep `SKILL.md` under 500 lines.** Push long reference material into `references/*.md` files the skill can pull in on demand (progressive disclosure).
5. **Update** the plugin's `README.md` skill index and the root `README.md` plugin table.

## Adding a new plugin

1. Create `plugins/<plugin>/.claude-plugin/plugin.json` with `name`, `description`, `version`, `author`.
2. Add an entry in `.claude-plugin/marketplace.json` — just `{ "name", "source": "<plugin>", "description" }` thanks to `metadata.pluginRoot`.
3. Add `plugins/<plugin>/README.md` documenting the plugin's scope.
4. Ship at least one skill before linking the plugin in the root table.

Plugin names should be kebab-case and describe a *domain* ("scaffolding", "deploy") rather than a single action ("create-app"). Skills inside a plugin handle individual actions.

## Validation

```sh
/plugin validate .
# or
claude plugin validate .
```

Run this before committing — it catches malformed `marketplace.json`, duplicate plugin names, and broken SKILL frontmatter.

## References

- [Claude Code skills docs](https://code.claude.com/docs/ko/skills)
- [Claude Code plugins docs](https://code.claude.com/docs/ko/plugins)
- [Apps In Toss developer docs](https://developers-apps-in-toss.toss.im)
