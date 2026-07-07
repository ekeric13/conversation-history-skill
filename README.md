# Conversation History Skill

A cross-harness skill for turning AI coding-agent sessions into concise conversation history.

This skill helps an agent reconstruct the useful parts of a session: the original ask, key constraints, pivots, decisions, implementation outcome, and verification. It is intentionally editorial rather than transcript-like, so the result is readable in a PR, ticket, or future handoff.

It supports Claude Code, Codex, and generic agent transcript logs.

## What It Does

Use this skill when you want an agent to:

- Summarize the conversation behind a code change.
- Reconstruct the prompts and decisions that shaped an implementation.
- Add readable session history to a PR, issue, ticket, or handoff note.
- Extract major pivots from a long Claude Code or Codex session.
- Preserve enough context for a future agent session without dumping raw logs.
- Filter out tool noise, harness records, hidden instructions, and sensitive data.

The skill is in [`conversation-history/`](./conversation-history/SKILL.md).

## Install With skills.sh

For Codex, Claude Code, Cursor, and other Agent Skills-compatible tools, use the skills.sh installer:

```bash
npx skills@latest add ekeric13/conversation-history-skill
```

Because this repo contains one skill, the installer should discover `conversation-history` directly.

## Install For Codex

Install it as a personal Codex skill:

```bash
mkdir -p ~/.codex/skills/conversation-history
rsync -a --delete conversation-history/ ~/.codex/skills/conversation-history/
```

Then start a new Codex session and invoke it explicitly:

```text
Use $conversation-history to summarize the key prompts, decisions, and pivots from this session.
```

Codex can also invoke it automatically when the task matches the skill description.

## Install For Claude Code As A Plugin

Once this repo is published to GitHub, install it as a Claude Code plugin:

```text
/plugin marketplace add https://github.com/ekeric13/conversation-history-skill
```

After that finishes, run:

```text
/plugin install conversation-history@conversation-history
```

Or from your shell:

```bash
claude plugin marketplace add https://github.com/ekeric13/conversation-history-skill
claude plugin install conversation-history@conversation-history
```

Plugin skills are namespaced. Invoke it as:

```text
/conversation-history:conversation-history
```

This is the best Claude Code install path for sharing the skill with other people because the repo ships `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.

## Install For Claude Code As A Personal Skill

For local-only use without a plugin marketplace, install it as a personal Claude Code skill:

```bash
mkdir -p ~/.claude/skills/conversation-history
rsync -a --delete conversation-history/ ~/.claude/skills/conversation-history/
```

Then use it in Claude Code:

```text
/conversation-history
```

Or ask naturally:

```text
Summarize the conversation history for this session and include the major decisions.
```

Claude Code can load the skill automatically when the request matches the `description` in `SKILL.md`.

## Install For One Project Only

For Codex project-local use, copy the skill into your repo's local skills directory if your Codex setup loads project skills from there:

```bash
mkdir -p .codex/skills/conversation-history
rsync -a --delete conversation-history/ .codex/skills/conversation-history/
```

For Claude Code project-local use:

```bash
mkdir -p .claude/skills/conversation-history
rsync -a --delete conversation-history/ .claude/skills/conversation-history/
```

Project-local install is useful when the conversation-history workflow should travel with a repo.

## Development Install

If you are editing this skill and want changes to show up without copying after every edit, symlink it:

```bash
mkdir -p ~/.codex/skills ~/.claude/skills
ln -sfn "$PWD/conversation-history" ~/.codex/skills/conversation-history
ln -sfn "$PWD/conversation-history" ~/.claude/skills/conversation-history
```

For Claude Code, symlinked skills require a recent version that follows symlinks from skill directories.

## Verify The Skill

Run the skill validator:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ./conversation-history
```

If that system skill path is not present on your machine, run the equivalent validator from your local skill-creator installation.

## How To Use It Well

Best input:

- The destination: chat response, PR description, ticket, handoff note, or future-session context.
- The desired level of detail.
- Whether to use visible context only or inspect local Codex/Claude Code history.
- Any repo diff or specific session ID that should be included.
- Any sensitive details that should be omitted or generalized.

If no transcript is available, the skill should produce a current-context recap and say that it is based on visible context only.

## License

No license has been specified yet. Add one before publishing or distributing this skill.
