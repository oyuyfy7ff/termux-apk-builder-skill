# ios.md (companion file of apk.md)

Use when: the user wants an iOS build (Capacitor or Flutter only). Needed from Phase 0 if iOS is wanted.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

## iOS support (optional, Capacitor and Flutter only)
What is possible, say this in Phase 0:
- Kotlin (native) is Android only. For iOS use Capacitor or Flutter.
- iOS apps build only on a macOS runner on GitHub Actions. Termux cannot build or install them.
- This skill has no Apple Developer account. The build makes an UNSIGNED `.ipa`. It cannot be installed directly. The user signs and installs it with a sideloading tool of their own. Ask which tool they use. Do not guess its steps. An app signed with a free Apple ID normally stops opening after about 7 days and must be signed again.
- Claude cannot test iOS. Every iOS "Done when" line is confirmed by the user on the iPhone.
- On private repos, macOS minutes count about 10 times as much as Linux minutes. So iOS has its own workflow `ios.yml` that runs only by hand (`gh workflow run ios.yml`) and on `v*` tags, never on every push. Public repos are free.
- Order: Android build green first (unless the user wants iOS only), then iOS. `build.yml` stays unchanged.

Steps:
1. Capacitor only: add `"@capacitor/ios": "^8.0.0"` to `dependencies` in `package.json`. Flutter needs no change.
2. Create `.github/workflows/ios.yml` from the template of the stack (quoted heredoc, spaces only).
3. `git pull --rebase`, commit (`ci: add iOS workflow`), push, then `gh workflow run ios.yml`.
4. The first run creates the `ios` folder and commits it back. Then `git pull --rebase` before the next push. Do not add `ios/` to `.gitignore`.
5. Watch and download:
```
git pull --rebase
gh workflow run ios.yml
SHA=$(git rev-parse HEAD)
RUN=""
for i in 1 2 3 4 5 6 7 8 9 10 11 12; do
  RUN=$(gh run list --workflow ios.yml --commit "$SHA" --limit 1 --json databaseId --jq '.[0].databaseId')
  [ -n "$RUN" ] && break
  sleep 5
done
[ -n "$RUN" ] || echo "No iOS run found for this commit yet. Check the Actions tab."
[ -n "$RUN" ] && gh run watch "$RUN" --exit-status && rm -rf ~/storage/downloads/app-ios && gh run download "$RUN" -n app-unsigned-ipa -D ~/storage/downloads/app-ios
```
6. Getting the file to the iPhone: the simplest way is a version tag (the release then holds the IPA). The user opens the release page in Safari on the iPhone, signed in to GitHub (needed for a private repo), downloads the `.ipa`, and opens it with their sideloading tool.
- App id: the iOS bundle id is the same as the app id. Keep it identical everywhere.
- App name: Capacitor `ios/App/App/Info.plist`, Flutter `ios/Runner/Info.plist` (`CFBundleDisplayName`). Read the file first.
- Version: Flutter uses `version:` in `pubspec.yaml` for both platforms. Capacitor: `MARKETING_VERSION` and `CURRENT_PROJECT_VERSION` in `ios/App/App.xcodeproj/project.pbxproj`. Read the file first.
- The Android release step and the iOS release step each create the release when it does not exist yet and upload into it when it does, so the order of the two runs does not matter.
- Tag runs cannot push to the branch, so the "commit generated iOS project" step is skipped on tags. Run `ios.yml` by hand once before the first tag.

Capacitor `.github/workflows/ios.yml`
```yaml
name: ios
on:
  workflow_dispatch:
  push:
    tags:
      - 'v*'
permissions:
  contents: write # needed to commit the ios folder and to add the IPA to a release
jobs:
  ipa:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-node@v7
        with:
          node-version: 22
      - name: Install dependencies
        run: |
          if [ -f package-lock.json ]; then npm ci; else npm install; fi
      - name: Select newest stable Xcode
        run: |
          ls /Applications | grep -i xcode
          NEWEST=$(ls -d /Applications/Xcode_*.app 2>/dev/null | grep -vi beta | sort -V | tail -n 1)
          if [ -n "$NEWEST" ]; then sudo xcode-select -s "$NEWEST"; fi
          xcodebuild -version
      - name: Add iOS project if missing
        id: ios
        run: |
          if [ -d ios ]; then
            echo "created=false" >> "$GITHUB_OUTPUT"
          else
            npx cap add ios
            echo "created=true" >> "$GITHUB_OUTPUT"
          fi
      - name: Commit generated iOS project (first build only)
        if: steps.ios.outputs.created == 'true' && !startsWith(github.ref, 'refs/tags/')
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add ios
          git commit -m "chore: add generated iOS project"
          git pull --rebase
          git push
      - name: Sync web files into the iOS project
        run: npx cap sync ios
      - name: List iOS project files
        run: find ios -maxdepth 3 \( -name "*.xcodeproj" -o -name "*.xcworkspace" \)
      - name: Build unsigned app
        working-directory: ios/App
        run: |
          if [ -d App.xcworkspace ]; then
            xcodebuild -workspace App.xcworkspace -scheme App -configuration Release -sdk iphoneos -derivedDataPath build CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
          else
            xcodebuild -project App.xcodeproj -scheme App -configuration Release -sdk iphoneos -derivedDataPath build CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
          fi
      - name: Make unsigned IPA
        run: |
          find ios/App/build/Build/Products -maxdepth 2 -name "*.app"
          APP=$(find ios/App/build/Build/Products/Release-iphoneos -maxdepth 1 -name "*.app" | head -n 1)
          mkdir Payload
          cp -R "$APP" Payload/
          zip -qr app-unsigned.ipa Payload
          ls -l app-unsigned.ipa
      - uses: actions/upload-artifact@v7
        with:
          name: app-unsigned-ipa
          path: app-unsigned.ipa
          if-no-files-found: error
      - name: Add IPA to the release (version tags only)
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          for i in 1 2 3 4 5 6 7 8 9 10; do
            gh release view "$TAG" >/dev/null 2>&1 && break
            sleep 30
          done
          gh release view "$TAG" >/dev/null 2>&1 || gh release create "$TAG" --generate-notes
          cp app-unsigned.ipa "app-$TAG-unsigned.ipa"
          gh release upload "$TAG" "app-$TAG-unsigned.ipa" --clobber
```

Flutter `.github/workflows/ios.yml`
```yaml
name: ios
on:
  workflow_dispatch:
  push:
    tags:
      - 'v*'
permissions:
  contents: write # needed to commit the ios folder and to add the IPA to a release
jobs:
  ipa:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v6
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true
      - name: Select newest stable Xcode
        run: |
          ls /Applications | grep -i xcode
          NEWEST=$(ls -d /Applications/Xcode_*.app 2>/dev/null | grep -vi beta | sort -V | tail -n 1)
          if [ -n "$NEWEST" ]; then sudo xcode-select -s "$NEWEST"; fi
          xcodebuild -version
      - name: Add iOS project if missing
        id: ios
        run: |
          if [ -d ios ]; then
            echo "created=false" >> "$GITHUB_OUTPUT"
          else
            flutter create --platforms=ios --org com.example --project-name demoapp .
            echo "created=true" >> "$GITHUB_OUTPUT"
          fi
      - name: Commit generated iOS project (first build only)
        if: steps.ios.outputs.created == 'true' && !startsWith(github.ref, 'refs/tags/')
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add ios
          git commit -m "chore: add generated iOS project"
          git pull --rebase
          git push
      - run: flutter pub get
      - name: Build unsigned app (release, a debug app cannot start from the home screen)
        run: flutter build ios --release --no-codesign
      - name: Make unsigned IPA
        run: |
          ls build/ios/iphoneos
          mkdir Payload
          cp -R build/ios/iphoneos/Runner.app Payload/
          zip -qr app-unsigned.ipa Payload
          ls -l app-unsigned.ipa
      - uses: actions/upload-artifact@v7
        with:
          name: app-unsigned-ipa
          path: app-unsigned.ipa
          if-no-files-found: error
      - name: Add IPA to the release (version tags only)
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          for i in 1 2 3 4 5 6 7 8 9 10; do
            gh release view "$TAG" >/dev/null 2>&1 && break
            sleep 30
          done
          gh release view "$TAG" >/dev/null 2>&1 || gh release create "$TAG" --generate-notes
          cp app-unsigned.ipa "app-$TAG-unsigned.ipa"
          gh release upload "$TAG" "app-$TAG-unsigned.ipa" --clobber
```
iOS notes:
- Flutter: in the `flutter create` line of `ios.yml`, replace `com.example` (after `--org`) and `demoapp` (after `--project-name`) with the chosen app id parts, before the first run. The result `<org>.<project-name>` must equal the app id, and `--project-name` must equal `name:` in `pubspec.yaml`.
- Flutter: if the Android build pinned a Flutter version, use the same `flutter-version` here.
- Plugins that need native iOS setup (camera, location, notifications) need entries in `Info.plist` (permission texts). Look up the exact key for the plugin, never guess it.
- iOS only (no Android): skip `build.yml`, still create the project files from the stack template, and use `ios.yml` as the only workflow.

## iOS checklists
Before every iOS run:
1. The Android build for the same commit is green.
2. `ios/` folder is committed (after the first iOS run), and not in `.gitignore`.
3. The `xcodebuild` or `flutter build ios` line still has the no-signing options.
   Flutter: the `flutter create` line uses the real app id, not `com.example.demoapp`.
4. `PROGRESS.md` says which sideloading tool the user uses.
5. Run `gh workflow run ios.yml` once, not after every small change (macOS minutes).

After a green iOS run:
1. The artifact `app-unsigned-ipa` exists.
2. Tell the user: Verified (build green, IPA made), Not verified (everything on the iPhone), Known issues (7-day expiry with a free Apple ID).
