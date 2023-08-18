# FlutterMixUp

## 背景
目前Flutter的应用越来越多，那么如何在已有的iOS&Android项目中嵌入一个Flutter Module？接下来我们将介绍一下如何完成这个操作。这里先只介绍源码混编的方式(这种方式需要在开发环境配置好Flutter sdk)，至于集成framework和aar的方式后续再讲。

## 开发环境
- Flutter版本: 3.13.0
- Xcode Version 14.3.1 (14E300c)
- Android Studio Giraffe | 2022.3.1    
- macOS 13.4.1 (Apple M2 Pro)
- Gradle版本: 8.1.0
- kotlin版本: 1.8.10
- Apple Swift version 5.8.1 (swiftlang-5.8.0.124.5 clang-1403.0.22.11.100)
- Dart版本: 2.19.6


## Flutter Module项目
首先创建一个空目录，例如：`some/path/`
接下来执行以下命令（这里假设你的flutter sdk环境已配置完毕）：
```
// 到达指定目录
cd some/path/

// 创建Flutter Module
flutter create -t module --org com.example flutter_mix_up_demo
```
这会创建一个 `some/path/my_flutter/` 的 Flutter 模块项目，其中包含一些 Dart 代码来帮助你入门以及一个隐藏的子文件夹 `.android/` 和 `.ios/`。 `.android` 文件夹包含一个 Android 项目，该项目不仅可以帮助你通过 `flutter run` 运行这个 Flutter 模块的独立应用，而且还可以作为封装程序来帮助引导 Flutter 模块作为可嵌入的 Android 库。iOS同理。
## Native项目
**将已有的Android和iOS项目全部放到上面提到的 `some/path/` 目录下，这一步是必须的。**     
假设已有的Android项目为`AndroidFlutterMixUpDemo`，iOS项目为：`iOSFlutterMixUpDemo`
### iOS
打开iOS项目的 Podfile 文件，添加如下代码
```
platform :ios, '12.0'

target 'iOSFlutterMixUpDemo' do
  use_frameworks!

  flutter_application_path = '../flutter_mix_up_demo'
  load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')
  install_all_flutter_pods(flutter_application_path)
end

post_install do |installer|
  flutter_post_install(installer) if defined?(flutter_post_install)
end
```
将上面的`flutter_mix_up_demo`替换成你的Flutter Module的名称和路径。执行 `pod install`，编译或者run iOS主项目，编译成功即可
### Android
进入Android项目根目录，打开`AndroidFlutterMixUpDemo/settings.gradle`，插入以下代码：
```
// Android项目名称，无需修改
rootProject.name = "AndroidFlutterMixUpDemo"
// Include the host app project.一般无需修改
include ':app'

setBinding(new Binding([gradle: this]))                      
evaluate(new File(                                
        settingsDir.parentFile,                                      
        'flutter_mix_up_demo/.android/include_flutter.groovy'
))

// 这个配置可以在Android studio打开安卓主项目时编辑或修改flutter代码.可加可不加，不加的话只是看不到Flutter Module代码，不会影响编译及运行
include ':flutter_mix_up_demo'
project(':flutter_mix_up_demo').projectDir = new File("../flutter_mix_up_demo")
```
打开Android项目的 `AndroidFlutterMixUpDemo/app/build.gradle`， 在依赖中插入一下代码：
```
dependencies {
  implementation project(':flutter')
}
```
执行Gradle sync操作。

如果以后Google官方修改了依赖问题那么应该就可以直接成功，当时目前我用的最新版本还是会出现依赖拉去问题，如果出现依赖问题，那么打开 `AndroidFlutterMixUpDemo/settings.gradle`，将dependencyResolutionManagement改为如下配置：
```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS) // 这里要修改为 PREFER_SETTINGS
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
执行执行Gradle sync操作，这次不出意外的话就可以成功了，如果还是拉去不到，那么考虑下翻墙等网络原因，如果没有条件，可以替换成国内的下载源，至于国内的下载源可以直接在网上搜索获取。

到这一步Android+iOS+Flutter源码混编的教程就完毕了，对于后续具体开发的细节就不在这篇文章介绍了。

**[Demo示例](https://github.com/houboye/FlutterMixUp)**

## 参考文档
- https://flutter.cn/docs/add-to-app/android/project-setup
- https://flutter.cn/docs/add-to-app/ios/project-setup