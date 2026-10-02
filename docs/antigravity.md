# Antigravity (`agy`) as a provider

`--provider antigravity` runs both model sessions through Google Antigravity's command-line tool,
`agy`, in headless mode (`agy -p`). Claude Code and Codex are isolated with flags. `agy` has no
flags for most of that, so this provider builds the same guarantees another way. Each guarantee
below was checked against `agy` 1.2.14 and 1.2.15 on macOS.

## What each session is given

Every start of `agy` (each session, the sign-in check and the agent check) gets a **settings
folder of its own** through `--gemini_dir`. hazina-review creates it in the temporary directory
and deletes it, with everything `agy` wrote there, when that start ends. It holds:

| What | Why |
|---|---|
| An agent named `hazina-review-<random>`, with only `view_file`, `list_dir`, `grep_search` and `find_by_name`, `commandExecutionPolicy: off`, `inheritCustomizations: false`, `inheritMcp: false` | With no shell, web, write or MCP tools, nothing it reads can make it act. The random name means no repository can ship an agent of the same name to replace it. |
| `settings.json` denying `command(*)`, `write_file(*)`, `read_url(*)`, `execute_url(*)` and `mcp(*)`, and allowing reads of the two folders it is given | A second layer, in case some other agent ever runs. |
| On Linux without a keyring: a link to `~/.gemini/antigravity-cli/antigravity-oauth-token` | Where `agy` keeps the sign-in there. The file is linked by path and never opened. |

Because the settings folder is new each time, none of the operator's own `agy` set-up (rules,
skills, plugins, MCP servers) reaches a session. Nothing is added to their conversation history
either.

## What the copy leaves out

`agy` reads instructions from the folder it works in: `GEMINI.md`, `AGENTS.md`, `.agents/`
(rules, workflows, skills, agents, settings) and the variants `.agent/`, `_agents/`, `_agent/`,
plus `.gemini/` and `.antigravity/`. Tested: a repository's own `.agents/agents/<name>.md`
**replaced** an agent of the same name and gave itself a shell and web search. These paths are
therefore left out of the scratch copy the model reads, at any depth and in any letter case. The
count is kept as `instruction_paths_withheld` in the copy's own record. The history brief still
shows changes to them like any other file.

## Checks

- **Before every session**, `agy --gemini_dir=<folder> agents` must list that session's agent. If
  `agy` cannot find the agent it was told to run, it silently runs its default agent, which has
  every tool, and exits 0. So a session whose agent is not listed is never started, and the run
  stops with "did not take up the read-only agent".
- **The model check** (`--check`, without `--skip-model-check`) plants a `GEMINI.md`, an
  `AGENTS.md` and an `.agents/rules/` file in the folder it reads, each telling the model to answer
  with a marker word. If the marker comes back, the check fails.
- **Sign-in** is checked with `agy models`, which costs nothing and prints "Please sign in" when
  signed out. A signed-out `agy -p` instead opens a browser and waits, so the run never starts
  one without this check.

## Paths with two spellings

If a session reads a path outside its workspace that `agy` cannot ask about, it does not fail
that read; it **ends the whole session with an empty answer**. On macOS, temporary folders have
two spellings (`/var/...` and `/private/var/...`), and `agy` records its working directory in
one while the prompt names the history in the other. So every folder is passed in both
spellings, and reads of the scratch folder holding them are allowed. If a census session still
stops on a refused action (`denied_actions` in its answer), it counts as worth another try.

## Failures

`agy` exits 3 on a model or API error and prints an `AGY_ERROR:` line on stderr. A
`RESOURCE_EXHAUSTED` status counts as a usage limit. `UNAUTHENTICATED`, "Please sign in" and
"Authentication required" count as sign-in failures. `UNAVAILABLE`, `DEADLINE_EXCEEDED` and
`"retryable": true` count as transient, so the census tries again. An unknown `--model` exits 1
with "invalid model selection".

## Not yet verified

- **Windows.** The provider passes `APPDATA` and `LOCALAPPDATA` through, and falls back to a hard
  link when a symbolic link is not allowed. Neither has been run on Windows.
- **Linux desktops with a keyring.** `DBUS_SESSION_BUS_ADDRESS` and `XDG_RUNTIME_DIR` are passed
  through so `agy` can reach it. Headless Linux uses the token file described above. Neither
  path has been signed in and run end to end; the container test only confirmed where the file
  lives.
- **Undocumented controls.** `--gemini_dir`, `inheritCustomizations` and `inheritMcp` do not
  appear in `agy`'s help or docs, and `agy` updates itself. The per-session agent check and the
  model check's instruction canaries exist to catch an update that changes them.
