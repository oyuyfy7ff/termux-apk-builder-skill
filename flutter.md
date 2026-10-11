# flutter.md (companion file of apk.md)

Use when: the chosen stack is Flutter. Needed before Phase 1.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

## Flutter look (design notes)
- Keep Material 3: `useMaterial3: true` with `colorSchemeSeed`. Use the light "Light main" color of the chosen hue as the seed (any hex works; hue guide seeds: red 0xFFBE222A, orange 0xFF9F5100, yellow 0xFF7C6700, green 0xFF007F38, teal 0xFF007879, blue 0xFF006BB9, violet 0xFF634ECF, pink 0xFFB22578).
```dart
theme: ThemeData(colorSchemeSeed: const Color(0xFF315DD4), useMaterial3: true),
darkTheme: ThemeData(
  colorSchemeSeed: const Color(0xFF315DD4),
  useMaterial3: true,
  brightness: Brightness.dark,
),
themeMode: ThemeMode.system,
```
For a dark-only look use `themeMode: ThemeMode.dark`.
- Use the stock widgets, not custom ones: `FilledButton` (primary), `FilledButton.tonal` (soft), `OutlinedButton` (ghost), `NavigationBar`, `ListTile`, `Switch`, `SnackBar` for errors, `showModalBottomSheet` for a sheet. Wrap screens in `SafeArea`.
- Cards: `Card(elevation: 0, shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(14), side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant)))`. Use the radius of the chosen character (soft 14, sharp 6, round 20).
- Padding and gaps are multiples of 4, screen edge padding 16, buttons at least 48 high. The Design rules and the screen recipes in `ui.md` apply as written.


### Flutter template
File tree (the `android` folder is created by the first workflow run and committed back automatically):
```
.github/workflows/build.yml
.gitignore
PROGRESS.md
pubspec.yaml
analysis_options.yaml
lib/main.dart
test/widget_test.dart
```

`.gitignore`
```
.dart_tool/
.flutter-plugins
.flutter-plugins-dependencies
.packages
.pub-cache/
.pub/
build/
.metadata
*.iml
*.keystore
*.jks
```
Do not add `android/` to `.gitignore`. The generated `android/.gitignore` already skips the files that must not be committed.

`pubspec.yaml`
```yaml
name: demoapp
description: Demo app
publish_to: none
version: 1.0.0+1

environment:
  sdk: ^3.5.0

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
```
The `name` must be a valid Dart package name (lowercase letters, digits, underscores) and must match the import in the test file.

`analysis_options.yaml`
```yaml
linter:
  rules:
    avoid_print: true
    prefer_const_constructors: true
```
Keeping this file in the repo stops `flutter create` from generating a different one that needs an extra package.

`lib/main.dart`
```dart
import 'package:flutter/material.dart';

void main() => runApp(const DemoApp());

class DemoApp extends StatelessWidget {
  const DemoApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Demo App',
      // Seed = "Light main" of the chosen hue (see the look notes above).
      theme: ThemeData(colorSchemeSeed: const Color(0xFF315DD4), useMaterial3: true),
      darkTheme: ThemeData(
        colorSchemeSeed: const Color(0xFF315DD4),
        useMaterial3: true,
        brightness: Brightness.dark,
      ),
      themeMode: ThemeMode.system,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Demo App')),
      body: Center(
        child: Text(
          '$_count',
          style: Theme.of(context).textTheme.displayLarge,
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _count++),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

`test/widget_test.dart`
```dart
import 'package:demoapp/main.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('counter goes up when plus is tapped', (tester) async {
    await tester.pumpWidget(const DemoApp());
    expect(find.text('0'), findsOneWidget);
    await tester.tap(find.byType(FloatingActionButton));
    await tester.pump();
    expect(find.text('1'), findsOneWidget);
  });
}
```
"Done when" for this skeleton: the number goes from 0 to 1 after one tap on the plus button.

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
      - uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: 21
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true
      - name: Restore signing key
        env:
          KEYSTORE_B64: ${{ secrets.DEBUG_KEYSTORE_B64 }}
        run: |
          mkdir -p ~/.android
          echo "$KEYSTORE_B64" | base64 -d > ~/.android/debug.keystore
      - name: Add Android project if missing
        id: android
        run: |
          if [ -d android ]; then
            echo "created=false" >> "$GITHUB_OUTPUT"
          else
            flutter create --platforms=android --org com.example --project-name demoapp .
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
      - run: flutter pub get
      - name: Analyze
        continue-on-error: true
        run: flutter analyze
      - name: Test
        run: flutter test
      - name: Build debug APK
        run: flutter build apk --debug
      - uses: actions/upload-artifact@v7
        with:
          name: app-debug
          path: build/app/outputs/flutter-apk/app-debug.apk
          if-no-files-found: error
      - name: Publish release (version tags only)
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          cp build/app/outputs/flutter-apk/app-debug.apk "app-$TAG.apk"
          if gh release view "$TAG" >/dev/null 2>&1; then
            gh release upload "$TAG" "app-$TAG.apk" --clobber
          else
            gh release create "$TAG" "app-$TAG.apk" --generate-notes
          fi
```
Flutter notes:
- Tag runs skip the "Commit generated Android project" step. Push to main once before the first tag.
- The app id is `<org>.<project-name>`, here `com.example.demoapp`. Set `--org` and `--project-name` to match the chosen id before the first build.
- Pin the Flutter version after the first green build. The log of the step "Set up Flutter" shows the version. Then change the action input to `channel: stable` plus `flutter-version: <that version>`, commit as `ci: pin Flutter version`, and record it in `PROGRESS.md`. This stops a new Flutter release from breaking a working build.
- The debug APK is large (about 100 MB or more). That is normal. A smaller APK comes from `flutter build apk --release` (output `build/app/outputs/flutter-apk/app-release.apk`). The Flutter template signs release builds with the debug key, so the fixed key still applies. If you switch, change the path in the upload step and the release step too.
- Debug builds show a red error screen on exceptions, which helps the user report problems with a screenshot.
- Native settings (permissions, app name) are edited in `android/app/src/main/AndroidManifest.xml`. Read the file first.

