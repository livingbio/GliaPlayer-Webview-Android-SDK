A library for loading GliaPlayer.

## Quick Start

Import the library:

```kotlin
implementation("com.gliacloud:gliaplayer:1.0.0-beta01")
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

To load an GliaPlayer, use the `GliaPlayer` composable:

```kotlin
GliaPlayer(
    modifier = Modifier
        .width(480.dp)
        .height(320.dp)
)
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