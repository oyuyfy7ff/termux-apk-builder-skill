# kotlin.md (companion file of apk.md)

Use when: the chosen stack is Kotlin (native). Needed before Phase 1.
Read apk.md first. It holds the rules and the phases. This file adds detail and is not complete alone.

## Kotlin look (design notes)
The template builds views in code, so apply the look in code. Neutral colors come from the generator at hue 260 and saturation 0.3; the accent is the "Light main" color of the chosen hue (dark: the dark accent from the table below, text on it `#0e1624`):
```kotlin
private fun isNight() =
    (resources.configuration.uiMode and android.content.res.Configuration.UI_MODE_NIGHT_MASK) ==
        android.content.res.Configuration.UI_MODE_NIGHT_YES

private fun dp(v: Int) = (v * resources.displayMetrics.density).toInt()

private fun c(light: String, dark: String) =
    android.graphics.Color.parseColor(if (isNight()) dark else light)

private fun shape(fill: Int, stroke: Int) = android.graphics.drawable.GradientDrawable().apply {
    setColor(fill)
    setStroke(dp(1), stroke)
    cornerRadius = dp(14).toFloat()
}

// inside onCreate, before building the views
val bg = c("#f7f8fa", "#0e0f11")
val surface = c("#ffffff", "#17181b")
val surface2 = c("#eceff3", "#222428")
val text = c("#191b1d", "#eaebed")
val muted = c("#5b5e63", "#a2a5a9")
val line = c("#dcdee1", "#2e3034")
val danger = c("#b7191c", "#f47b74")
val ok = c("#137738", "#76cf8a")
val main = c("#315DD4", "#8BAFFF")   // accent: take it from the hue guide
val onMain = c("#FFFFFF", "#0E1624")
```
- Dark accent colors by hue: red #fe8b83, orange #f49752, yellow #cdaf33, green #68ca80, teal #00caca, blue #6eb6ff, violet #a9a5ff, pink #f38abe.
- Set the root view background to `bg`, text colors to `text` and `muted`, buttons to `shape(main, main)` with `onMain` text, `isAllCaps = false` and `minHeight = dp(48)`, cards to `shape(surface, line)`.
- Padding at least `dp(16)` on screen edges and `dp(8)` between items. Keep the framework theme from the template.


### Kotlin (native) template
File tree:
```
.github/workflows/build.yml
.gitignore
PROGRESS.md
settings.gradle.kts
build.gradle.kts
gradle.properties
app/build.gradle.kts
app/src/main/AndroidManifest.xml
app/src/main/java/com/example/demoapp/MainActivity.kt
```
The path `com/example/demoapp` must match the app id with slashes. This template has no dependencies, so the first build has the fewest things that can break.

`.gitignore`
```
.gradle/
build/
local.properties
*.keystore
*.jks
.idea/
*.iml
```

`settings.gradle.kts`
```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "DemoApp"
include(":app")
```

`build.gradle.kts`
```kotlin
plugins {
    id("com.android.application") version "8.13.0" apply false
    id("org.jetbrains.kotlin.android") version "2.3.21" apply false
}
```

`gradle.properties`
```
org.gradle.jvmargs=-Xmx2g -Dfile.encoding=UTF-8
android.useAndroidX=true
```

`app/build.gradle.kts`
```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.example.demoapp"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.example.demoapp"
        minSdk = 26
        targetSdk = 36
        versionCode = 1
        versionName = "1.0.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}

kotlin {
    compilerOptions {
        jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_17)
    }
}
```

`app/src/main/AndroidManifest.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:label="Demo App"
        android:theme="@android:style/Theme.DeviceDefault.Light.NoActionBar">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
`android:exported="true"` on the launcher activity is required for apps that target Android 12 or higher.

`app/src/main/java/com/example/demoapp/MainActivity.kt`
```kotlin
package com.example.demoapp

import android.app.Activity
import android.os.Bundle
import android.view.Gravity
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView

class MainActivity : Activity() {
    private var count = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val label = TextView(this).apply {
            text = "0"
            textSize = 48f
            gravity = Gravity.CENTER
        }
        val button = Button(this).apply {
            text = "Add"
            setOnClickListener {
                count += 1
                label.text = count.toString()
            }
        }
        val root = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            gravity = Gravity.CENTER
            addView(label)
            addView(button)
        }
        setContentView(root)
    }
}
```
"Done when" for this skeleton: the number goes from 0 to 1 after one tap on Add.

`.github/workflows/build.yml`
```yaml
name: build
on:
  push:
  workflow_dispatch:
permissions:
  contents: write # needed to publish releases
jobs:
  apk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: 21
      - name: Install Gradle
        env:
          GRADLE_VERSION: "8.14.3"
        run: |
          curl -fsSL "https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip" -o gradle.zip
          unzip -q gradle.zip -d "$HOME/gradle"
          echo "$HOME/gradle/gradle-${GRADLE_VERSION}/bin" >> "$GITHUB_PATH"
      - name: Restore signing key
        env:
          KEYSTORE_B64: ${{ secrets.DEBUG_KEYSTORE_B64 }}
        run: |
          mkdir -p ~/.android
          echo "$KEYSTORE_B64" | base64 -d > ~/.android/debug.keystore
      - name: Lint
        continue-on-error: true
        run: gradle lintDebug --no-daemon
      - name: Unit tests
        run: gradle testDebugUnitTest --no-daemon
      - name: Build debug APK
        run: gradle assembleDebug --no-daemon
      - name: List APK files
        run: find app -name "*.apk"
      - uses: actions/upload-artifact@v7
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/*.apk
          if-no-files-found: error
      - name: Publish release (version tags only)
        if: startsWith(github.ref, 'refs/tags/v')
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          cp app/build/outputs/apk/debug/app-debug.apk "app-$TAG.apk"
          if gh release view "$TAG" >/dev/null 2>&1; then
            gh release upload "$TAG" "app-$TAG.apk" --clobber
          else
            gh release create "$TAG" "app-$TAG.apk" --generate-notes
          fi
```
Kotlin notes:
- There is no Gradle wrapper (Termux cannot make one), so the workflow installs a fixed Gradle version (set once in `GRADLE_VERSION`). Keep Gradle, the Android Gradle plugin and Kotlin as in the Known-good versions table.
- Adding libraries: add `dependencies { implementation("<group>:<name>:<version>") }` to `app/build.gradle.kts`. Look up the exact version first. Never guess it.
- Using AndroidX or Compose later: add the libraries first, build, and only then use them in code. Compose also needs the Compose compiler plugin matching the Kotlin version.
- If lint finds only warnings (for example "missing app icon"), they do not stop the build.

## Edge-to-edge (Android 15 and higher)
With `targetSdk = 36`, Android 15 and higher draw the app behind the status bar and the gesture bar, so text at the top or bottom edge can be covered. The skeleton is not affected (its content is centered). Add this when the first real screen is built (Phase 3), not in the skeleton. NOT proven: it was not compiled here. In the first build ask the user for a screenshot of the top and the bottom edge, as for Capacitor.
```kotlin
@Suppress("DEPRECATION")
private fun keepClearOfSystemBars(view: android.view.View) {
    view.setOnApplyWindowInsetsListener { v, insets ->
        v.setPadding(
            insets.systemWindowInsetLeft,
            insets.systemWindowInsetTop,
            insets.systemWindowInsetRight,
            insets.systemWindowInsetBottom
        )
        insets
    }
}
```
Call `keepClearOfSystemBars(root)` just before `setContentView(root)`. The old inset fields are deprecated but exist on every API level the template supports (`minSdk = 26`); warnings do not stop the build. If the screenshot shows the text still covered, report it in `LESSONS.md` before changing anything.

## Crash screen (development)
Add at the top of `onCreate`, after `super.onCreate(savedInstanceState)`, while developing. Remove it at release.
```kotlin
val crashFile = java.io.File(filesDir, "last_crash.txt")
Thread.setDefaultUncaughtExceptionHandler { _, e ->
    crashFile.writeText(e.stackTraceToString())
    android.os.Process.killProcess(android.os.Process.myPid())
}
if (crashFile.exists()) {
    setContentView(TextView(this).apply {
        text = crashFile.readText()
        textSize = 12f
    })
    crashFile.delete()
    return
}
```

## Icon recipe
Text files only, works in Termux. Kotlin version, needs `minSdk = 26` (the template already uses 26):
- `app/src/main/res/values/colors.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="ic_launcher_background">#3F51B5</color>
</resources>
```
- `app/src/main/res/drawable/ic_launcher_foreground.xml`
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M54,30 L78,78 L30,78 Z" />
</vector>
```
- `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```
- In the manifest, add to `<application`: `android:icon="@mipmap/ic_launcher"`.
