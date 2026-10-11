# errors.md (companion file of apk.md)

Use when: a build or the app fails. Attach it when the build is red or the app crashes. Nothing else needs it.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

## Error table
Find the symptom, apply the fix, push, check the run. One fix at a time.

| Symptom | Cause | Fix |
|---|---|---|
| Push rejected: `refusing to allow an OAuth App to create or update workflow` | Login lacks the workflow permission | `gh auth refresh -s workflow`, push again |
| Commit fails: `Author identity unknown` | Git name and email not set | `git config --global user.name "<name>"` and `git config --global user.email "<email>"` |
| Push rejected: `Updates were rejected because the remote contains work` | The workflow committed the android folder, or an earlier push exists | `git pull --rebase`, then push again |
| Workflow does not start (nothing in `gh run list`) | Workflow file not in `.github/workflows/`, wrong name, or invalid YAML | Check the path and `.yml` ending. Run `gh run list` again after 1 minute. If a run named after the file path failed at once, open it with `gh run view <id>`: it shows the YAML error |
| Run fails in 0 seconds, no jobs | YAML syntax error (tabs, wrong indent, unquoted colon) | Fix the line named in `gh run view <id>`. Use spaces only |
| `Unable to resolve action` | The action tag does not exist | `gh release list -R <owner>/<repo> -L 3`, use the newest major |
| `Error: Input required and not supplied` or unknown input warning | Wrong input name in `with:` | Compare with the template |
| Secret is empty, `base64: invalid input` in "Restore signing key" | Secret `DEBUG_KEYSTORE_B64` missing or has the wrong name | `gh secret list`. If missing, recreate it (Signing key section) |
| `permission denied: ./gradlew` | Script not executable | Add `chmod +x gradlew` before running it (already in the Capacitor template) |
| `The Capacitor CLI requires NodeJS` | Node too old | Use `node-version: 22` |
| `Could not find the web assets directory: ./www` | `webDir` does not match the folder, or `www/index.html` missing | Create `www/index.html` or fix `webDir` |
| `npm error ... ENOENT package.json` | No `package.json` in the repo root | Create it from the template |
| `npm ci` fails: `package-lock.json` out of sync | Lock file does not match `package.json` | Delete `package-lock.json` from the repo, push (the workflow then runs `npm install`) |
| `Unsupported class file major version` or `Gradle requires JVM 17 or later` | Wrong Java version | Use `java-version: 21` as in the templates |
| `Minimum supported Gradle version is X. Current version is Y` | Android Gradle plugin and Gradle do not match | Use the AGP table in Known-good versions. For Kotlin change the one `GRADLE_VERSION` line in `build.yml` (the download address and the path both use it), for Capacitor and Flutter do not override the wrapper |
| `Plugin [id: 'com.android.application'] was not found` | `pluginManagement` repositories missing | Use `settings.gradle.kts` from the template |
| `Namespace not specified` | AGP 8 needs `namespace` | Add `namespace = "<app id>"` in `android { }` |
| `android:exported needs to be explicitly specified` | Manifest merge error on targetSdk 31 or higher | Add `android:exported="true"` to the launcher activity |
| `resource style/... not found` or `AAPT: error: resource ... not found` | Theme or resource name does not exist | Use a framework theme (as in the template) or create the resource file |
| `Unresolved reference` (Kotlin) | Missing import or wrong name | Read the file and line in the log, fix that line only |
| `SDK location not found` | `ANDROID_HOME` not set for Gradle | Add a step: `echo "sdk.dir=$ANDROID_HOME" > local.properties` (in `android/` for Capacitor) |
| `Failed to find target with hash string 'android-36'` or license not accepted | Platform missing on the runner | Add a step: `yes \| "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager" "platforms;android-36"` |
| Flutter: `flutter: command not found` | Setup action missing or after the first use of `flutter` | Put `subosito/flutter-action@v2` before every `flutter` command |
| Flutter: `The current Dart SDK version is X ... requires SDK version` | `environment: sdk:` too high for the Flutter version | Lower the constraint in `pubspec.yaml` or use the stable channel |
| Flutter: `Target of URI doesn't exist: 'package:...'` | Import name differs from the `name:` in `pubspec.yaml` | Make them identical |
| Flutter: `flutter analyze` fails on missing `flutter_lints` | A generated `analysis_options.yaml` includes a package that is not installed | Keep the `analysis_options.yaml` from the template in the repo |
| `No files were found with the provided path` (upload step) | APK path differs | Read the output of the step "List APK files", fix the `path:` |
| `gh run download`: `no valid artifacts found` | Run not finished, or artifact name differs | `gh run watch` first. The name must be `app-debug` |
| `gh release create` fails with 403 | No write permission | Keep `permissions: contents: write` at the top of the workflow. If it still fails: repository Settings > Actions > General > Workflow permissions > Read and write |
| `gh release create` fails: `already exists` | An old workflow created the release after `ios.yml` had already made it | Use the release step of the current templates (it uploads when the release exists), then `gh run rerun <id> --failed` |
| `git push` inside the workflow fails (first build) | Same permission problem, or the branch moved | Same fix as above, then re-run with `gh run rerun <id>` |
| Phone: `App not installed` | New APK signed with another key, or lower `versionCode`, or not enough storage | Check the secret exists and the restore step ran. Uninstall the old app once. Raise `versionCode` |
| Phone: `There was a problem parsing the package` | Download incomplete, or `minSdk` higher than the phone | Download again. Compare `getprop ro.build.version.sdk` with `minSdk` |
| `termux-open` does nothing | No handler for the APK | `cp <file>.apk ~/storage/downloads/` and open it from the file app |
| App opens and closes at once | Runtime crash | Use the crash screen and screenshot from "Debugging without a computer" |
| iOS: `requires a development team` or `Code Signing Error` | Signing not switched off in `xcodebuild` | Keep `CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""` on the command |
| iOS: `does not exist` for the project or workspace, or `scheme App` not found | Wrong path or scheme | Read the output of "List iOS project files" and fix the path or scheme name |
| iOS: `requires Xcode` or `SDK ... cannot be located` or deployment target error | Xcode on the runner is older than the project needs | Read the output of "Select newest stable Xcode". If the newest listed Xcode is still too old, switch the runner (`runs-on`) to a newer macOS image after checking the runner list, then run again |
| iOS: `npx cap add ios` fails, package not found | `@capacitor/ios` missing in `package.json` | Add `"@capacitor/ios": "^8.0.0"`, delete `package-lock.json` if `npm ci` complains, push |
| iOS: `cp` or `find` fails in "Make unsigned IPA" | Build output folder differs | Read the `find` output of the step, fix the path |
| iOS: upload to the release fails | The Android release step failed, or the tag run is not finished | Fix the Android run first, then `gh run rerun <id> --failed` for `ios.yml` |
| iPhone: tool refuses the IPA, or app opens and closes at once | Sideloading tool problem, or a debug build of Flutter (cannot run from the home screen) | Check the tool's own message. The Flutter template builds release. Ask which tool and which message, do not guess |
| iPhone: app stops opening after about 7 days | Free Apple ID signing expires | Sign and install again with the sideloading tool |
| iOS run used many minutes | macOS minutes cost more than Linux minutes on private repos | Run `ios.yml` only when needed (by hand or on tags) |
| Script prints `cd: ... No such file or directory` as its first line | The script guessed the repo folder | The rest may still work when the user runs it inside the repo, but do not rely on that. Use the first lines from "Script delivery" |
| The AI says it read a file but cannot show its exact text | A web fetch tool gives a summary | Clone the repo in the sandbox, or ask the user to paste the file. Never edit from a summary |
| New APK installed but the version line did not change | The version was typed wrong in the reply, or the install did not replace the old app | Compare with the `APP_VERSION` line in the code, then see "App not installed" in this table |
| Build used all free minutes | Too many pushes | Batch changes, avoid pointless rebuilds, check usage in the GitHub billing page |

