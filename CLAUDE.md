# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Python scripts for managing Claude Code session files under `~/.claude/projects/`.

- **`migrate-cc-project`** — Migrate all sessions from an old project path to CWD after a repo rename/move. Rewrites `cwd` paths in JSONL/JSON files. Prompts to delete the old directory afterward.
- **`archive-cc-session`** — Zip a session (by UUID) for backup or transfer. Searches all project directories; errors on ambiguous matches.
- **`restore-cc-session`** — Extract an archived session zip into the current project directory. Auto-detects the original path from session data and rewrites it to match CWD.

## Usage

```
cd /new/path/to/repo
./migrate-cc-project /old/path/to/repo
./archive-cc-session <session-uuid>
./restore-cc-session <session.zip>
```

## Style

Use `sys.exit("message")` for user-facing errors, not assertions.

## How Claude Code Session Storage Works

- Sessions live in `~/.claude/projects/<mangled-path>/` where the mangled path replaces `/` with `-` (e.g. `/Volumes/code/foo` → `-Volumes-code-foo`)
- Each session is a `.jsonl` file named by UUID, with `cwd` and path references embedded in every JSON line
- Sessions may have a subdirectory (same UUID) containing subagent `.jsonl` and `.meta.json` files
- To make a session resumable from a new directory: copy the files to the new project dir and rewrite all old-path occurrences to the new path
