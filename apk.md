---
name: apk-builder
description: Build an Android APK app, and optionally an unsigned iOS IPA, using only Termux and GitHub CLI. Works in phases, logs progress, and writes a context file when done.
---

# APK Builder (Termux + GitHub CLI), Android and iOS

## How to use this file
- Read "Golden rules" and "Style rules" first, every session.
- This skill is split into companion files so that the user attaches only what is needed. See "File map" right below. When a step needs a file the user has not attached, ask for it by name before doing the step. Do not work from memory.
- Then read only what you need: the section of the current phase, the template of the chosen stack, and the error table when something fails.
- Order of this file: rules, versions, files, stack choice, verification, checker, signing key, handoff between AIs, working from a chat AI with a sandbox, phases 0-6, stack templates, debugging, checklists, error handling, resuming.
- If the user attaches skill files, read each attached file in full (every page, no skipping) before any other action. A copy loaded by a tool can be older. Compare the date in "Known-good versions" and use the newer one.
- If the user rejects or interrupts a tool call, stop and ask what to read or do. Do not retry the same call.
- This file is plain text and is the whole skill. Some AIs have no skill feature and no tools: the user attaches or pastes this file and nothing else. First work out what you can do: run commands in a sandbox, read web pages, or only chat. Then take the matching path. With a sandbox, follow "Working from a chat AI with a sandbox". With only chat, give every file and command as copy-paste blocks, ask the user to paste the files and the command output you need, and never say that anything was tested or built.

- First line of the first reply of every session: `Read: <the files you read>`. A skipped file is then visible to the user.

## File map (which file to ask the user for)
| File | Contains | Ask for it when |
|---|---|---|
| `apk.md` (this file) | rules, versions, phases 0-6, handoff, checklists | always attached |
| `capacitor.md` | Capacitor template (skeleton files, `build.yml`) | the chosen stack is Capacitor, before Phase 1 |
| `ui.md` | UI rules, tokens, theme generator, components, screen recipes, icons, motion | Phase 0, to pick the look, and for any screen work from Phase 1 |
| `recipes.md` | Capacitor recipes: movable and resizable pieces, chat with an AI service | Phase 3, only when the app has that feature |
| `errors.md` | error table | the build is red or the app crashes |
| `ios.md` | iOS build (unsigned IPA, `ios.yml`) | the user wants iOS, from Phase 0 |
| `flutter.md` | Flutter template, Flutter look notes | the chosen stack is Flutter, before Phase 1 |
| `kotlin.md` | Kotlin template, Kotlin look notes, crash screen code, Kotlin icon recipe | the chosen stack is Kotlin, before Phase 1 |
| `security.md` | login screens, backend (Supabase), database rules, security checks and tests | the app has login or accounts, a server or database, uploads, or an API key for a paid service. Ask in Phase 0 |

Rules for the file map:
- Phase 0 starts with this file only. Once the scope is clear, ask for `ui.md` (needed to pick the look), and for `ios.md` or `security.md` if they apply. After the stack is chosen, tell the user in one line which extra files to attach before Phase 1 (`capacitor.md`, `flutter.md` or `kotlin.md` for the chosen stack, plus any of `ui.md`, `ios.md`, `security.md` not attached yet).
- Words that mean `security.md` is needed: login, sign in, accounts, users, password, server, database, backend, upload, API key, "secure", "hack". Stop and ask for the file before designing any of it. Never invent a login system or keep a key inside the app.
- If a needed file is missing, say which one and why, then wait. Never invent its content.
- Version numbers always come from "Known-good versions" in this file, even when a companion file mentions a stack.

## Files to read in each phase
Read the files of the current phase in full before any other action. Ask the user for a missing file by name. Start the reply with `Read: <files>`.
| Phase or situation | Read |
|---|---|
| Phase 0 (scope) | `apk.md`. Add `ui.md` to pick the look (ask for it before the scope is approved). Add `ios.md` if iOS is wanted. Add `security.md` if there is login, accounts, a server, a database, uploads or an API key |
| Phase 1 (skeleton) | `apk.md`, `ui.md`, the stack file (`capacitor.md`, `flutter.md` or `kotlin.md`). Add `ios.md` and `security.md` if they apply |
| Phase 2 (first APK) | `apk.md`. Add `errors.md` if the build is red. Add `ios.md` for the iOS build |
| Phase 3 (features) | `apk.md`, `ui.md`, the stack file. Add `security.md` for login or data, `recipes.md` for movable pieces or AI chat, `errors.md` on failure |
| Phase 4 (fix and polish) | `apk.md`, `ui.md`. Add `security.md` if it applies, `flutter.md` if Flutter, `kotlin.md` if Kotlin |
| Phase 5 (release) | `apk.md`. Add `security.md` if it applies, `ios.md` if iOS |
| Phase 6 (context file) | `apk.md` |
| New chat on an existing project | `PROGRESS.md` first, then the files of the recorded phase from this table |

## Environment (the only tools you may assume)
- Termux on Android: bash, git, nano, curl, basic file tools
- GitHub CLI, installed in Termux with `pkg install gh git` if missing
- Builds run on GitHub Actions, never locally. Termux has no Android SDK and cannot generate a Gradle wrapper. The GitHub Actions runner already has the Android SDK.
- Get the APK with `gh run download` or `gh release download`.
- iOS (optional): built on a macOS runner on GitHub Actions. Termux cannot build or install iOS apps. Only Capacitor and Flutter can target iOS. See `ios.md`.
- A backend is never assumed. It exists only when the user asks for accounts, a database or a server, and then `security.md` decides how: the user's own Supabase project, created in the dashboard in the phone browser, with the database kept as SQL files in the repo.
- Do not assume Node, Flutter, Gradle or Android tools exist in Termux. The only extra package ever needed locally is `openjdk-17`, for one command (`keytool`, see Signing key).

## Style rules
- Replies: short, direct, no recaps. Max ~6 lines unless showing code.
- One phase at a time. Do not start the next phase until the user confirms the current one works.
- Ask before starting if the request is unclear (max 3 questions).
- In explanations and file text, write "GitHub" and "GitHub CLI" in full, never the shorthand "gh". The only exception is command text written in backticks or code blocks, because the shell needs it exactly.
- Give the user one copy-paste block per step. Each block must work on its own, from the repo folder.
- No stickers, ever. No emoji, emoticons, decorative symbols, or check mark and cross icons. This holds in chat replies, in files, in commit messages and in the text of the app. Use plain words (Verified, Not verified) and plain punctuation. In an app, use line icons (SVG), never emoji. The only exception is an explicit request from the user in the current message.
- Report format after every finished step, in this order, plain text, no headings, no tables: what was done and which files changed; Verified; Not verified; Known issues or Known risks (only when there are any); Next. Then the one copy-paste block. Keep the text before the block to about 6 lines.
- Auto-download: when a block pushes a change that starts a build, the block must end with commands that wait for the run and save the result into `~/storage/downloads/` by themselves, so the user never types a second command. This replaces the last line (`gh run list --limit 1`) of the block template. The run is found by the commit that was just pushed (never "the latest run", which can be the previous push when GitHub starts the new run late), so the block is placed after `git push` and uses `git rev-parse HEAD`. Never put `exit` in a block the user pastes into Termux: it closes the session. Use exactly:
```
[ -d ~/storage/downloads ] || termux-setup-storage
SHA=$(git rev-parse HEAD)
RUN=""
for i in 1 2 3 4 5 6 7 8 9 10 11 12; do
  RUN=$(gh run list --workflow build.yml --commit "$SHA" --limit 1 --json databaseId --jq '.[0].databaseId')
  [ -n "$RUN" ] && break
  sleep 5
done
[ -n "$RUN" ] || echo "No run found for this commit yet. Check the Actions tab."
[ -n "$RUN" ] && gh run watch "$RUN" --exit-status && rm -rf ~/storage/downloads/app-debug && gh run download "$RUN" -n app-debug -D ~/storage/downloads/app-debug
```
- A document the user asked for (for example a `.md` file) is written with `cat > file <<'EOF'` in the same block and then copied with `cp file ~/storage/downloads/`, so it lands on the phone without a manual step.
- If `termux-setup-storage` asks for permission, the user taps Allow once and runs the block again.
- Version on every update: every build the user will install gets a higher version, in the same commit as the change. Raise `versionCode` by 1 and update `versionName` (file locations: see "Where to bump the version" in Phase 5). Also show the version inside the app (one small line on the main screen or in settings) so the user can see which build is installed. After a green build, tell the user the exact version to look for. Without this the user cannot tell whether the new APK was installed.
- Reply in the language the user writes in. Commands, file names and code stay in English. Text shown inside the app is in the user's language.
- Version text: copy the version string from the code (for example the `APP_VERSION` line) when telling the user what to look for. Never type it from memory. Record where it lives in `PROGRESS.md`.
- A big change goes in a script file, not in the chat (see "Working from a chat AI with a sandbox", Script delivery). A small change stays as one copy-paste block.

## Golden rules (these prevent most errors)
1. Never guess a version number, action tag, package name or file path. Use the "Known-good versions" table, or look it up (see that section).
2. Copy the stack template exactly first. Change names and ids only. Add features after the first build is green.
3. Write every file with a quoted heredoc: `cat > path <<'EOF'` ... `EOF`. The quotes around EOF matter. Without them the shell expands `$HOME`, `$GITHUB_OUTPUT` and `${{ secrets.X }}` and breaks the file.
4. Workflow files (YAML): spaces only, never tabs, 2 spaces per level. One wrong indent makes the whole workflow invalid.
5. Before every push run `git pull --rebase`. The first build commits the `android` folder back to the repo (Capacitor and Flutter), so the local copy is behind.
6. When a build fails: read the failed log first (`gh run view <id> --log-failed`), find the first real error, change one thing, push, check again. Never change several things at once.
7. Never say "works", "fixed" or "done" without evidence: a green run, and the user's confirmation for anything visible on the phone.
8. Never commit keystores, passwords or tokens. Secrets only through `gh secret set`.
9. Choose the app id once (for example `com.example.todo`: lowercase letters and digits, dots between parts, no hyphens, each part starts with a letter). Changing it later is painful.
10. Baseline security, always, even before `security.md` is attached: the app is never trusted (anyone can unpack an APK), so no secret or paid key inside the app; never write your own login or password handling (use the backend's auth); show user text with `textContent`, never `innerHTML`; every table with user data has Row Level Security. If any of this is in play, ask for `security.md` before designing it.
11. If a rule here conflicts with what the user explicitly asks, tell the user the risk in one line, then follow the user.

## Known-good versions (checked on 2026-10-10, re-verified 2026-10-11)
Re-verified on 2026-10-11 by generating a Capacitor 8.5 project (Gradle 8.14.3, AGP 8.13.0, compileSdk 36, minSdk 24 confirmed) and listing the tags of the actions, Gradle and Kotlin (all exist). Sources: Capacitor 8 upgrade guide, Android Gradle plugin release page, Kotlin Gradle plugin compatibility page, release pages of the actions. The templates below follow these. The first build in Phase 2 is the real test.

| Item | Capacitor | Flutter | Kotlin (native) |
|---|---|---|---|
| Java in the workflow (`actions/setup-java`) | 21 (Temurin) | 21 (Temurin) | 21 (Temurin) |
| Node | 22 (Capacitor 8 needs 22 or higher) | not used | not used |
| Gradle | project wrapper, 8.14.3 in the Capacitor 8 template | wrapper made by Flutter | 8.14.3, downloaded in the workflow (one line, `GRADLE_VERSION` in `build.yml`) |
| Android Gradle plugin | set by the template, 8.13.0 in Capacitor 8 | set by the template | 8.13.0 |
| Kotlin | not used | not used | 2.3.21 |
| compileSdk / targetSdk | 36 / 36 (template) | Flutter default | 36 / 36 |
| minSdk | 24 (template) | Flutter default | 26 |

Action tags used in the templates: `actions/checkout@v6`, `actions/setup-java@v6`, `actions/setup-node@v7`, `actions/upload-artifact@v7`, `subosito/flutter-action@v2`.

Android Gradle plugin and minimum Gradle (from the Android release page):

| AGP | Minimum Gradle |
|---|---|
| 8.6, 8.5 | 8.7 |
| 8.7 | 8.9 |
| 8.8 | 8.10.2 |
| 8.9, 8.10 | 8.11.1 |
| 8.11, 8.12, 8.13 | 8.13 |
| 9.4 | 9.6.0 |

- compileSdk 36 needs AGP 8.9.1 or higher.
- AGP 9 changes how Kotlin is applied. The Kotlin template targets AGP 8.13.0. Do not move to AGP 9 unless the user asks.
- Re-check rule: if this table is older than 3 months, or a build fails with a version error, look up the newest tag of each action before changing anything:
  `gh release list -R actions/checkout -L 3` (same for `actions/setup-java`, `actions/setup-node`, `actions/upload-artifact`, `subosito/flutter-action`).
- If a build says `Unable to resolve action`, that tag does not exist. Use the newest major from the release list.
- Record any version you change in `PROGRESS.md`.

iOS (added 2026-10-10, NOT yet proven by a build: the first iOS run is the real test):
- Runner: `macos-latest`. Xcode: the workflow picks the newest non-beta `Xcode_*.app` found on the runner (never a guessed path) and prints the version. Capacitor 8 needs a recent Xcode (believed to be 26 or higher, not verified): read the printed version if the build fails.
- Capacitor iOS package: `@capacitor/ios` with the same major as `@capacitor/core` (`^8.0.0`).
- Same action tags as above. Record any change in `PROGRESS.md`.

## Files you maintain (in the repo root)
- `PROGRESS.md` — update at the end of EVERY phase and after every fix:
  - Current phase, done list, next step, known bugs, last run ID and status
  - Template: see "Report and file block" below.
- `SKILL.md` — a copy of this file, kept in the repo root so any AI can read it (see "Handoff between AIs").
- `CONTEXT.md` — written when the app is finished (see Phase 6, which holds the template).
- `.gitignore` — create in Phase 1 (build folders, local properties, keystores). Per-stack content is in the templates.
- `LESSONS.md` — one short entry each time something unexpected happened (date, symptom, cause, fix). The user folds these into the error table (`errors.md`) of this skill.
- `NOTICE.md` — only if code from another project is copied (see Built-in checker).

The shape of `PROGRESS.md` is the template in "Report and file block after EVERY finished step" below. A filled scope example is in Phase 0.

## Choosing the stack (recommend one, with a reason, in Phase 0)
| Stack | Language | Best for | Typical size of a small app |
|---|---|---|---|
| Capacitor (web app packed as Android app) | HTML, CSS, JavaScript | Simple tools, forms, lists, dashboards. Easiest to build and fix. | about 8-15 files |
| Flutter | Dart | Polished custom interface, animations, one codebase | about 10-20 files |
| Kotlin (native) | Kotlin | Camera, sensors, background services, video or audio, heavy performance | about 15-30 files |

- Default recommendation: Capacitor for simple apps, Kotlin when the app needs deep Android features or speed.
- If the user wants iOS too: only Capacitor or Flutter. Kotlin (native) is Android only. Say this in Phase 0.
- Tell the user the pick and why in 2 lines, then let them change it.
- Keep each file under about 300 lines with one job. If a file grows past that, split it.
- List the planned file tree in `PROGRESS.md` in Phase 1 and keep it current.

Example of the 2-line recommendation:
"Stack: Capacitor. The app is lists and forms, and a web page is the easiest to build and fix. Want Flutter or Kotlin instead?"

## Verification (never claim "works" without evidence)
- In Phase 0, write one testable line for each feature: "Done when: <what the user sees or does>". Store them in `PROGRESS.md`.
- Automatic checks in the workflow: build succeeds, lint and unit tests run (if the stack has them), and the APK file exists in the artifact. Use the built-in checker (see section below).
- Manual checks: the user installs the APK and runs through each "Done when" line.
- Report status in three parts: Verified (build green, tests passed), Not verified (needs the user to test), Known issues.
- A feature counts as done only when the build is green AND the user confirms it.
- After each fix, rebuild and re-check the earlier features still work.

Good "Done when" lines are observable, one action, one result:
- "Done when: tap Add with text 'milk', and 'milk' shows in the list."
- "Done when: close the app from recent apps, open it again, and the list is still there."
- "Done when: with Wi-Fi off, the app shows the message 'No connection' and does not crash."
Bad (never write): "works well", "looks nice", "is fast", "has no bugs".


## Built-in checker (reuse open-source tools, license first)
Goal: the project has its own automatic check step. Do not write checkers from scratch if a good open-source one exists.

Default checks that need no extra tools and no license review, because they come with the stack:
| Stack | Default check step |
|---|---|
| Capacitor | `for f in www/*.js; do node --check "$f"; done` (JavaScript syntax check, built into Node) |
| Flutter | `flutter analyze` and `flutter test` |
| Kotlin | `gradle lintDebug` (Android lint) and `gradle testDebugUnitTest` |

These are already in the templates below. Add a third-party checker only when the user wants stricter checks.

Steps, in order:
1. Search GitHub with GitHub CLI for checkers that fit the stack (code style, static analysis, tests). The license filter takes one license per search, so run it once per allowed license, or search without it and read the license afterwards. Examples:
   - `gh search repos <keywords> --license mit --sort stars --limit 10`
   - `gh search repos <keywords> --license apache-2.0 --sort stars --limit 10`
   - `gh search repos <keywords> --license bsd-3-clause --sort stars --limit 10`
2. Pick candidates that are popular, recently updated, and still maintained.
3. READ THE LICENSE BEFORE USING ANYTHING. Command: `gh api repos/<owner>/<name>/license` (or open the LICENSE file in the repo). Confirm what it allows.
4. Allowed without asking: permissive licenses (MIT, Apache-2.0, BSD).
5. Ask the user first: copyleft licenses (GPL, AGPL, LGPL). No license file = do not use.
6. Prefer running the tool as a workflow step or dependency over copying its source code.
7. If you do copy or adapt any code: keep the original copyright and license notice, and record it in a `NOTICE.md` (tool name, repo link, license, what was used).
8. Add the checker to `.github/workflows/build.yml` so it runs on every push, and note it in `PROGRESS.md` and `CONTEXT.md`.
9. Strictness: in Phase 1 the checker is non-blocking (`continue-on-error: true`) so the skeleton build is never stopped by style warnings. In Phase 4, remove that line so the checker blocks the build.

Examples to look at first (confirm each license yourself, do not trust this list): the built-in Android lint, ktlint and detekt for Kotlin, ESLint for JavaScript, the built-in analyzer for Flutter.


## Signing key (one fixed key for the whole project)
Problem: GitHub Actions creates a fresh debug key on every run, so a new APK cannot install over the old one ("App not installed") and app data is lost. Fix: create one key in Phase 1 and reuse it in every build.

Create it once in Termux, inside the repo folder (needs `pkg install openjdk-17`):
```
keytool -genkeypair -keystore debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000 -dname "CN=Android Debug,O=Android,C=US"
base64 -w0 debug.keystore | gh secret set DEBUG_KEYSTORE_B64
rm debug.keystore
gh secret list
```
- The last command must list `DEBUG_KEYSTORE_B64`. If it does not, the secret was not saved. Check that you are inside the repo folder (`git remote -v` shows the GitHub repo) and repeat.
- `debug.keystore` is also listed in `.gitignore`. Never commit it.
- If the secret is ever lost or replaced, the new key does not match the installed app. The user must uninstall the old app once before installing the new APK.

Workflow step, placed before the build step, puts the key where Gradle looks for the debug key:
```
- name: Restore signing key
  env:
    KEYSTORE_B64: ${{ secrets.DEBUG_KEYSTORE_B64 }}
  run: |
    mkdir -p ~/.android
    echo "$KEYSTORE_B64" | base64 -d > ~/.android/debug.keystore
```

## Handoff between AIs (the repo is the memory)
The user may run out of tokens and continue with another AI, or use several AIs on the same project. Nothing lives in the chat. Everything lives in the GitHub repo.
- Keep `SKILL.md`, `PROGRESS.md` and (when it exists) `CONTEXT.md` in the repo root. Push them like any other file.
- A chat AI cannot push. The user runs the commands in Termux. Give the user one copy-paste block that writes `PROGRESS.md` and pushes.
- At the end of every phase and every fix: follow "Report and file block after EVERY finished step" (report in chat, plus the `cat` and `EOF` block that updates `PROGRESS.md` and pushes).
- If the user says tokens are low, or that another AI will continue: update `PROGRESS.md` first, before anything else.
- Private repo: the new AI cannot read it. The user pastes `SKILL.md` and `PROGRESS.md` (and any file in question) into the new chat. Public repo: an AI with web access can read the raw files.
- A new AI starts like this:
  1. Read `SKILL.md` and `PROGRESS.md`.
  2. Ask the user for `git log -5 --stat` and the last run status (`gh run list --limit 3`).
  3. Do not trust "done" or "fixed" in the notes without a green run (Golden rule 7). Check first.
  4. Do not redo finished work. Do not rewrite whole files. Change one thing at a time.
  5. Write its own name and date in the "Handoff" block before it finishes.
- Record any version or file change in `PROGRESS.md`. The next AI relies on it.

### Report and file block after EVERY finished step (mandatory)
Every time a step, feature or fix is finished, the reply must contain both parts below. No exceptions, also for tiny changes.
1. A short report in the chat: what was done, which files changed, what is verified, what is not verified, the next step.
2. One copy-paste block that writes `PROGRESS.md` with `cat` and a quoted `EOF`, then commits and pushes. The user runs it in Termux from the repo folder.

First time (no `PROGRESS.md` yet): create it in the same block. It must contain: scope and app id, features with "Done when", status, the current file tree (every file that exists now, one line each with its purpose), the Handoff block, known bugs, versions. Use the template below as the shape.

Later times: rewrite the whole `PROGRESS.md` with the new state (never leave old status lines). Update the file tree if a file was added, removed or renamed.

Template of the block (change only the content between the EOF lines and the commit message):
```
cat > PROGRESS.md <<'EOF'
# Progress
Read SKILL.md first, then this file.

## Scope
App: <name and one line>. Stack: <stack>.
App id: <id>
Platforms: Android (iOS: <yes/no>)
Look: <hue, sat, characters, mode>
Features and "Done when":
1. <feature>: <what the user sees>

## Status
Phase: <n> (<name>)
Last run: <id> - <success/failure/not started>
Done: <list>
Next: <one concrete step>

## Handoff
Last worked by: <AI name>, <date>
Changed: <files and what>
Not tested: <list>

## File tree (current)
- <path>: <purpose>

## Known bugs
- (none)

## Versions
- <Node, Java, Capacitor, and so on>
EOF
git add -A
git pull --rebase
git commit -m "docs: update PROGRESS.md (<what was done>)"
git push
gh run list --limit 1
```
Rules for the block:
- Every file the AI creates or changes in that step is written in the same block with `cat > path <<'EOF'` ... `EOF`, before `git add -A`.
- The report in the chat and the content of `PROGRESS.md` must say the same thing.
- If the user's tokens may run out, put this block first in the reply.

## Working from a chat AI with a sandbox (added 2026-10-11, from a real session)
Applies when the AI has its own Linux sandbox (it can clone, edit files and run tests) but the user builds in Termux. The AI never pushes. It prepares and tests the change, then the user runs one command.

### Reading an existing repo
- A web fetch tool returns a summary of a page, not the exact file text. Never edit a file from a summary.
- Public repo: clone it in the sandbox (`git clone https://github.com/<owner>/<repo>.git`) and read the real files. Private repo: ask the user to paste the files (or the `cat` output) you need.
- Read `PROGRESS.md` first (and `CONTEXT.md` if present), then only the files the change touches. Tell the user in two lines what the project already has, so they can correct you before you start.
- If the user asked to read the skill first, do that before cloning or running anything.
- Before editing, reset the sandbox clone to the pushed state (`git fetch`, then `git reset --hard origin/main`) so it matches what the user has.

### Test before handing over
- Syntax: `node --check` on every JavaScript file (Capacitor).
- Dry run: write the script to a file, clone the repo again into an empty folder, run the script there without the `git` and `gh` lines, run the syntax check, then `diff -r` the result against the tested working copy. Only the files meant to differ (for example `PROGRESS.md`) may differ.
- Behavior: if the sandbox has Chromium and Playwright, serve `www/` with a local web server, open it in a phone sized window (390 by 800), and drive the feature with mouse and keyboard events. Mock calls to outside services with `page.route`, so no real key is needed. Check the browser console for errors.
- Say exactly what was tested: "Verified in desktop Chromium with a mouse. Not verified: touch on the real phone, the real service". Do not write "works".

### Script delivery
- A change that needs more than about 150 lines of text is delivered as one file (for example `note-1.7.0.sh`), sent to the user, plus one line to run it: `bash ~/storage/downloads/note-1.7.0.sh`. The script holds every `cat > path <<'EOF'` block, the `sed` edits, the `PROGRESS.md` rewrite, the commit, the push and the Auto-download lines.
- First lines of every script. They stop the script when it runs in the wrong place, instead of changing the wrong folder:
```
set -e
ROOT=$(git rev-parse --show-toplevel 2>/dev/null) || { echo "Run this inside the repo folder"; exit 1; }
cd "$ROOT"
git remote get-url origin | grep -q "<owner>/<repo>" || { echo "This is not the <repo> repo"; exit 1; }
git pull --rebase
```
- Do not guess the folder name (`cd ~/Note`). The user's folder can be elsewhere.
- Check the script with `bash -n script.sh` before sending it.
- Small edit to an existing file: `sed -i` with a pattern that matches exactly one line, or `cat >> file <<'EOF'` to add at the end. Run the exact command on a fresh clone and read `git diff`. A file that changes in many places is rewritten whole with a quoted heredoc.
- Put a separate feature in a new file (for example `www/chat.js`), so the change to old files stays small. Add its `<script>` tag to `index.html` after the files it uses.

## Phases

### Phase 0: Scope
- Confirm: app goal, 3-5 must-have features each with a "Done when" line, tech stack (see Choosing the stack), platforms (Android, iOS or both)
- If iOS is wanted: ask the user to attach `ios.md`, read it, and tell the user its limits (unsigned IPA, sideloading, no testing by Claude) before starting.
- Ask: does the app have accounts or login, a server or database, file uploads, or an API key for a paid service? If yes, ask the user to attach `security.md`, read it, and write its Security section into `PROGRESS.md` before the scope is approved.
- Check setup: `gh auth status` works. To push workflow files the login needs the workflow permission: `gh auth refresh -s workflow`
- Set the git identity once, or the first commit fails: `git config --global user.name "<name>"` and `git config --global user.email "<email>"`
- Make later pulls simple: `git config --global pull.rebase true`
- Pick the look: choose the hue (by mood), the character and the mode from `ui.md` (any kind of app works; ask the user to attach `ui.md` now if it is missing) and confirm it with the user in one line.
- Create `PROGRESS.md` with the scope and the look.
- Done when: user approves scope.

Example scope reply:
```
Scope: to-do list app, Capacitor, id com.example.todo, look calm (hue 265, soft).
1 Add task. 2 Complete task. 3 Delete task. 4 Tasks survive restart.
Approve? Then I start Phase 1.
```

Setup commands for a fresh Termux, in order:
```
pkg update
pkg install git gh openjdk-17
termux-setup-storage
gh auth login
gh auth refresh -s workflow
gh auth status
```
- In `gh auth login` choose: GitHub.com, HTTPS, login with a web browser. It shows a code and a link. The user opens the link in the phone browser and enters the code.

### Phase 1: Repo + skeleton
- `gh repo create <name> --private --clone`
- Minimal app that shows one screen. Set a unique application ID.
- Create the fixed signing key (see Signing key section) and store it as a secret.
- Add `.github/workflows/build.yml`:
  - Triggers: push, manual run (`workflow_dispatch`)
  - Use the template of the chosen stack (Capacitor: `capacitor.md`; Flutter: `flutter.md`; Kotlin: `kotlin.md`). Do not write the workflow from memory.
- Set up the built-in checker, non-blocking for now (license read first, see section above)
- If `security.md` applies: also add its `security.yml` workflow and `scripts/check-security.sh`, non-blocking for now.
- Copy this skill file into the repo root as `SKILL.md` (the user has it on the phone, for example `cp ~/storage/downloads/apk.md SKILL.md`). The companion files stay with the user: the AI asks for them again by name when needed. `PROGRESS.md` says "Read SKILL.md first", so the file must exist.
- Commit, push
- Note: private repos use the free monthly GitHub Actions minutes. Avoid pointless rebuilds.
- Done when: workflow is triggered (`gh run list`)

Command order (keep this order, the secret must exist before the first push):
```
cd ~
gh repo create myapp --private --clone
cd myapp
git branch -M main
```
Then: create the signing key and secret, create the files from the template (quoted heredocs), then:
```
git add -A
git status
git commit -m "feat: project skeleton"
git push -u origin main
gh run list --limit 3
```
- Read `git status` before committing: no keystore, no `node_modules`, no `build` folders.
- If `gh run list` shows nothing after 1 minute, see the error table in `errors.md` ("workflow does not start"; ask the user to attach it).

### Phase 2: First APK
- Watch: `gh run watch`
- If failed: `gh run view --log-failed`, fix, push, repeat
- Download: `gh run download -n <artifact>`
- Install: run `termux-setup-storage` once, then open the file with `termux-open <file>.apk`. Android asks to allow installs from Termux the first time.
- If `termux-open` cannot open the file: copy it with `cp <file>.apk ~/storage/downloads/` and open it from the phone's file app.
- Done when: user confirms the app opens.
- iOS: separate step after the Android build is green (see `ios.md`).

Exact commands:
```
SHA=$(git rev-parse HEAD)
git pull --rebase
RUN=""
for i in 1 2 3 4 5 6 7 8 9 10 11 12; do
  RUN=$(gh run list --workflow build.yml --commit "$SHA" --limit 1 --json databaseId --jq '.[0].databaseId')
  [ -n "$RUN" ] && break
  sleep 5
done
[ -n "$RUN" ] || echo "No run found for this commit yet. Check the Actions tab."
[ -n "$RUN" ] && gh run watch "$RUN" --exit-status && rm -rf ~/storage/downloads/app-debug && gh run download "$RUN" -n app-debug -D ~/storage/downloads/app-debug && termux-open ~/storage/downloads/app-debug/*.apk
```
- `git pull --rebase` is needed because on Capacitor and Flutter the first build adds the `android` folder to the repo. `SHA` is taken before it on purpose: that bot commit does not start a run, so searching for it would find nothing.
- The first Flutter build can take 5-10 minutes. That is normal.
- If the app asks "Install unknown apps", the user must allow it for Termux (or for the file app used to open the APK).

### Phase 3: Features (loop, one feature each)
- Implement one feature -> push -> build -> download -> user tests
- Update `PROGRESS.md` after each
- Commit message format: `feat: <short description>`
- Done when: every feature's "Done when" line is confirmed by the user.

Per-feature loop:
```
git pull --rebase
# edit the files for this feature (small change, one feature only)
git add -A
git commit -m "feat: add task list"
git push
```
Then watch the run, download, and ask the user to test exactly the "Done when" line of this feature. Re-test one earlier feature too.

Commit types: `feat`, `fix`, `chore`, `ci`, `docs`, `refactor`.

### Phase 4: Fix and polish
- Fix bugs listed in `PROGRESS.md`
- Check: permissions, screen rotation, back button, app icon and name
- Make the checker blocking (remove `continue-on-error: true`) and fix what it reports
- If `security.md` applies: make its checks blocking too, run the access test and the manual checks from that file, and report Verified, Not verified and Known risks.
- Done when: user says no known bugs remain.

Polish checklist (check each, one line of evidence per item):
- Permissions: only the ones the app really uses are in the manifest.
- Rotation: turn the phone sideways, nothing is cut off or lost.
- Back button: goes back one screen, and closes the app only on the first screen.
- App name: shown under the icon is the real name, not a placeholder.
- App icon: not the default one (see icon recipe below).
- Empty states: the screen with no data shows a clear message.
- Errors: no network, empty input and double taps do not crash the app.
- Look: every color comes from a variable, one filled primary button per screen, spacing from the scale (see `ui.md`).
- Contrast: text is readable in light mode and in dark mode (ask for one screenshot of each).
- Icons: line SVG icons at 24 px inside 48 px buttons, navigation items have icon and label (see "Icons").
- Flutter: the version is pinned (done after the first green build, see `flutter.md`) and `PROGRESS.md` names it.

Where to change the app name:
- Capacitor: `appName` in `capacitor.config.json`, and `app_name` plus `title_activity_main` in `android/app/src/main/res/values/strings.xml`.
- Flutter: `android:label` in `android/app/src/main/AndroidManifest.xml`.
- Kotlin: `android:label` in `app/src/main/AndroidManifest.xml`.
- iOS: see `ios.md` (App name).
- Read the file before editing it. Do not guess its content.

Icon recipe (text files only, works in Termux). Kotlin: the three files and the manifest line are in `kotlin.md` (section "Icon recipe").
- Capacitor and Flutter already have icon files under `android/app/src/main/res`. Read the existing `mipmap-anydpi-v26/ic_launcher.xml` first, keep the names it references, and replace only the files it points to.

### Phase 5: Release
- Bump the app version number
- Remove the development error box and push, so the release has none (Capacitor: the error script, the `err` element and its CSS rule; Kotlin: the crash screen code).
- The templates already contain the release step (it runs only for tags that start with `v`). Do not add another one.
- Tag and push: `git tag v1.0.0 && git push origin v1.0.0`
- The release job needs permission to write, or it fails with a 403 error. The templates already set `permissions: contents: write` at the top of the workflow, and `GH_TOKEN` on the release step.
- Workflow builds the APK and publishes it as a release
- Signing: for personal use, the fixed key from Phase 1 is fine. For a separate release key, generate it with `keytool`, store it with `gh secret set`, and never commit it.
- If `security.md` applies: do its Phase 5 steps first (error box and `'unsafe-inline'` out, no secret key anywhere, sign-in still works on the release build).
- Done when: the release APK downloads and installs.
- iOS: the same `v*` tag also runs `ios.yml`, which adds the unsigned IPA to the same release (see `ios.md`).

Where to bump the version (raise `versionCode` by 1 every release, a lower number cannot be installed over a higher one):
- Capacitor: `versionCode` and `versionName` in `android/app/build.gradle`
- Flutter: `version: 1.0.1+2` in `pubspec.yaml` (name before the plus, code after it)
- Kotlin: `versionCode` and `versionName` in `app/build.gradle.kts`
- iOS: see `ios.md` (Version).

Bump commands (paste-safe: they never call `exit`, and they edit only when the file has exactly one of each line, otherwise they say so and change nothing). Set the two values first. The Capacitor line format is from memory and not proven until the first run: the last command prints the result, so read it.
```
NEW_CODE=2
NEW_NAME=1.0.1
F=android/app/build.gradle   # Kotlin: F=app/build.gradle.kts
if [ "$(grep -cE '^[[:space:]]*versionCode' "$F")" = 1 ] && [ "$(grep -cE '^[[:space:]]*versionName' "$F")" = 1 ]; then
  sed -i -E "s/^([[:space:]]*versionCode[[:space:]]*=?[[:space:]]*)[0-9]+/\1${NEW_CODE}/; s/^([[:space:]]*versionName[[:space:]]*=?[[:space:]]*).*/\1\"${NEW_NAME}\"/" "$F"
else
  echo "Expected one versionCode and one versionName line in $F. Read the file and edit it by hand."
fi
grep -nE 'versionCode|versionName' "$F"
```
Flutter (one line, name before the plus, code after it):
```
NEW_CODE=2
NEW_NAME=1.0.1
if [ "$(grep -c '^version:' pubspec.yaml)" = 1 ]; then
  sed -i -E "s/^version:.*/version: ${NEW_NAME}+${NEW_CODE}/" pubspec.yaml
else
  echo "Expected one version line in pubspec.yaml. Edit it by hand."
fi
grep -n '^version:' pubspec.yaml
```
Also change the `APP_VERSION` line in the app code to the same name (Style rules, "Version on every update").

Release commands:
```
git pull --rebase
git add -A
git commit -m "chore: bump version to 1.0.1"
git push
git tag v1.0.1
git push origin v1.0.1
RUN=""
for i in 1 2 3 4 5 6 7 8 9 10 11 12; do
  RUN=$(gh run list --workflow build.yml --branch v1.0.1 --limit 1 --json databaseId --jq '.[0].databaseId')
  [ -n "$RUN" ] && break
  sleep 5
done
[ -n "$RUN" ] || echo "No run found for tag v1.0.1 yet. Run: gh run list --limit 5"
[ -n "$RUN" ] && gh run watch "$RUN" --exit-status && gh release download v1.0.1 -p "*.apk" -D ~/storage/downloads --clobber
```
- A tag starts two runs (`build.yml` and `ios.yml`), and the push to main before it starts a third on the same commit. So the run is found by the tag name (`--branch v1.0.1`: a tag run reports the tag as its branch), never by commit or as "the latest". Believed correct, not proven until the first release: if the loop finds nothing, read `gh run list --limit 5` and pass the run ID by hand. Always pass the run ID, never call `gh run watch` without one: it asks an interactive question.
- Wait until the run for the tag is green before downloading.
- The release contains the APK signed with the fixed key. It is meant for personal use and for sharing by link. A store release needs a separate release key and is out of scope unless the user asks.

### Phase 6: Context file
- Write `CONTEXT.md` with:
  - What the app does
  - Tech stack and project structure (key files, one line each)
  - How to build (workflow names, how to download the APK and the IPA, which sideloading tool the user uses)
  - Features done, known limits, ideas for next version
  - Useful commands (GitHub CLI and git)
- Template (write it with a quoted heredoc, like `PROGRESS.md`; fill every placeholder from the repo, never from memory):
```
cat > CONTEXT.md <<'EOF'
# Context
Read SKILL.md first, then this file.

## What the app does
<2-3 lines>

## Stack and structure
Stack: <stack>. App id: <id>. Platforms: Android (iOS: <yes/no>)
- <path>: <purpose, one line per key file>

## How to build
- Workflow: build.yml (iOS: ios.yml, by hand or on v* tags)
- APK: <how the user downloads it, for example the Auto-download block or the release page>
- iOS: <sideloading tool the user uses, or not used>
- Version shown in the app: <where the APP_VERSION line lives>

## Features done
1. <feature>: confirmed by the user on <date>

## Known limits and ideas
- <limit or idea>

## Useful commands
- gh run list --limit 3
- gh run view <id> --log-failed
- git pull --rebase
EOF
```
- Commit and push `CONTEXT.md` and final `PROGRESS.md`
- Done when: files are pushed. Tell the user to give `CONTEXT.md` to the AI at the start of any new chat.


## Stack templates
The templates are in companion files: `capacitor.md`, `flutter.md`, `kotlin.md`.
Replace the placeholders: `demoapp` (project name, lowercase), `Demo App` (display name), `com.example.demoapp` (app id). Use the same values in every file.

## Debugging without a computer
Termux cannot read the logs of another app, so the app must show its own errors.
- Capacitor: the template shows JavaScript errors in red on the screen. Ask the user for a screenshot.
- Flutter: debug APK shows a red error screen. Ask for a screenshot.
- Kotlin: add the crash screen code from `kotlin.md` (section "Crash screen") at the top of `onCreate` while developing. A crash is saved to a file and shown on the next launch.
- iOS: Capacitor shows the same red JavaScript errors on screen. Flutter iOS is a release build, so there is no red error screen: add a visible error text in the app (for example a `FlutterError.onError` handler that stores the message and shows it) and ask for a screenshot.
- Ask the user: what they tapped, what they expected, what happened, and the screenshot. Then reproduce the cause in the code before changing anything.
- If the app does not install: check the phone's Android version with `getprop ro.build.version.sdk` (Termux) and compare it with `minSdk`.

## Checklists
Before every push:
1. `git status`: only the files you meant to change. No keystore, no `node_modules`, no build folders.
2. YAML files: spaces only, same indent as the template.
3. App id, name and file paths are identical everywhere they appear.
4. `PROGRESS.md` updated, and `versionCode` raised by 1 with `versionName` updated (see Style rules).
5. `git pull --rebase`
6. `git commit -m "<type>: <short description>"`, then `git push`
7. `gh run list --limit 1` to confirm the run started.

After a green build:
1. The artifact contains the APK (the step "List APK files" shows it).
2. Tell the user: Verified, Not verified, Known issues.
3. Name the exact "Done when" line the user should test.
4. Update `PROGRESS.md` with the run ID.
5. Name the version the user should see in the app after installing it.

Before a release:
1. Every "Done when" line confirmed by the user.
2. `versionCode` raised by 1, `versionName` updated.
3. Checker is blocking and green (Phase 4).
4. Tag starts with `v`.

## Error handling
- Build failed: read the failed log only, quote the key error in 1-2 lines, fix. Use the error table first (`errors.md`; ask the user to attach it if missing).
- Same error 3 times: stop, summarize what was tried in `PROGRESS.md`, ask the user.

## Resuming
- Saving tokens: a long chat re-sends everything on every message. When the user confirms a phase is done, say in one line that a new chat is cheaper, and that they attach only `apk.md`, the files of the next phase (see "Files to read in each phase") and `PROGRESS.md`.
- At the start of any chat, ask the user for the companion files of the recorded phase (see "Files to read in each phase"), then read `PROGRESS.md` (and `CONTEXT.md` if present), then continue from the recorded phase. If you are a different AI than the last one, follow "Handoff between AIs".
- If the repo is not on the device, clone it first: `gh repo clone <owner>/<name>`
- Run `git pull --rebase` before the first change. Re-check the Known-good versions date.
