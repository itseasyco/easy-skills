# easylabs-skills

Skills for [Claude Code](https://claude.com/claude-code) from [Easy Labs](https://itseasy.co) — decision-quality techniques worth keeping in muscle memory.

## Available skills

| Skill | What it does |
|---|---|
| [premortem](skills/premortem/) | Imagines your plan has already failed and works backward to identify why. Generates parallel deep-dives per failure mode, then synthesizes a revised plan + pre-launch checklist as a self-contained "intelligence brief" HTML report. Triggers on phrases like `premortem this`, `what could kill this`, `stress test this plan`. |

More skills land here as we build them.

## Install

Two paths — pick whichever your tool supports.

### Skills CLI (works with Claude Code, Cursor, OpenCode, and any agent the [Skills CLI](https://skills.sh) supports)

```bash
# Install all skills, globally (user-level)
npx skills add itseasyco/easylabs-skills -g -y

# Or install a specific skill
npx skills add itseasyco/easylabs-skills --skill premortem -g -y
```

### Claude Code plugin marketplace

```text
/plugin marketplace add itseasyco/easylabs-skills
/plugin install easylabs-skills
```

Both paths install the same skills from the same repo. Use whichever fits your workflow.

## Using the skills

Once installed, the skills auto-trigger when you say a matching phrase. Example:

```text
> premortem this: I'm about to launch a $297 live workshop on X for Y…
```

Each skill's `SKILL.md` documents its trigger phrases, the procedure it follows, and the output it produces.

## Updating

```bash
# Skills CLI
npx skills update -g

# Claude Code plugin
/plugin update easylabs-skills
```

## Contributing / feedback

Issues and PRs welcome at [github.com/itseasyco/easylabs-skills](https://github.com/itseasyco/easylabs-skills).

## License

MIT — see [LICENSE](LICENSE).
