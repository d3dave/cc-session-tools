# cc-session-tools

Scripts for managing [Claude Code](https://claude.ai/code) session files. Useful when you rename or move a repo and want to keep your session history resumable.

## Scripts

- **`migrate-cc-project`** — Migrate all sessions from an old project path to CWD after a repo rename/move. Rewrites `cwd` paths in JSONL/JSON files. Prompts to delete the old directory afterward.
- **`archive-cc-session`** — Zip a session (by UUID) for backup or transfer. Searches all project directories; errors on ambiguous matches.
- **`restore-cc-session`** — Extract an archived session zip into the current project directory. Auto-detects the original path from session data and rewrites it to match CWD.

## Usage

```sh
# After renaming/moving a repo, migrate old sessions to the new location
cd /new/path/to/repo
./migrate-cc-project /old/path/to/repo

# Archive a session for backup or transfer
./archive-cc-session <session-uuid>

# Restore an archived session into the current project
./restore-cc-session <session.zip>
```

## How Claude Code stores sessions

Sessions live in `~/.claude/projects/<mangled-path>/` where the mangled path replaces `/` with `-` (e.g. `/Volumes/code/foo` → `-Volumes-code-foo`). Each session is a `.jsonl` file named by UUID, with `cwd` and path references embedded in every JSON line. Sessions may also have a subdirectory (same UUID) containing subagent files.

When a repo moves, sessions become unresumable because the paths no longer match. These scripts fix that by copying files to the new project directory and rewriting all path references.
