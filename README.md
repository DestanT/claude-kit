# claude-kit

Skills and mods for [Claude Code](https://claude.com/claude-code), in one place. Every item is a standalone plugin under `plugins/`: install only what you need.

> Based on [johnnyvizz/claude-kit](https://github.com/johnnyvizz/claude-kit) by johnnyvizz, used under the MIT License. This copy starts from upstream commit `c871c15` (13 Oct 2026) and is maintained separately.

| Plugin | Type | What it does |
| --- | --- | --- |
| [savvy-flow](plugins/savvy-flow) | skill + agents | `/savvy-flow <task>`: the session model plans, delegates to tiered worker subagents and reviews their work. |
| [savvy-progress](plugins/savvy-progress) | mod | A progress bar above the prompt and a live agents panel. Made for savvy-flow, shows any subagents. |
| [session-band](plugins/session-band) | mod | A band above the prompt: turn, speed, cost, prompt cache countdown and context; notifies before the cache expires. |

savvy-flow and savvy-progress work on their own and light up together: with both installed, the flow drives the bar and workers report their progress to the panel.

## Install

All three routes below pin a specific commit, so new pushes never reach a session until you choose to move the pin. Replace `<commit>` with the commit you want (the full ID from `git log`).

### On your Mac, for one project only

Clone the repo, lock it to a commit, and register the clone as the `claude-kit` marketplace:

```bash
git clone https://github.com/DestanT/claude-kit.git ~/code/claude-kit-pinned
git -C ~/code/claude-kit-pinned checkout --detach <commit>
claude plugin marketplace add ~/code/claude-kit-pinned
```

Then turn the plugins on for just that project, from its folder:

```bash
cd ~/code/<project>
claude plugin install savvy-flow@claude-kit --scope local
claude plugin install savvy-progress@claude-kit --scope local
```

Restart the session afterwards. Plugin skills and agents are namespaced: the skill is `/savvy-flow:savvy-flow`, the agents `savvy-flow:savvy-careful` and so on. This works in the CLI and in the Claude Desktop Code tab.

To move to a newer commit: `git -C ~/code/claude-kit-pinned checkout --detach <new-commit>`, then `claude plugin marketplace update claude-kit`.

### In cloud sessions (claude.ai projects and Claude Code on the web)

Cloud sessions don't load marketplace plugins, so a setup script copies the skill and agents in when each session starts. Paste this into the environment's setup script (Project settings, then Cloud environment, then the gear, then Setup script):

```bash
KIT=/opt/claude-kit
REVIEWED=<commit>
if [ ! -d "$KIT/.git" ]; then
  git clone --quiet https://github.com/DestanT/claude-kit.git "$KIT"
fi
if git -C "$KIT" checkout --quiet --detach "$REVIEWED" && [ "$(git -C "$KIT" rev-parse HEAD)" = "$REVIEWED" ]; then
  mkdir -p ~/.claude/skills ~/.claude/agents
  cp -R "$KIT/plugins/savvy-flow/skills/savvy-flow" ~/.claude/skills/
  cp "$KIT/plugins/savvy-flow/agents/"*.md ~/.claude/agents/
else
  echo "claude-kit: commit not found, skipping" >&2
fi
```

The skill is then `/savvy-flow` in new sessions. The mods are left out: they draw in the Claude Code terminal and desktop UI, which cloud project threads don't show.

### By hand, without the plugin system

Copy the skill and agents into a project's `.claude/` folder (or `~/.claude/` for every project). Names stay short: `/savvy-flow`, `savvy-careful`.

```bash
cp -R ~/code/claude-kit-pinned/plugins/savvy-flow/skills/savvy-flow <project>/.claude/skills/
mkdir -p <project>/.claude/agents
cp ~/code/claude-kit-pinned/plugins/savvy-flow/agents/*.md <project>/.claude/agents/
```

A mod can be loaded for one session with:

```bash
claude --plugin-dir ~/code/claude-kit-pinned/plugins/savvy-progress
```

Mods are built on function hooks (`hooks/register.tsx`), an early-access Claude Code API: tested on Claude Code 2.1.286, and it may change between releases.

### Notes

- `savvy-fable` runs on the Fable model. Without access to it, change `model:` in `plugins/savvy-flow/agents/savvy-fable.md` (to `opus`, say).
- savvy-flow fans work out to subagents at high effort, so it uses more than a single session.
- session-band sends a push notification 10 and 5 minutes before the prompt cache expires.

## Layout

```
.claude-plugin/marketplace.json   the catalog `/plugin marketplace add` reads
plugins/<name>/
  .claude-plugin/plugin.json      manifest
  skills/<skill>/SKILL.md         a skill
  agents/*.md                     subagents it ships
  hooks/hooks.json                a mod: points at the hooks module
  hooks/register.tsx              a mod: the module itself
  types/index.d.ts                a mod: its $.state contract
```

Check a plugin before publishing: `claude plugin validate plugins/<name>`, and the catalog with `claude plugin validate .`.

## License

MIT. Original work copyright (c) 2026 johnnyvizz; see [LICENSE](LICENSE).
