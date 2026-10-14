# Wear OS Test App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create and run a minimal standalone Wear OS app (Kotlin + Jetpack Compose) with a counter button that vibrates the watch, to validate the full build → install → run flow for Wear OS development on this machine.

**Architecture:** Single-module Android Gradle project (`app`), standalone Wear OS app (no phone companion module). One `MainActivity` hosting one Compose screen (`CounterScreen`) with local UI state and a vibration side-effect on click.

**Tech Stack:** Kotlin 1.9.24, Android Gradle Plugin 8.5.2, Gradle 8.9, Jetpack Compose for Wear OS (`androidx.wear.compose` 1.3.1), compileSdk/targetSdk 34, minSdk 30.

**Testing note:** Per the approved design spec (`docs/superpowers/specs/2026-10-14-wear-os-test-app-design.md`), this is a throwaway validation prototype with no application logic worth unit-testing. There is no automated test suite — verification is a manual build + run + tap pass at the end (Task 5). Tasks below substitute "write the file" / "verify it's wired correctly" steps for the usual red/green test cycle, but keep the same bite-sized, frequent-commit discipline.

**Machine context already verified during planning:**
- Android SDK at `~/Android/Sdk`, platform `android-34` and build-tools `34.0.0` already installed.
- No Wear OS system image or AVD installed yet — Task 4 provisions one.
- No system `gradle` binary and no `gradlew` yet — Task 5 opens the project in Android Studio (already installed), which auto-generates the Gradle wrapper (`gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.jar`) on first sync, matching the `gradle-wrapper.properties` version pinned in Task 1.

---

### Task 1: Root Gradle project configuration

**Files:**
- Create: `settings.gradle.kts`
- Create: `build.gradle.kts`
- Create: `gradle.properties`
- Create: `gradle/wrapper/gradle-wrapper.properties`
- Create: `.gitignore`

- [ ] **Step 1: Create `settings.gradle.kts`**

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

rootProject.name = "SmartWatch"
include(":app")
```

- [ ] **Step 2: Create root `build.gradle.kts`**

```kotlin
plugins {
    id("com.android.application") version "8.5.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}
```

- [ ] **Step 3: Create `gradle.properties`**

```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
kotlin.code.style=official
```

- [ ] **Step 4: Create `gradle/wrapper/gradle-wrapper.properties`**

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.9-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

This pins the exact Gradle version Android Studio will use to generate `gradlew` in Task 5 — it must stay compatible with AGP 8.5.2 (requires Gradle 8.7+).

- [ ] **Step 5: Create `.gitignore`**

```gitignore
*.iml
.gradle/
/local.properties
/.idea/
.DS_Store
/build
/captures
.externalNativeBuild
.cxx
/app/build
```

- [ ] **Step 6: Verify the files are in place**

Run: `find . -maxdepth 3 -type f | sort`
Expected output includes exactly these new files (plus the existing `docs/` tree):
```
./.gitignore
./build.gradle.kts
./gradle.properties
./gradle/wrapper/gradle-wrapper.properties
./settings.gradle.kts
```

- [ ] **Step 7: Commit**

```bash
git add settings.gradle.kts build.gradle.kts gradle.properties gradle/wrapper/gradle-wrapper.properties .gitignore
git commit -m "Add root Gradle project configuration"
```

---

### Task 2: App module configuration (manifest, build file, resources)

**Files:**
- Create: `app/build.gradle.kts`
- Create: `app/src/main/AndroidManifest.xml`
- Create: `app/src/main/res/values/strings.xml`
- Create: `app/src/main/res/values/colors.xml`
- Create: `app/src/main/res/drawable/ic_launcher_foreground.xml`
- Create: `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`
- Create: `app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml`

- [ ] **Step 1: Create `app/build.gradle.kts`**

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.guilherme.smartwatch"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.guilherme.smartwatch"
        minSdk = 30
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.14"
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }
}

dependencies {
    implementation("androidx.activity:activity-compose:1.9.2")
    implementation(platform("androidx.compose:compose-bom:2024.09.00"))
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.wear.compose:compose-material:1.3.1")
    implementation("androidx.wear.compose:compose-foundation:1.3.1")
    debugImplementation("androidx.compose.ui:ui-tooling")
}
```

- [ ] **Step 2: Create `app/src/main/AndroidManifest.xml`**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-feature
        android:name="android.hardware.type.watch"
        android:required="true" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@android:style/Theme.DeviceDefault">

        <uses-library
            android:name="com.google.android.wearable"
            android:required="true" />

        <meta-data
            android:name="com.google.android.wearable.standalone"
            android:value="true" />

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:taskAffinity=""
            android:theme="@android:style/Theme.DeviceDefault">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

`com.google.android.wearable.standalone = true` and the `android.hardware.type.watch` feature mark this as a standalone watch app, per the design spec (no phone companion).

- [ ] **Step 3: Create `app/src/main/res/values/strings.xml`**

```xml
<resources>
    <string name="app_name">SmartWatch Test</string>
</resources>
```

- [ ] **Step 4: Create `app/src/main/res/values/colors.xml`**

```xml
<resources>
    <color name="ic_launcher_background">#1976D2</color>
</resources>
```

- [ ] **Step 5: Create `app/src/main/res/drawable/ic_launcher_foreground.xml`**

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M54,24 C65.05,24 74,32.95 74,44 C74,55.05 65.05,64 54,64 C42.95,64 34,55.05 34,44 C34,32.95 42.95,24 54,24 Z M54,70 C68,70 80,76 80,84 L80,90 L28,90 L28,84 C28,76 40,70 54,70 Z" />
</vector>
```

A simple white "person" glyph on a solid background — just enough for a launcher icon, no binary image assets needed.

- [ ] **Step 6: Create `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`**

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```

- [ ] **Step 7: Create `app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml`**

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```

Since `minSdk` is 30 (> 26), the adaptive icon alone covers all supported devices — no legacy density-specific PNGs are needed.

- [ ] **Step 8: Verify the resource tree**

Run: `find app/src/main -type f | sort`
Expected output:
```
app/src/main/AndroidManifest.xml
app/src/main/res/drawable/ic_launcher_foreground.xml
app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml
app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml
app/src/main/res/values/colors.xml
app/src/main/res/values/strings.xml
```

- [ ] **Step 9: Commit**

```bash
git add app/build.gradle.kts app/src/main/AndroidManifest.xml app/src/main/res
git commit -m "Add app module config, manifest, and launcher icon"
```

---

### Task 3: Compose UI — counter screen with vibration

**Files:**
- Create: `app/src/main/java/com/guilherme/smartwatch/presentation/CounterScreen.kt`
- Create: `app/src/main/java/com/guilherme/smartwatch/MainActivity.kt`

- [ ] **Step 1: Create `app/src/main/java/com/guilherme/smartwatch/presentation/CounterScreen.kt`**

```kotlin
package com.guilherme.smartwatch.presentation

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.wear.compose.material.Button
import androidx.wear.compose.material.MaterialTheme
import androidx.wear.compose.material.Text

@Composable
fun CounterScreen(onIncrement: () -> Unit) {
    var count by remember { mutableIntStateOf(0) }

    MaterialTheme {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(8.dp),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(text = count.toString())
            Button(onClick = {
                count++
                onIncrement()
            }) {
                Text(text = "+1")
            }
        }
    }
}
```

`onIncrement` is a plain callback (not a vibration call directly) so this composable has no dependency on `android.os.Vibrator` — the hardware side-effect lives in `MainActivity`, keeping the UI unit independently previewable.

- [ ] **Step 2: Create `app/src/main/java/com/guilherme/smartwatch/MainActivity.kt`**

```kotlin
package com.guilherme.smartwatch

import android.content.Context
import android.os.Build
import android.os.Bundle
import android.os.VibrationEffect
import android.os.Vibrator
import android.os.VibratorManager
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import com.guilherme.smartwatch.presentation.CounterScreen

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val vibrator = getVibrator()

        setContent {
            CounterScreen(onIncrement = { vibrator?.vibrateShort() })
        }
    }

    private fun getVibrator(): Vibrator? {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            val manager = getSystemService(Context.VIBRATOR_MANAGER_SERVICE) as? VibratorManager
            manager?.defaultVibrator
        } else {
            @Suppress("DEPRECATION")
            getSystemService(Context.VIBRATOR_SERVICE) as? Vibrator
        }
    }
}

private fun Vibrator.vibrateShort() {
    if (hasVibrator()) {
        vibrate(VibrationEffect.createOneShot(50, VibrationEffect.DEFAULT_AMPLITUDE))
    }
}
```

Branches on `Build.VERSION.SDK_INT` because `VIBRATOR_MANAGER_SERVICE` only exists from API 31 (`S`) onward; below that, the deprecated `VIBRATOR_SERVICE` path is the only option. `minSdk` is 30, so both branches are reachable in practice.

- [ ] **Step 3: Verify the source tree**

Run: `find app/src/main/java -type f | sort`
Expected output:
```
app/src/main/java/com/guilherme/smartwatch/MainActivity.kt
app/src/main/java/com/guilherme/smartwatch/presentation/CounterScreen.kt
```

- [ ] **Step 4: Commit**

```bash
git add app/src/main/java
git commit -m "Add counter screen UI and vibration on increment"
```

---

### Task 4: Provision a Wear OS emulator

**Context:** Checked during planning — no Wear OS system image or AVD exists yet on this machine. The host is `x86_64`, so the Wear OS 4 (API 33) x86_64 image is the best fit (the API 30 image only ships as 32-bit `x86`, which is slower/deprecated on modern emulator builds). `minSdk` 30 still works fine on an API 33 emulator.

**Files:** none (SDK/AVD provisioning only, no project files).

- [ ] **Step 1: Install the Wear OS system image**

Run:
```bash
"$HOME/Android/Sdk/cmdline-tools/latest/bin/sdkmanager" --install "system-images;android-33;android-wear;x86_64"
```
Expected: downloads and installs the package; exits 0. Accept any license prompt by typing `y`.

- [ ] **Step 2: Create the Wear OS AVD**

Run:
```bash
"$HOME/Android/Sdk/cmdline-tools/latest/bin/avdmanager" create avd \
  -n Wear_OS_Test \
  -k "system-images;android-33;android-wear;x86_64" \
  -d wearos_small_round
```
Expected: prompts "Do you wish to create a custom hardware profile? [no]" — answer `no` (press Enter). Exits 0 with the AVD created at `~/.android/avd/Wear_OS_Test.avd`.

- [ ] **Step 3: Verify the AVD was created**

Run: `"$HOME/Android/Sdk/cmdline-tools/latest/bin/avdmanager" list avd`
Expected: output includes a block with `Name: Wear_OS_Test`.

No commit for this task — it changes local SDK/AVD state only, not the project repo.

---

### Task 5: Build and run on the emulator (manual verification)

**Files:** none created in this task; it exercises the project built in Tasks 1-3 on the emulator from Task 4.

- [ ] **Step 1: Open the project in Android Studio**

Launch Android Studio → "Open" → select `/home/guilherme/Documents/projetos/smart-watch`. Trust the project if prompted.

- [ ] **Step 2: Let Android Studio generate the Gradle wrapper and sync**

Android Studio detects the project has Gradle build files but no `gradlew`/`gradle-wrapper.jar`, and on first sync generates them using the Gradle version pinned in `gradle/wrapper/gradle-wrapper.properties` (8.9). Wait for the sync to finish (status bar).

Expected: sync completes with no errors; `./gradlew`, `./gradlew.bat`, and `gradle/wrapper/gradle-wrapper.jar` now exist in the project root.

If Android Studio does not prompt automatically: open the Gradle tool window (View → Tool Windows → Gradle) and click "Sync Project with Gradle Files" (the icon with two circular arrows) — this forces wrapper generation.

- [ ] **Step 3: Verify the build from the command line**

Run:
```bash
ANDROID_HOME="$HOME/Android/Sdk" ./gradlew assembleDebug
```
Expected: `BUILD SUCCESSFUL`, with `app/build/outputs/apk/debug/app-debug.apk` produced.

- [ ] **Step 4: Start the Wear OS emulator**

Run:
```bash
"$HOME/Android/Sdk/emulator/emulator" -avd Wear_OS_Test &
```
Expected: emulator window opens showing a round Wear OS watch face. Wait until it finishes booting (home screen visible).

- [ ] **Step 5: Install and launch the app**

Run:
```bash
"$HOME/Android/Sdk/platform-tools/adb" install -r app/build/outputs/apk/debug/app-debug.apk
"$HOME/Android/Sdk/platform-tools/adb" shell am start -n com.guilherme.smartwatch/.MainActivity
```
Expected: both commands print `Success` / start the activity with no errors; the app opens on the emulator showing `0` and a `+1` button.

- [ ] **Step 6: Manually verify the counter and vibration**

Tap the `+1` button on the emulator (or click it with the mouse) 3 times.
Expected: the number updates `0 → 1 → 2 → 3` on screen. Vibration can't be felt on an emulator, but confirm there's no crash and no `Vibrator`-related error in logcat:
```bash
"$HOME/Android/Sdk/platform-tools/adb" logcat -d | grep -i "com.guilherme.smartwatch" | grep -i -E "exception|crash"
```
Expected: no output (no matching lines).

This step is the manual test pass called for in the design spec — there is no automated test to run here.

---

## Self-Review Notes

- **Spec coverage:** Standalone single-module app (Task 2 manifest), Compose counter UI (Task 3), vibration on click (Task 3 `MainActivity`), manual build/install/tap/vibrate verification (Task 5) — all design spec sections are covered. No persistence, network, or second module was added, matching "out of scope."
- **Type consistency:** `CounterScreen(onIncrement: () -> Unit)` in Task 3 Step 1 matches the call site `CounterScreen(onIncrement = { vibrator?.vibrateShort() })` in Task 3 Step 2. Package `com.guilherme.smartwatch` is consistent across `app/build.gradle.kts` (`namespace`/`applicationId`), the manifest's `.MainActivity` reference, and both Kotlin files' `package` declarations.
- **No automated tests:** intentional per spec — noted explicitly in the plan header and in Task 5 rather than omitted silently.
