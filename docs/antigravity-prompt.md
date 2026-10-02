# A prompt to paste into Antigravity

Open the folder that holds all your repositories as the workspace in Antigravity, then paste
everything inside the box below into the agent. It works on macOS, Linux and Windows.

```text
Run hazina-review on every git repository in this workspace, using Antigravity as the AI.

hazina-review measures how much substantial engineering work each repository holds and writes
one zip for me to send to Hazina Labs. It reads the code through Antigravity's command-line tool,
`agy`, under my own account. It takes roughly 15 to 60 minutes per repository. Nothing leaves this
computer until I send the zip myself.

Rules for the whole task:
- Never change, commit, stash or delete anything in the repositories in this workspace. All the
  work happens on fresh clones in a separate folder.
- Never type passwords, sign-in codes or API keys. When something needs a sign-in, ask me to do it.
- Never send, upload or share the zip. I will.
- Do not weaken hazina-review's checks (no --dangerously-skip-permissions, no --skip-model-check).
- Ask me before installing anything.

STEP 1. Folders and repositories
- WORKSPACE = the absolute path of the folder open in this workspace.
- WORK = a folder called hazina-review-antigravity in my home folder (macOS/Linux:
  ~/hazina-review-antigravity, Windows: %USERPROFILE%\hazina-review-antigravity). It must not be
  inside WORKSPACE. Create it if it is not there; if it is, keep what is in it.
- Find the git repositories directly inside WORKSPACE (folders that contain .git; skip hidden
  folders). If WORKSPACE is itself a git repository, it is the only one. Show me the list and the
  count, then carry on.

STEP 2. What hazina-review needs. Check each and install only what is missing (ask me first):
- Python 3.11 or newer. Check: `python3 --version` (Windows: `py -3 --version`).
  Install: macOS `brew install python`, Ubuntu/Debian `sudo apt install python3 python3-venv`,
  Windows `winget install -e --id Python.Python.3.12`.
- git. Check: `git --version`. Install: macOS `brew install git`, Ubuntu/Debian
  `sudo apt install git`, Windows `winget install -e --id Git.Git`.
- pipx. Check: `pipx --version` (Windows: `py -m pipx --version`). Install: macOS
  `brew install pipx`, Ubuntu/Debian `sudo apt install pipx`, Windows
  `py -m pip install --user pipx`. Then run `pipx ensurepath` (Windows: `py -m pipx ensurepath`).
- The Antigravity command-line tool, agy. The Antigravity app alone is not enough. Check:
  `agy --version`. Install: macOS/Linux `curl -fsSL https://antigravity.google/cli/install.sh | bash`,
  Windows PowerShell `irm https://antigravity.google/cli/install.ps1 | iex`.
After installing anything, PATH changes only reach new terminals. Open a new one, or use full
paths: agy is at ~/.local/bin/agy (Windows: %LOCALAPPDATA%\agy\bin\agy.exe), and pipx puts
hazina-review at ~/.local/bin/hazina-review (Windows: %USERPROFILE%\.local\bin\hazina-review.exe).

STEP 3. agy must be signed in. Run `agy models`. If it lists models, carry on. If it says
"Please sign in", stop and ask me to open a terminal, run `agy`, finish the sign-in in the
browser and then type /exit. Wait for me to say it is done, then run `agy models` again.

STEP 4. Install hazina-review. This fork adds Antigravity support:
  pipx install --force git+https://github.com/adoistic/hazina-review.git@v0.1.4-antigravity.1
Then `hazina-review --version` must print `hazina-review 0.1.4+antigravity.1`.

STEP 5. Fresh clones, so the build check never touches my working folders. For each repository
from step 1:
  git clone "<WORKSPACE>/<name>" "<WORK>/repos/<name>"
If that clone folder already exists, leave it as it is. A clone holds only committed work, so
list the repositories that have uncommitted changes (`git -C <repo> status --porcelain`) and
tell me. Do not commit anything for me. Skip any repository that cannot be cloned or has no
commits, and report it.

STEP 6. Check. If <WORK>/hazina-review-out already exists from an earlier run, ask me whether to
resume it (see step 8) or start again with a different --out folder. Then run:
  hazina-review --check --provider antigravity --all "<WORK>/repos" --out "<WORK>/hazina-review-out"
Lines start with ✓ (fine), – (skipped), ! (a warning; the build-check warning is expected) or
✗ (must be fixed). For each ✗, do what the line says and run the check again. If you cannot fix
it, stop and show me the output.

STEP 7. Run. Start it so it keeps going in the background with its output in files:
- macOS/Linux:
    nohup hazina-review --provider antigravity --all "<WORK>/repos" --out "<WORK>/hazina-review-out" > "<WORK>/run.log" 2>&1 &
- Windows (PowerShell):
    Start-Process -WindowStyle Hidden -FilePath "$env:USERPROFILE\.local\bin\hazina-review.exe" -ArgumentList '--provider','antigravity','--all',"`"<WORK>\repos`"",'--out',"`"<WORK>\hazina-review-out`"" -RedirectStandardError "<WORK>\run.log" -RedirectStandardOutput "<WORK>\run-result.log"
  (On Windows the progress goes to run.log and the closing "Zip to send" line to run-result.log.
  If that form fails on this machine, start it in the background any other way that keeps it
  running after this chat and saves its output to files.)
It writes progress lines such as "[3/12] my-repo: task mining done in 4m 10s". Read the end of
the log every 10 minutes or so and give me one line of progress each time. Never start a
second run while one is still going.

STEP 8. If it stops before the end. The last lines of the log say why and what to type:
- "usage limit": tell me. Once the limit resets, start
  `hazina-review --resume "<WORK>/hazina-review-out"` in the background, as in step 7.
- "not signed in" or "refused the sign-in": ask me to sign in as in step 3, then resume as above.
- anything else: show me the last 30 lines of the log and the file
  <WORK>/hazina-review-out/hazina-review-log.txt, and ask me what to do.

STEP 9. When it has finished, the output ends with "Zip to send: <path>". Tell me:
- the zip's path, and the summary of how many repositories completed;
- that the zip holds each repository's name and numbers plus short sentences about the kind of
  work, never the code, and that I should look it over before sending it;
- that I should email it to partners@hazinalabs.com myself.
Leave the WORK folder where it is. You are done.
```

## Notes for whoever hands this out

- The AI use counts against the Antigravity account that `agy` is signed in to.
- The run uses fresh clones in `~/hazina-review-antigravity`, so the build check, which installs
  dependencies and runs each project's tests, can take disk space but never touches the
  originals.
- `hazina-review --resume ~/hazina-review-antigravity/hazina-review-out` carries on after any stop.
  Finished repositories are kept.
