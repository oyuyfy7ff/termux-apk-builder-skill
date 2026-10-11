# security.md (companion file of apk.md)

Use when: the app has any of these: login or accounts, a server or database, file uploads, or an API key for a paid service. Needed from Phase 0, before any of it is designed.
Read apk.md first. It holds the rules and the phases. This file adds the security part and is not complete alone. For screens use `ui.md` as well.

Stack coverage: the login screens, the content security policy, the library setup and the pattern checks below are written for Capacitor. For Flutter or Kotlin, rules 1 to 12, the Supabase section, the database template, the access test and the manual checks still apply, but the screens and client setup are not written yet. Tell the user this, use the backend's official Flutter or Android library, look its current setup up before writing code, and never invent it.

## Security rules (accounts, servers, databases)

### Scope and honesty
- This file reduces the common mistakes. It does not prove that an app is secure. Never write "the app is secure" or "no vulnerabilities". Report: Verified (checks that ran and passed), Not verified (what only the user or a real test can show), Known risks.
- Claude cannot test against the user's real server. Every item marked "not proven" below is confirmed by the user, with the AI's help, before it counts as done.
- Only test the user's own app and the user's own backend. Never attack or scan anything else.

### The rules (every one is checked in the sections below)
1. The phone app is not trusted. Anyone can unpack an APK and read everything inside it. All permission checks happen on the server, never only in the app.
2. No secret inside the app: no secret key, no service key, no paid API key, no password. The only key allowed in the app is the public one (Supabase publishable key).
3. Never write your own login system, password hashing or token system. Use the backend's built-in auth.
4. Every table that holds user data has Row Level Security turned on, with a policy that ties each row to its owner.
5. The server never trusts what the app sends: not the user id, not the role, not the price. The owner comes from the login token (`auth.uid()`), set by a column default and checked by the policy.
6. Database queries use parameters or the client library. Never build SQL by joining strings.
7. Never run text from a user as code or as a shell command. No `eval`, no `new Function`, no shell calls with user data.
8. Show user text with `textContent`, never `innerHTML`.
9. Only `https` to the outside. No cleartext traffic.
10. Files in storage buckets are private by default, with a policy per owner. A public bucket only for files that anyone may see.
11. Login errors are generic. Do not reveal whether an email exists.
12. Keep dependencies few and current. Run the dependency scan.

### Default backend: Supabase (ask the user to confirm in Phase 0)
Supabase is open source, has a free plan, and has built-in auth and Row Level Security. If the user wants another backend, keep rules 1 to 12 and adapt the details.
- The user creates the project in the Supabase dashboard (phone browser). Claude cannot do it. The Supabase CLI is not in Termux, so the database is kept as `supabase/schema.sql` in the repo and the user pastes it into the dashboard SQL editor.
- Keys: use the publishable key (`sb_publishable_...`) in the app. It is public by design, so it may be committed. The secret key (`sb_secret_...`, and the old `service_role` key) bypasses all policies. It must never be in the repo, in `www/`, or in the app. Older projects may still show a legacy `anon` key, which plays the same role as the publishable key. Check the current key names in the dashboard (Settings, API Keys) because Supabase is moving from the old names to the new ones.
- Free plan: a project is paused after about 7 days without activity and must be restored by hand. Say this in Phase 0. Check the current limits on supabase.com/pricing before promising anything.
- Use a separate Supabase project for tests (the free plan allows two). Never run the access test on the project that holds real users.
- Dashboard settings to check (Authentication): email confirmation on, a minimum password length of at least 8, rate limits left on. Settings change over time, so ask the user to open the page and read it out.

### Where secrets go
| Secret | Where it lives | Never |
|---|---|---|
| Supabase publishable key, project URL | `www/config.js` in the repo (public) | |
| Supabase secret key | nowhere in this project. Only if a server function needs it: Supabase function secrets (dashboard) | repo, `www/`, app, chat |
| Paid API key (for example OpenRouter) | Option A: the user types their own key in the app, stored on the phone only. Option B: a Supabase Edge Function holds it, the app calls the function with the user's login token, and the function checks the user and limits the use | in `www/`, in the repo, in `PROGRESS.md` |
| Signing key password | GitHub secrets (see Signing key in apk.md) | repo |
If a secret was ever pushed to the repo, treat it as leaked: revoke and replace it. Deleting the line from the file is not enough because git history keeps it.

## Login screens (Capacitor, uses `ui.md` classes)
Build these after the skeleton works (Phase 3). One screen per job: sign in, sign up. Add "forgot password" only when the user asks: the reset email needs a redirect URL registered in the dashboard and must be tested on the phone (not proven).

Add the library by hand: in `package.json` add `"@supabase/supabase-js": "<exact version>"`. Termux has no Node or npm, so look the version up with `curl -s https://registry.npmjs.org/@supabase/supabase-js/latest` (field `version`) or `gh release list -R supabase/supabase-js -L 5` (drop the leading `v`). Take a stable version at least 14 days old to `dependencies`. In `build.yml`, right after the "Install dependencies" step and before the "Sync web files" step (`npx cap sync android`), copy the browser file into the app:
```yaml
      - name: Add Supabase client file
        run: |
          mkdir -p www/vendor
          test -f node_modules/@supabase/supabase-js/dist/umd/supabase.js
          cp node_modules/@supabase/supabase-js/dist/umd/supabase.js www/vendor/supabase.js
```
If the `test` line fails, run `ls node_modules/@supabase/supabase-js/dist` and use the real path. Do not load the library from the internet (the app must work offline and a remote script is a risk). Add `www/vendor/` to the pattern checks' skip list (the script below already skips it).

`www/config.js`
```js
// Public values. The publishable key is safe to commit. NEVER put a secret key here.
const SUPABASE_URL = "https://YOUR-PROJECT.supabase.co";
const SUPABASE_PUBLISHABLE_KEY = "sb_publishable_REPLACE_ME";
```

In `www/index.html` load, in this order, before `app.js`: `vendor/supabase.js`, `config.js`, `auth.js`.

Markup of the sign-in screen (labels only, icons later):
```html
<header class="appbar"><h1>Sign in</h1></header>
<main class="screen" id="auth-screen">
  <div class="field">
    <label for="email">Email</label>
    <input id="email" type="email" inputmode="email" autocomplete="username">
  </div>
  <div class="field">
    <label for="password">Password</label>
    <input id="password" type="password" autocomplete="current-password">
    <p class="error" id="auth-err" hidden></p>
  </div>
  <button type="button" class="block" id="sign-in">Sign in</button>
  <button type="button" class="block ghost" id="to-sign-up">Create an account</button>
  <p class="muted small" id="auth-status" hidden>Loading</p>
</main>
```
The sign-up screen is the same with a second password field ("Repeat password"), `autocomplete="new-password"`, the button "Create account", and a link back. Check on the screen: both passwords equal and at least 8 characters, with a short error line under the field. This is for comfort only, the server enforces the real rule.

`www/auth.js`
```js
const sb = window.supabase.createClient(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY);

function showError(text) {
  const el = document.getElementById("auth-err");
  el.textContent = text;          // textContent, never innerHTML
  el.hidden = !text;
}

function setBusy(busy) {
  document.getElementById("auth-status").hidden = !busy;
  document.getElementById("sign-in").disabled = busy;   // stops double taps
}

async function signIn() {
  showError("");
  const email = document.getElementById("email").value.trim();
  const password = document.getElementById("password").value;
  if (!email || !password) { showError("Type your email and password."); return; }
  setBusy(true);
  const { error } = await sb.auth.signInWithPassword({ email, password });
  setBusy(false);
  if (error) { showError("Email or password is wrong."); return; }   // same text for every failure
  showApp();
}

async function signUp(email, password) {
  const { error } = await sb.auth.signUp({ email, password });
  if (error) { showError("Could not create the account. Check the email and use at least 8 characters."); return false; }
  return true;   // tell the user to check the email if confirmation is on
}

async function signOut() {
  await sb.auth.signOut();
  showAuth();
}

document.getElementById("sign-in").addEventListener("click", signIn);

sb.auth.getSession().then(function (r) { r.data.session ? showApp() : showAuth(); });
sb.auth.onAuthStateChange(function (_event, session) { session ? showApp() : showAuth(); });
```
`showApp()` and `showAuth()` switch which screen is visible. The app screens never trust "I am signed in" from the screen alone: every data call goes through the library, and the server policies decide what comes back. Sign-out must clear anything cached in memory or on the screen.

The session is stored by the library in the app's private web storage. That is acceptable for most apps. If the app holds very sensitive data, tell the user the limit and ask whether to add a secure-storage plugin.

## Database template (`supabase/schema.sql`)
Every user-data table follows this shape. The owner column has a default, so the app never sends it.
```sql
create table public.notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null default auth.uid() references auth.users (id) on delete cascade,
  title text not null check (char_length(title) <= 200),
  body text not null default '' check (char_length(body) <= 20000),
  created_at timestamptz not null default now()
);

alter table public.notes enable row level security;

create policy "notes select own" on public.notes
  for select to authenticated using ((select auth.uid()) = user_id);
create policy "notes insert own" on public.notes
  for insert to authenticated with check ((select auth.uid()) = user_id);
create policy "notes update own" on public.notes
  for update to authenticated
  using ((select auth.uid()) = user_id) with check ((select auth.uid()) = user_id);
create policy "notes delete own" on public.notes
  for delete to authenticated using ((select auth.uid()) = user_id);

create index notes_user_id_idx on public.notes (user_id);
```
- Without a policy, nobody gets rows. That is the safe default, so add policies one at a time.
- Never write `using (true)` on a table with user data. That opens it to everyone.
- Shared data (for example a team) needs its own membership table and a policy that checks it. Ask the user before designing that.
- Functions marked `security definer` skip the policies. Avoid them. If one is needed, add `set search_path = ''` and check `auth.uid()` inside, and tell the user it needs a manual review.

Storage buckets (files): create the bucket as private. Policies go on `storage.objects`, tied to the owner, for example files stored under a folder named by the user id:
```sql
create policy "files select own" on storage.objects
  for select to authenticated
  using (bucket_id = 'files' and (storage.foldername(name))[1] = (select auth.uid())::text);
create policy "files insert own" on storage.objects
  for insert to authenticated
  with check (bucket_id = 'files' and (storage.foldername(name))[1] = (select auth.uid())::text);
```
Limit file size and allowed types in the bucket settings. Not proven until the access test below also covers a file.

Database calls from the app use the library (`sb.from("notes").select()`, `.insert({ title })`, `.update(...).eq("id", id)`), which sends values as parameters. Do not call the database with SQL text built from user input. If an Edge Function needs SQL, use parameters (`$1`) or the client library, never joined strings.

## Content security and app settings (Capacitor)
Add to the `<head>` of `www/index.html` (change the project address):
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; connect-src 'self' https://YOUR-PROJECT.supabase.co">
```
- `'unsafe-inline'` in `script-src` is only for the development error box. At release (Phase 5) remove the error script from `index.html` and remove `'unsafe-inline'` from `script-src`, then test the app on the phone.
- If the app also calls another service, add that exact address to `connect-src`. Never use `*`.
- If the app uses Supabase Realtime (live updates), also add `wss://YOUR-PROJECT.supabase.co` to `connect-src`: realtime runs over a websocket, which `https://` does not cover. Not proven until a realtime feature is tested on the phone.
- Set `android:allowBackup="false"` on `<application>` in `android/app/src/main/AndroidManifest.xml` (Kotlin: `app/src/main/AndroidManifest.xml`), so the saved session is not copied by backup. Read the file first.
- Keep `capacitor.config.json` free of `cleartext` and `allowMixedContent: true`. The pattern check below flags them.

## Threat list (what each risk is, what stops it, how it is checked)
| Risk | What stops it | Automatic check | Manual check |
|---|---|---|---|
| SQL injection | library calls with parameters, no joined SQL (rule 6) | pattern check finds SQL joined with `+` or `${}` in server code | none needed if there is no custom SQL. With custom SQL: read each query |
| Broken access control (reading or changing another user's data, also another user's files) | Row Level Security with owner policies, storage policies (rules 4, 5, 10) | schema check: every table has RLS and a policy, no `using (true)`. Access test (below) | two-account test by hand |
| Remote code execution | no `eval`, no shell calls with user data, no unsafe loading of code (rule 7), few dependencies | pattern check for `eval`, `new Function`, `child_process`, `exec`, `Deno.Command`, `subprocess` | read any server function that takes input |
| Cross-site scripting | `textContent`, Content Security Policy (rules 8) | pattern check for `innerHTML`, `insertAdjacentHTML`, `document.write` | type `<b>x</b>` into every text field and look at how it shows: it must appear as text |
| Leaked secrets | rule 2, secret scan | gitleaks, pattern check for `sb_secret_` and `service_role` in client files | ask the user to confirm no key was ever pasted into the repo or the chat |
| Weak login (guessing, user listing) | built-in auth, rate limits on, generic messages (rules 3, 11) | none | try a wrong password and a missing email: the messages must be identical |
| Client sends data it should not control (user id, role, price) | owner column default plus policy `with check` (rule 5) | access test case "claim another owner" | the same, by hand |
| Vulnerable libraries | few dependencies, scanner (rule 12) | osv-scanner on the lock file | none |
| Cleartext traffic | https only (rule 9) | pattern check for `http://` and `cleartext` | none |

## Automatic checks (copy exactly, then fill the placeholders)
Same strictness policy as the built-in checker in apk.md: in Phase 1 to 3 the steps run with `continue-on-error: true` so a warning never blocks the first builds. In Phase 4 remove that line from every step so they block.

Before using any tool, read its license (`gh api repos/<owner>/<name>/license`, see "Built-in checker" in apk.md). Expected: gitleaks MIT, osv-scanner Apache-2.0, both allowed. Confirm it yourself, do not trust this list. Do not add tools with GPL, AGPL or LGPL licenses without asking the user. Look up the newest release tag that is at least 14 days old with `gh release list --repo <owner>/<name> --limit 10`, and pin that exact tag in the workflow.

`.github/workflows/security.yml`
```yaml
name: security
on:
  push:
  workflow_dispatch:
permissions:
  contents: read
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - name: Own pattern checks
        continue-on-error: true
        run: bash scripts/check-security.sh
      - name: Secret scan (gitleaks)
        continue-on-error: true
        env:
          GH_TOKEN: ${{ github.token }}
          GITLEAKS_TAG: "REPLACE_WITH_PINNED_TAG"
        run: |
          mkdir -p /tmp/gl
          gh release download "$GITLEAKS_TAG" --repo gitleaks/gitleaks --pattern '*linux_x64.tar.gz' --dir /tmp/gl
          tar -xzf /tmp/gl/*.tar.gz -C /tmp/gl gitleaks
          /tmp/gl/gitleaks detect --source . --redact --no-banner
      - name: Dependency scan (osv-scanner)
        continue-on-error: true
        env:
          GH_TOKEN: ${{ github.token }}
          OSV_TAG: "REPLACE_WITH_PINNED_TAG"
        run: |
          mkdir -p /tmp/osv
          gh release download "$OSV_TAG" --repo google/osv-scanner --pattern 'osv-scanner_linux_amd64' --dir /tmp/osv
          chmod +x /tmp/osv/osv-scanner_linux_amd64
          /tmp/osv/osv-scanner_linux_amd64 scan source -r .
```
If a command is rejected (flags change between versions), run the tool with `--help`, fix the one line, and note it in `LESSONS.md`. The file names of the release assets can also differ: check them with `gh release view <tag> --repo <owner>/<name> --json assets -q '.assets[].name'`. Not proven until the first green run.

`scripts/check-security.sh` (no dependencies; add `# security-ok: reason` at the end of a line to accept one reviewed finding)
```bash
#!/usr/bin/env bash
# Pattern checks for the rules in security.md. Exit 1 when something is found.
fail=0
CLIENT="www lib app src"
SERVER="supabase/functions server api functions"

scan() {  # label, regex, dirs...
  local label="$1" re="$2"; shift 2
  local out
  out=$(grep -rnEi --include='*.js' --include='*.mjs' --include='*.ts' --include='*.html' \
        --include='*.json' --include='*.dart' --include='*.kt' --include='*.py' -e "$re" "$@" 2>/dev/null \
        | grep -v -e 'www/vendor/' -e 'node_modules/' -e 'security-ok' -e 'sb_publishable_' -e 'http://www.w3.org/' || true)
  if [ -n "$out" ]; then echo "== $label"; echo "$out"; fail=1; fi
}

scan "code from strings (rule 7)" 'eval\(|new Function\(|document\.write\(|setTimeout\(["`]' $CLIENT $SERVER
scan "html from text (rule 8)" '\.innerHTML *\+?=|\.outerHTML *=|insertAdjacentHTML|dangerouslySetInnerHTML' $CLIENT
scan "server key in client (rule 2)" 'service_role|sb_secret_' $CLIENT
scan "hard-coded secret (rule 2)" '(api[_-]?key|secret|passw(or)?d|token)["'"'"']? *[:=] *["'"'"'][A-Za-z0-9_./+-]{16,}["'"'"']' $CLIENT $SERVER
scan "cleartext (rule 9)" 'cleartext"? *: *true|allowMixedContent"? *: *true|"http://[^"]*"|'"'"'http://[^'"'"']*'"'" $CLIENT capacitor.config.json
scan "shell or process calls (rule 7)" 'child_process|execSync|spawnSync|\bexec\(|Deno\.run|Deno\.Command|subprocess|os\.system' $SERVER
scan "SQL joined from strings (rule 6)" '(select|insert into|update|delete from)[^;`"'"'"']*(["'"'"']+ *\+|\$\{)' $SERVER

# Tables must have RLS and at least one policy; no open policies.
SQL=$(find . -name '*.sql' -not -path './node_modules/*' 2>/dev/null)
if [ -n "$SQL" ]; then
  ALL=$(cat $SQL | tr '\n' ' ')
  for t in $(echo "$ALL" | grep -oEi 'create table( if not exists)? +(public\.)?"?[a-z_0-9]+' | sed -E 's/.* +(public\.)?"?([a-z_0-9]+)$/\2/'); do
    echo "$ALL" | grep -qEi "alter table +(public\.)?\"?$t\"? +enable row level security" || { echo "== table $t has no RLS (rule 4)"; fail=1; }
    echo "$ALL" | grep -qEi "create policy[^;]* on +(public\.)?\"?$t\"?" || { echo "== table $t has no policy (rule 4)"; fail=1; }
  done
  echo "$ALL" | grep -qEi '(using|with check) *\( *true *\)' && { echo "== open policy using (true) (rule 4)"; fail=1; }
  echo "$ALL" | grep -qEi 'security definer' && { echo "== security definer function: needs manual review"; fail=1; }
fi

[ "$fail" = 0 ] && echo "check-security: nothing found"
exit $fail
```
These checks are rough pattern matches. A clean result means only "none of these patterns appear". It does not mean the app is safe. Not proven until the first run: if the script reports something wrong, fix the script and note it in `LESSONS.md`.

## Access test (the only automatic way to catch broken access control)
It needs a separate test Supabase project and two test accounts (A and B, created by the user with sign-up in the test project). Store these as GitHub secrets with `gh secret set`: `TEST_SUPABASE_URL`, `TEST_SUPABASE_PUBLISHABLE_KEY`, `TEST_A_EMAIL`, `TEST_A_PASSWORD`, `TEST_B_EMAIL`, `TEST_B_PASSWORD`. Never use real user accounts. The test assumes a table with an `id` primary key and a `user_id` owner column (change `TABLE` and `ROW` for the app).

`tests/access.mjs`
```js
// Run: node tests/access.mjs   (Node 18 or newer)
const URL_ = process.env.SUPABASE_URL;
const KEY = process.env.SUPABASE_PUBLISHABLE_KEY;
const TABLE = process.env.TABLE || "notes";
const ROW = JSON.parse(process.env.ROW_JSON || '{"title":"access-test"}');
const A = { email: process.env.TEST_A_EMAIL, password: process.env.TEST_A_PASSWORD };
const B = { email: process.env.TEST_B_EMAIL, password: process.env.TEST_B_PASSWORD };

let bad = 0;
function check(name, ok) { console.log((ok ? "PASS " : "FAIL ") + name); if (!ok) bad += 1; }

async function login(u) {
  const r = await fetch(URL_ + "/auth/v1/token?grant_type=password", {
    method: "POST",
    headers: { apikey: KEY, "Content-Type": "application/json" },
    body: JSON.stringify(u),
  });
  const j = await r.json();
  if (!j.access_token) throw new Error("login failed for " + u.email);
  return { token: j.access_token, id: j.user.id };
}

function call(path, method, token, body) {
  const headers = { apikey: KEY, "Content-Type": "application/json", Prefer: "return=representation" };
  if (token) headers.Authorization = "Bearer " + token;
  return fetch(URL_ + "/rest/v1/" + path, { method, headers, body: body ? JSON.stringify(body) : undefined });
}

async function rows(resp) {
  if (!resp.ok) return [];
  const j = await resp.json();
  return Array.isArray(j) ? j : [];
}

const a = await login(A);
const b = await login(B);

const made = await call(TABLE, "POST", a.token, { ...ROW });
const mine = await rows(made);
check("A can create a row", made.ok && mine.length === 1);
const id = mine[0] && mine[0].id;

check("A can read the row", (await rows(await call(TABLE + "?id=eq." + id, "GET", a.token))).length === 1);
check("B cannot read A's row", (await rows(await call(TABLE + "?id=eq." + id, "GET", b.token))).length === 0);
check("B cannot change A's row", (await rows(await call(TABLE + "?id=eq." + id, "PATCH", b.token, { ...ROW, user_id: b.id }))).length === 0);
check("B cannot delete A's row", (await rows(await call(TABLE + "?id=eq." + id, "DELETE", b.token))).length === 0);
check("B cannot create a row owned by A", !(await call(TABLE, "POST", b.token, { ...ROW, user_id: a.id })).ok);
check("signed-out request sees nothing", (await rows(await call(TABLE + "?id=eq." + id, "GET", null))).length === 0);
check("A still has the row after B's attempts", (await rows(await call(TABLE + "?id=eq." + id, "GET", a.token))).length === 1);

await call(TABLE + "?id=eq." + id, "DELETE", a.token);   // clean up
console.log(bad === 0 ? "ACCESS TEST PASSED" : "ACCESS TEST FAILED: " + bad);
process.exit(bad === 0 ? 0 : 1);
```
Workflow job (add to `security.yml`, `needs` nothing; skip it until the test project and secrets exist):
```yaml
  access:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v7
        with:
          node-version: 22
      - name: Access test
        env:
          SUPABASE_URL: ${{ secrets.TEST_SUPABASE_URL }}
          SUPABASE_PUBLISHABLE_KEY: ${{ secrets.TEST_SUPABASE_PUBLISHABLE_KEY }}
          TEST_A_EMAIL: ${{ secrets.TEST_A_EMAIL }}
          TEST_A_PASSWORD: ${{ secrets.TEST_A_PASSWORD }}
          TEST_B_EMAIL: ${{ secrets.TEST_B_EMAIL }}
          TEST_B_PASSWORD: ${{ secrets.TEST_B_PASSWORD }}
        run: node tests/access.mjs
```
The script logic was run against a mock server: it passes when the policies work and fails when they are missing. Real responses from Supabase are not proven until the first real run. The test project must have the same `schema.sql` applied as the real one. Add one test group per table that holds user data. The test is not proven until the user has run it once with a real failure: temporarily drop one policy in the test project and confirm the test turns red. A test that cannot fail proves nothing.

## Manual checks (ask the user to do these on the phone, one at a time)
1. Wrong password: the message is the same as for an email that does not exist.
2. Sign in as A, create an item. Sign out. Sign in as B. A's item is not visible anywhere.
3. Type `<b>x</b>` and `'; drop table notes; --` into every text field and save. They show up as plain text and nothing breaks.
4. Sign out, press back: no data on screen.
5. Turn on airplane mode: the app shows a short message and does not crash or show old private data after sign-out.
6. Search the repo history for keys: `git log -p | grep -E "sb_secret_|service_role"` returns nothing.

## Where this fits in the phases
- Phase 0: ask "does the app have accounts, a server, a database, uploads, or a paid API key?". If yes, record in `PROGRESS.md` a Security section: backend, auth method, each table with its owner column, each bucket, where each secret lives. Done when: the user approves this section.
- Phase 1: add `.github/workflows/security.yml` and `scripts/check-security.sh` (non-blocking). Create `supabase/schema.sql` empty.
- Phase 3, for the login feature, write "Done when" lines like:
  - "Done when: a wrong password shows 'Email or password is wrong.' and nothing else."
  - "Done when: sign up, confirm the email, sign in, close the app from recent apps, open it again, and still be signed in."
  - "Done when: sign out, press back, and no private data is on screen."
  For every table: "Done when: the access test passes and the user ran manual checks 2 and 3."
- Phase 4: remove `continue-on-error` from the security steps, fix every finding or mark it `security-ok` with a reason, set `android:allowBackup="false"`, and run all manual checks.
- Phase 5: remove the error box script and `'unsafe-inline'` from the Content Security Policy, confirm no secret key anywhere (manual check 6), confirm the release build still signs in. Tell the user that the shared APK is a debug build (debuggable): someone with the phone and a cable can read the app's private data, including the saved session. That is fine for personal use. Before giving the app to other people, switch to a release build with a separate release key (apk.md Phase 5) and list this under Known risks.
- Report format after each security step: Verified (which checks ran green, with the run id), Not verified (every manual check still open), Known risks (anything accepted with `security-ok`, and why).
