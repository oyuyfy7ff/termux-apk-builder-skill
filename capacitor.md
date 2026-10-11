# capacitor.md (companion file of apk.md)

Use when: the chosen stack is Capacitor. Needed before Phase 1.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

Replace the placeholders: `demoapp` (project name, lowercase), `Demo App` (display name), `com.example.demoapp` (app id). Use the same values in every file.

## Capacitor template
File tree (the `android` folder is created by the first workflow run and committed back automatically):
```
.github/workflows/build.yml
.gitignore
PROGRESS.md
package.json
capacitor.config.json
www/index.html
www/style.css
www/app.js
```

`.gitignore`
```
node_modules/
*.keystore
*.jks
local.properties
.gradle/
build/
.idea/
```
Do not add `android/` to `.gitignore`. It must be committed.

`package.json`
```json
{
  "name": "demoapp",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "@capacitor/android": "^8.0.0",
    "@capacitor/core": "^8.0.0"
  },
  "devDependencies": {
    "@capacitor/cli": "^8.0.0"
  }
}
```
Plugins: add them by hand to `dependencies` with the same major (`"@capacitor/<plugin>": "^8.0.0"`), push, and the workflow installs them and syncs.

`capacitor.config.json`
```json
{
  "appId": "com.example.demoapp",
  "appName": "Demo App",
  "webDir": "www"
}
```

`www/index.html`
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <title>Demo App</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <main>
    <h1 id="count">0</h1>
    <button id="btn" type="button">Add</button>
    <pre id="err"></pre>
  </main>
  <script>
    window.addEventListener("error", function (e) {
      document.getElementById("err").textContent += e.message + " (" + e.filename + ":" + e.lineno + ")\n";
    });
    window.addEventListener("unhandledrejection", function (e) {
      document.getElementById("err").textContent += "Promise: " + e.reason + "\n";
    });
  </script>
  <script src="app.js"></script>
</body>
</html>
```
The small error script shows JavaScript errors on the screen, so the user can send a screenshot (no computer needed). Keep it while developing.

`www/style.css`
```css
body {
  font-family: sans-serif;
  margin: 0;
  padding: 16px;
}
main {
  text-align: center;
  margin-top: 64px;
}
h1 {
  font-size: 64px;
}
button {
  font-size: 20px;
  padding: 12px 32px;
}
#err {
  color: #b00020;
  white-space: pre-wrap;
  text-align: left;
}
```
This plain CSS is only the minimum. In Phase 1 replace it: write `www/style.css` as Tokens + Theme generator + Components from `ui.md` (the `#err` rule is already inside Components), then add these skeleton-only lines at the end. The skeleton markup works with it unchanged and the 0 to 1 test stays the same:
```css
main { text-align: center; margin-top: 64px; padding: 16px; }
h1 { font-size: 64px; }
```

`www/app.js`
```js
let count = 0;
const label = document.getElementById("count");
document.getElementById("btn").addEventListener("click", function () {
  count += 1;
  label.textContent = String(count);
});
```
"Done when" for this skeleton: the number goes from 0 to 1 after one tap.

`.github/workflows/build.yml`
```yaml
name: build
on:
  push:
  workflow_dispatch:
permissions:
  contents: write # needed to commit the android folder and to publish releases
jobs:
  apk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v7
        with:
          node-version: 22
      - uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: 21
      - name: Restore signing key
        env:
          KEYSTORE_B64: ${{ secrets.DEBUG_KEYSTORE_B64 }}
        run: |
          mkdir -p ~/.android
          echo "$KEYSTORE_B64" | base64 -d > ~/.android/debug.keystore
      - name: Install dependencies
        run: |
          if [ -f package-lock.json ]; then npm ci; else npm install; fi
      - name: Check JavaScript syntax
        continue-on-error: true
        run: |
          for f in www/*.js; do node --check "$f"; done
      - name: Add Android project if missing
        id: android
        run: |
          if [ -d android ]; then
            echo "created=false" >> "$GITHUB_OUTPUT"
          else
            npx cap add android
            echo "created=true" >> "$GITHUB_OUTPUT"
          fi
      - name: Commit generated Android project (first build only)
        if: steps.android.outputs.created == 'true' && !startsWith(github.ref, 'refs/tags/')
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add android
          git commit -m "chore: add generated Android project"
          git pull --rebase
          git push
      - name: Sync web files into the Android project
        run: npx cap sync android
      - name: Build debug APK
        working-directory: android
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug
      - name: List APK files
        run: find android -name "*.apk"
      - uses: actions/upload-artifact@v7
        with:
          name: app-debug
          path: android/app/build/outputs/apk/debug/*.apk
          if-no-files-found: error
      - name: Publish release (version tags only)
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          cp android/app/build/outputs/apk/debug/app-debug.apk "app-$TAG.apk"
          if gh release view "$TAG" >/dev/null 2>&1; then
            gh release upload "$TAG" "app-$TAG.apk" --clobber
          else
            gh release create "$TAG" "app-$TAG.apk" --generate-notes
          fi
```
Capacitor notes:
- The Android project and its Gradle wrapper come from `npx cap add android`, so the Gradle and Android Gradle plugin versions always match the Capacitor version. Do not install another Gradle version for Capacitor.
- Tag runs cannot push to the branch, so the "Commit generated Android project" step is skipped on tags. Run `build.yml` by hand once (or push to main) before the first tag.
- Edit only `www/` for normal work. The workflow copies it into the app on every build.
- Changing the app id or app name after the first build: delete the `android` folder (`git rm -r android`, commit, push) and let the workflow create it again, or edit the files listed in Phase 4.
- Local storage (`localStorage`) works inside the app and keeps data between launches.
- The template already allows internet access.
