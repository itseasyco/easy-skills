# easy-skills

Skills for [Claude Code](https://claude.com/claude-code) from [Easy Labs](https://itseasy.co) — decision-quality techniques worth keeping in muscle memory.

## Available skills

| Skill | What it does |
|---|---|
| [premortem](skills/premortem/) | Imagines your plan has already failed and works backward to identify why. Generates parallel deep-dives per failure mode, then synthesizes a revised plan + pre-launch checklist as a self-contained "intelligence brief" HTML report. Triggers on phrases like `premortem this`, `what could kill this`, `stress test this plan`. |
| [ai-citation](skills/ai-citation/) | Audits a brand's likelihood of being cited by AI search engines (ChatGPT, Perplexity, Claude, Gemini, Google AI Overviews), walks the team through prioritized growth-hack moves to fix gaps, and tracks improvements over monthly delta runs. Produces a Composite AI Citation Score (Readiness + Action), Brand Surface Area Map, and per-engine projections. Triggers on phrases like `audit our AI citation`, `GEO audit`, `get cited by ChatGPT`, `raise our AI citation score`. |

More skills land here as we build them.

## Install

Two paths — pick whichever your tool supports.

### Skills CLI (works with Claude Code, Cursor, OpenCode, and any agent the [Skills CLI](https://skills.sh) supports)

```bash
# Install all skills, globally (user-level)
npx skills add itseasyco/easy-skills -g -y

# Or install a specific skill
npx skills add itseasyco/easy-skills --skill premortem -g -y
```

### Claude Code plugin marketplace

```text
/plugin marketplace add itseasyco/easy-skills
/plugin install easy-skills
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
/plugin update easy-skills
```

## Contributing / feedback

Issues and PRs welcome at [github.com/itseasyco/easy-skills](https://github.com/itseasyco/easy-skills).

## License

MIT — see [LICENSE](LICENSE).
