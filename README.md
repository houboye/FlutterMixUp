# FlutterMixUp

## 背景

iOS+Android 嵌入Flutter Module 混编demo

Gradle修改下载源
```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
    repositories {
        google()
        mavenCentral()
        jcenter()
        maven {
            url 'some/path/flutter_module/build/host/outputs/repo'
            // This is relative to the location of the build.gradle file
            // if using a relative path.
        }
        maven {
            url 'https://storage.googleapis.com/download.flutter.io'
        }
    }
}
```