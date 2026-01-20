# Flutter Integration Guide

This document explains how to integrate the GliaPlayer Android SDK, a WebView-based video ad player, into a Flutter project, embedding the native Android View into the Flutter App using Platform View technology.

## Table of Contents

* [Requirements](https://www.google.com/search?q=%23requirements)
* [Step 1: Set up GitHub Credentials](https://www.google.com/search?q=%23step-1-set-up-github-credentials)
* [Step 2: Configure Gradle Repository](https://www.google.com/search?q=%23step-2-configure-gradle-repository)
* [Step 3: Add SDK Dependency](https://www.google.com/search?q=%23step-3-add-sdk-dependency)
* [Step 4: Create Android Platform View](https://www.google.com/search?q=%23step-4-create-android-platform-view)
* [Step 5: Register Platform View](https://www.google.com/search?q=%23step-5-register-platform-view)
* [Step 6: Create Flutter Widget](https://www.google.com/search?q=%23step-6-create-flutter-widget)
* [Step 7: UI Integration](https://www.google.com/search?q=%23step-7-ui-integration)
* [FAQ & Troubleshooting](https://www.google.com/search?q=%23faq--troubleshooting)

---

## Requirements

* Flutter SDK 3.9.0+
* Android minSdk 23+ (Required by GliaPlayer)
* A valid GitHub Personal Access Token (PAT) with `read:packages` permission

---

## Step 1: Set up GitHub Credentials

The GliaPlayer SDK is hosted on GitHub Packages and requires authentication to download.

### Create `gradle.properties`

Add your GitHub credentials to `~/.gradle/gradle.properties`:

```properties
GITHUB_USER_ID=your-github-email@example.com
PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

> **Note**: This file is located in the `.gradle` folder within the user's home directory and will not be committed to the git repository.

### Obtain Personal Access Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens.
2. Click "Generate new token (classic)".
3. Check the `read:packages` permission.
4. Copy the generated token.

---

## Step 2: Configure Gradle Repository

### Modify `android/build.gradle.kts`

Add the GitHub Packages repository to the `allprojects.repositories` block:

```kotlin
allprojects {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/livingbio/GliaPlayer-Webview-Android-SDK")
            credentials {
                username = providers.gradleProperty("GITHUB_USER_ID").orNull
                    ?: System.getenv("GITHUB_USER_ID") ?: ""
                password = providers.gradleProperty("PERSONAL_ACCESS_TOKEN").orNull
                    ?: System.getenv("PERSONAL_ACCESS_TOKEN") ?: ""
            }
            content {
                // Only use this repository for GliaPlayer SDK
                includeGroup("com.gliacloud")
            }
        }
    }
}

```

> **Important**: `content { includeGroup("com.gliacloud") }` is a critical setting. It restricts this repository to be used only for downloading the GliaPlayer SDK, preventing other dependencies from attempting to download from GitHub Packages, which would cause authentication errors.

---

## Step 3: Add SDK Dependency

### Modify `android/app/build.gradle.kts`

Add GliaPlayer to the `dependencies` block:

```kotlin
dependencies {
    // ... existing dependencies

    // GliaPlayer SDK for video ads
    implementation("com.gliacloud:gliaplayer:1.0.0-beta05")
}

```

---

## Step 4: Create Android Platform View

### 4.1 Create ViewFactory

Create the file `android/app/src/main/kotlin/[your-package]/GliaPlayerViewFactory.kt`:

```kotlin
package com.your.package.name

import android.content.Context
import io.flutter.plugin.common.StandardMessageCodec
import io.flutter.plugin.platform.PlatformView
import io.flutter.plugin.platform.PlatformViewFactory

class GliaPlayerViewFactory : PlatformViewFactory(StandardMessageCodec.INSTANCE) {
    override fun create(context: Context, viewId: Int, args: Any?): PlatformView {
        val creationParams = args as? Map<String, Any?>
        return GliaPlayerPlatformView(context, viewId, creationParams)
    }
}

```

### 4.2 Create PlatformView

Create the file `android/app/src/main/kotlin/[your-package]/GliaPlayerPlatformView.kt`:

```kotlin
package com.your.package.name

import android.content.Context
import android.view.View
import android.widget.FrameLayout
import com.gliacloud.gliaplayer.GliaPlayer
import com.google.android.gms.ads.MobileAds
import io.flutter.plugin.platform.PlatformView

class GliaPlayerPlatformView(
    context: Context,
    private val viewId: Int,
    private val creationParams: Map<String, Any?>?
) : PlatformView {

    private val container: FrameLayout = FrameLayout(context)
    private var gliaPlayer: GliaPlayer? = null

    init {
        val slotKey = creationParams?.get("slotKey") as? String
        if (!slotKey.isNullOrEmpty()) {
            try {
                gliaPlayer = GliaPlayer(context).apply {
                    initGliaPlayer(slot_key = slotKey)
                    MobileAds.registerWebView(this)
                    container.addView(this, FrameLayout.LayoutParams(
                        FrameLayout.LayoutParams.MATCH_PARENT,
                        FrameLayout.LayoutParams.MATCH_PARENT
                    ))
                }
            } catch (e: Exception) {
                android.util.Log.e("GliaPlayerView", "Init failed: ${e.message}")
            }
        }
    }

    override fun getView(): View = container

    override fun dispose() {
        gliaPlayer?.let { player ->
            container.removeView(player)
            player.destroy()
        }
        gliaPlayer = null
    }
}

```

---

## Step 5: Register Platform View

### Modify `MainActivity.kt`

```kotlin
package com.your.package.name

import io.flutter.embedding.android.FlutterFragmentActivity
import io.flutter.embedding.engine.FlutterEngine

class MainActivity : FlutterFragmentActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        flutterEngine.platformViewsController.registry.registerViewFactory(
            "{com.your.package.name}/glia_player",
            GliaPlayerViewFactory()
        )
    }
}

```

---

## Step 6: Create Flutter Widget

### 6.1 Create Widget

Create the file `lib/core/ads/glia_player_widget.dart`:

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class GliaPlayerWidget extends StatelessWidget {
  final String slotKey;
  final double height;
  final VoidCallback? onPlatformViewCreated;

  const GliaPlayerWidget({
    super.key,
    required this.slotKey,
    this.height = 250,
    this.onPlatformViewCreated,
  });

  static const String _viewType = 'com.your.package.name/glia_player';

  @override
  Widget build(BuildContext context) {
    if (!Platform.isAndroid) {
      return _buildPlaceholder();
    }

    return SizedBox(
      height: height,
      child: AndroidView(
        viewType: _viewType,
        creationParams: {'slotKey': slotKey},
        creationParamsCodec: const StandardMessageCodec(),
        onPlatformViewCreated: (_) => onPlatformViewCreated?.call(),
      ),
    );
  }

  Widget _buildPlaceholder() {
    return SizedBox(
      height: height,
      child: Container(
        decoration: BoxDecoration(
          color: const Color(0xFF16181C),
          borderRadius: BorderRadius.circular(16),
        ),
        child: const Center(
          child: Icon(Icons.play_circle_outline, size: 48, color: Color(0xFF71767B)),
        ),
      ),
    );
  }
}

```

### 6.2 Create Slot Key Management

Create the file `lib/core/ads/glia_player_ids.dart`:

```dart
import 'dart:io';

class GliaPlayerIds {
  GliaPlayerIds._();

  /// Main video slot for dashboard
  static String mainVideoSlot() {
    if (!Platform.isAndroid) return '';
    return 'your_app_android_slot_key';
  }
}

```

---

## Step 7: UI Integration

### Usage Example

```dart
import 'dart:io' show Platform;
import 'package:your_app/core/ads/glia_player_widget.dart';
import 'package:your_app/core/ads/glia_player_ids.dart';

// In your widget tree
Widget build(BuildContext context) {
  return Column(
    children: [
      // Other widgets...

      // GliaPlayer - Android only
      if (Platform.isAndroid)
        GliaPlayerWidget(
          slotKey: GliaPlayerIds.mainVideoSlot(),
          height: 250,
        ),

      // Other widgets...
    ],
  );
}

```

---

## FAQ & Troubleshooting

### Problem 1: 401 Unauthorized Error

**Error Message:**
Could not GET '.../gliaplayer-1.0.0-beta05.pom'.
Received status code 401 from server: Unauthorized

**Cause:** GitHub credentials are incorrectly configured.

**Solution:**

1. Confirm that `~/.gradle/gradle.properties` exists.
2. Confirm that `GITHUB_USER_ID` and `PERSONAL_ACCESS_TOKEN` are set correctly.
3. Confirm that the PAT has the `read:packages` permission.
4. Confirm that the PAT has not expired.

---

### Problem 2: Other Dependencies Failed to Download

**Error Message:**
Could not resolve io.flutter:flutter_embedding_debug:xxx
Could not HEAD '[...github.com/.../flutter_embedding_debug-xxx.pom](https://www.google.com/search?q=https://...github.com/.../flutter_embedding_debug-xxx.pom)'.
Received status code 401 from server: Unauthorized

**Cause:** The GitHub Packages repository was not restricted to GliaPlayer usage only, causing Gradle to attempt to download all dependencies from GitHub Packages.

**Solution:**
Ensure the `content` filter is added to the repository configuration:

```kotlin
maven {
    url = uri("https://maven.pkg.github.com/...")
    credentials { ... }
    content {
        includeGroup("com.gliacloud")  // Critical!
    }
}

```

---

### Problem 3: Errors Caused by Using `dependencyResolutionManagement`

**Error Message:**
Same as Problem 2. This occurs because `repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)` in `settings.gradle.kts` forces all dependencies to be downloaded from the repositories defined in settings.

**Solution:**
Do not use `dependencyResolutionManagement` in `settings.gradle.kts` to configure the GitHub Packages repository. Instead, use `allprojects.repositories` in `build.gradle.kts` combined with the `content` filter.

---

### Problem 4: Crash on iOS

**Cause:** GliaPlayer only supports Android.

**Solution:**
Use `Platform.isAndroid` in the Flutter widget to check the platform, and display a placeholder for iOS:

```dart
if (!Platform.isAndroid) {
  return _buildPlaceholder();
}

```

---

### Problem 5: MobileAds Not Initialized

**Error Message:**
MobileAds.registerWebView() called before MobileAds.initialize()

**Solution:**
Ensure that MobileAds is initialized in `main.dart`:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await MobileAds.instance.initialize();
  runApp(MyApp());
}

```

---

## Verification Steps

1. Run `flutter clean && flutter pub get`.
2. Run `flutter build apk --debug` to confirm the build succeeds.
3. Run `flutter run -d android` on an Android device.
4. Confirm that the GliaPlayer is displaying correctly.
5. Run on an iOS device to confirm that the placeholder is shown instead of a crash.

---

## File Change List

| File | Action | Description |
| --- | --- | --- |
| `~/.gradle/gradle.properties` | Add | GitHub credentials |
| `android/build.gradle.kts` | Modify | Add GitHub Packages repository |
| `android/app/build.gradle.kts` | Modify | Add GliaPlayer dependency |
| `android/.../GliaPlayerViewFactory.kt` | Add | Platform View Factory |
| `android/.../GliaPlayerPlatformView.kt` | Add | Platform View Implementation |
| `android/.../MainActivity.kt` | Modify | Register Platform View |
| `lib/core/ads/glia_player_widget.dart` | Add | Flutter Widget |
| `lib/core/ads/glia_player_ids.dart` | Add | Slot Key Management |

---

## References

* [Flutter Platform Views](https://docs.flutter.dev/platform-integration/android/platform-views)
* [GitHub Packages - Working with the Gradle registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)
* [Google Mobile Ads - WebView integration](https://developers.google.com/admob/android/webview)