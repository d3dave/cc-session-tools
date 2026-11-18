# cc-session-tools

Scripts for managing [Claude Code](https://claude.ai/code) session files. Useful when you rename or move a repo and want to keep your session history resumable.

## Scripts

- **`migrate-cc-project`** — Migrate all sessions from an old project path to CWD after a repo rename/move. Rewrites `cwd` paths in JSONL/JSON files. Prompts to delete the old directory afterward.
- **`archive-cc-session`** — Zip a session (by UUID) for backup or transfer. Searches all project directories; errors on ambiguous matches.
- **`restore-cc-session`** — Extract an archived session zip into the current project directory. Auto-detects the original path from session data and rewrites it to match CWD.
- **`cc-tunnel`** — Bridge Claude Code peer-messaging sockets between the local host and a remote host over SSH, so `ListAgents` / `SendMessage` reach sessions on both machines. Runs in the foreground; tears everything down on exit.

## Usage

```sh
# After renaming/moving a repo, migrate old sessions to the new location
cd /new/path/to/repo
./migrate-cc-project /old/path/to/repo

# Archive a session for backup or transfer
./archive-cc-session <session-uuid>

# Restore an archived session into the current project
./restore-cc-session <session.zip>

# Bridge peer sessions to another host (Ctrl-C to tear down)
./cc-tunnel <remote-host>
```

## How Claude Code stores sessions

Sessions live in `~/.claude/projects/<mangled-path>/` where the mangled path replaces `/` with `-` (e.g. `/Volumes/code/foo` → `-Volumes-code-foo`). Each session is a `.jsonl` file named by UUID, with `cwd` and path references embedded in every JSON line. Sessions may also have a subdirectory (same UUID) containing subagent files.

When a repo moves, sessions become unresumable because the paths no longer match. These scripts fix that by copying files to the new project directory and rewriting all path references.

## How Claude Code peer messaging works

Each running session registers itself in `~/.claude/sessions/<pid>.json` (pid, session ID, name, status, and `messagingSocketPath`) alongside a `<pid>.<hash>.key` holding its peer token, and listens on a Unix socket at `/tmp/cc-socks/<pid>.sock`. `ListAgents` / `SendMessage` only see sessions registered on the same machine. `cc-tunnel` extends that across hosts: it SSH-forwards each side's sockets to the other (`-L` for remote sessions, `-R` for local ones) and copies the registry entries both ways, so each session resolves the other host's names. Sessions whose pid exists on both hosts are skipped to avoid clobbering a native entry.
