A library for loading GliaPlayer.

## Requirements

### Platform: Android Native 
- Android Min SDK version: `23`

### Platform: React Native
- Min React Native Version: `0.74`

## Integration with Flutter

[Documentation](flutter-integration.md)

## Quick Start

### 1. Import the library

```kotlin
implementation("com.gliacloud:gliaplayer:1.0.0-beta05")
```

In `settings.gradle`, make sure you select the `read:packages` scope and for the access token:

```kotlin
pluginManagement {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/livingbio/GliaPlayer-Webview-Android-SDK")
            credentials {
                username = System.getenv("GITHUB_USER_ID") ?: ""
                password = System.getenv("PERSONAL_ACCESS_TOKEN") ?: ""
            }
        }
    }
}

dependencyResolutionManagement {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/livingbio/GliaPlayer-Webview-Android-SDK")
            credentials {
                username = System.getenv("GITHUB_USER_ID") ?: ""
                password = System.getenv("PERSONAL_ACCESS_TOKEN") ?: ""
            }
        }
    }
}
```

### 2. Setup the Google Mobile Ads SDK

https://developers.google.com/ad-manager/mobile-ads-sdk/android/quick-start#kotlin_2


### 3. Load GliaPlayer Ads View

To load an GliaPlayer, use the `GliaPlayer` composable with the `SLOT_KEY`:

```kotlin
import com.gliacloud.gliaplayer.GliaPlayer

GliaPlayer(context).apply {
    initGliaPlayer(slot_key = "{SLOT_KEY}")
    // Register with Mobile Ads SDK for ad integration
    MobileAds.registerWebView(this)
}
```

## License

    Copyright 2026 GliaCloud

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.