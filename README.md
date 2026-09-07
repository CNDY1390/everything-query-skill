# Everything Query

An Agent Skill that turns natural-language file requests into Everything queries and runs them through the official ES command-line tool on Windows.

Covers names, directory structure, dates, content, duplicates, and metadata. Includes shared 1.4/1.5 syntax, version-specific behavior, and ES argument handling.

## Install

Create `everything-query/` in the agent's skills directory and copy [SKILL.md](SKILL.md) into it. For Codex, the default path is `~/.codex/skills/everything-query/SKILL.md`.

Local searches require a running [Everything](https://www.voidtools.com/) desktop client and the official [ES CLI](https://www.voidtools.com/support/everything/command_line_interface/). Advanced queries are marked where they require 1.5.

## Use

Ask the agent to use `everything-query` to find files.

[MIT License](LICENSE). Everything and ES are separate voidtools products.
