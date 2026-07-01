# OcrLiteAndroidNcnn 新手编译打包教程

这份文档按“第一次把 APK 打出来”为目标写。建议先编译 CPU 版，依赖少、包更小、手机 Android 5.0(API 21) 以上即可运行；CPU 版成功后，再尝试 GPU/Vulkan 版。

## 1. 先确认你要编译什么

项目里有三个主要模块：

```text
app         Kotlin Demo App，最终打 APK 常用这个
appjava     Java Demo App，可选
OcrLibrary  OCR 核心库，包含 C++/JNI，可打 AAR
```

编译产物分两类：

```text
CPU 版：不使用 Vulkan，最低 API 21，优先建议新手先打这个
GPU 版：使用 Vulkan，最低 API 24，需要额外 ncnn-vulkan 依赖
```

## 2. 安装基础工具

1. 安装 Android Studio。
2. 打开 Android Studio 的 SDK Manager。
3. 在 SDK Platforms 中安装 Android API 33。
4. 在 SDK Tools 中安装：
   - Android SDK Build-Tools
   - Android SDK Platform-Tools
   - NDK
   - CMake 3.22.1

本项目的 `OcrLibrary/build.gradle` 指定了 CMake 版本 `3.22.1`。如果本机没有这个版本，Gradle 会在 native 配置阶段失败。

## 3. 打开项目并等待同步

用 Android Studio 打开项目根目录：

```text
D:\morefun\github\OcrLiteAndroidNcnn
```

打开后先等待 Gradle Sync 完成。当前项目已经把 Gradle 下载源配置为腾讯云，Maven 仓库配置为阿里云，正常情况下不需要再手工改仓库。

如果提示找不到 SDK，确认项目根目录下有 `local.properties`，并且里面的 `sdk.dir` 指向本机 Android SDK 目录，例如：

```properties
sdk.dir=C\:\\Users\\你的用户名\\AppData\\Local\\Android\\Sdk
```

## 4. 准备 OpenCV 依赖

下载 `opencv-mobile-3.4.15-android.7z`，原项目文档提供的地址是：

```text
https://gitee.com/benjaminwan/ocr-lite-android-ncnn/attach_files/843219/download/opencv-mobile-3.4.15-android.7z
```

解压后，把里面的 `sdk` 目录放到：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/sdk
```

最终目录必须长这样：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/sdk
└── native
    ├── jni
    └── staticlibs
```

快速检查：

```powershell
Test-Path .\OcrLibrary\src\sdk\native\jni
```

输出 `True` 才说明 OpenCV 路径放对了。

## 5. 准备 ncnn 依赖

本项目使用 ncnn 20221128。下载地址：

```text
https://github.com/Tencent/ncnn/releases/tag/20221128
```

### 5.1 只打 CPU 版

新手建议先只打 CPU 版。下载下面两个包任选一个：

```text
ncnn-20221128-android.zip
ncnn-20221128-android-shared.zip
```

解压后，把 ABI 目录放到：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/main/ncnn
```

最终目录必须长这样：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/main/ncnn
├── arm64-v8a
├── armeabi-v7a
├── x86
└── x86_64
```

每个 ABI 目录下面都应该有：

```text
lib/cmake/ncnn/ncnn.cmake
```

快速检查：

```powershell
Test-Path .\OcrLibrary\src\main\ncnn\arm64-v8a\lib\cmake\ncnn\ncnn.cmake
```

输出 `True` 才说明 CPU 版 ncnn 路径放对了。

### 5.2 需要打 GPU/Vulkan 版

如果要打 GPU 版，再下载下面两个包任选一个：

```text
ncnn-20221128-android-vulkan.zip
ncnn-20221128-android-vulkan-shared.zip
```

解压后，把 ABI 目录放到：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/main/ncnn-vulkan
```

最终目录必须长这样：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/main/ncnn-vulkan
├── arm64-v8a
├── armeabi-v7a
├── x86
└── x86_64
```

快速检查：

```powershell
Test-Path .\OcrLibrary\src\main\ncnn-vulkan\arm64-v8a\lib\cmake\ncnn\ncnn.cmake
```

输出 `True` 才说明 GPU 版 ncnn 路径放对了。

### 5.3 修改 ncnn.cmake

原项目文档要求：解压 ncnn 后，需要修改每个 ABI 目录下的：

```text
lib/cmake/ncnn/ncnn.cmake
```

找到这一行：

```cmake
INTERFACE_COMPILE_OPTIONS "-fno-rtti;-fno-exceptions"
```

把它注释掉：

```cmake
# INTERFACE_COMPILE_OPTIONS "-fno-rtti;-fno-exceptions"
```

如果只打 CPU 版，要改 `OcrLibrary/src/main/ncnn` 下 4 个 ABI 的文件；如果打 GPU 版，还要改 `OcrLibrary/src/main/ncnn-vulkan` 下 4 个 ABI 的文件。

## 6. 确认模型文件已经存在

模型文件应该在：

```text
OcrLiteAndroidNcnn/OcrLibrary/src/main/assets
```

至少应包含：

```text
angle_op.bin
angle_op.param
crnn_lite_op.bin
crnn_lite_op.param
dbnet_op.bin
dbnet_op.param
keys.txt
```

当前仓库已经有这些文件，一般不需要额外下载。

## 7. 用命令行打 APK

在项目根目录打开 PowerShell：

```powershell
cd D:\morefun\github\OcrLiteAndroidNcnn
```

先确认 Gradle 能启动：

```powershell
.\gradlew.bat --version
```

打 CPU Release APK：

```powershell
.\gradlew.bat :app:assembleCpuRelease
```

输出 APK 在：

```text
app/build/outputs/apk/cpu/release/
```

如果要打 GPU Release APK：

```powershell
.\gradlew.bat :app:assembleGpuRelease
```

输出 APK 在：

```text
app/build/outputs/apk/gpu/release/
```

如果要一次打全部 app 变体：

```powershell
.\gradlew.bat :app:assembleRelease
```

注意：一次打全部 Release 会同时构建 CPU 和 GPU 变体，所以 `ncnn` 和 `ncnn-vulkan` 两套依赖都要准备好。

## 8. 用 Android Studio 打 APK

1. 打开左侧或底部的 Build Variants 面板。
2. 把 `app` 和 `OcrLibrary` 都选成同一种变体：
   - `cpuDebug`
   - `cpuRelease`
   - `gpuDebug`
   - `gpuRelease`
3. 新手建议先选 `cpuRelease`。
4. 菜单选择 `Build -> Build Bundle(s) / APK(s) -> Build APK(s)`。

不要让 `app` 是 CPU、`OcrLibrary` 是 GPU，或者反过来。两个模块的变体必须一致。

## 9. 打 AAR

如果你不是要安装 Demo APK，而是要把 OCR 引擎库打成 AAR 给别的项目用，执行：

```powershell
.\gradlew.bat :OcrLibrary:assembleCpuRelease
```

输出 AAR 在：

```text
OcrLibrary/build/outputs/aar/
```

GPU 版 AAR：

```powershell
.\gradlew.bat :OcrLibrary:assembleGpuRelease
```

## 10. 常见错误和处理方向

### 10.1 找不到 ncnn

错误示例：

```text
CMake Error at CMakeLists.txt:12 (find_package):
By not providing "Findncnn.cmake" in CMAKE_MODULE_PATH ...
```

原因：`OcrLibrary/src/main/ncnn` 或 `OcrLibrary/src/main/ncnn-vulkan` 没放好，或者目录多套了一层。

按当前项目的 CMake 配置：

```text
CPU 版找这里：
OcrLibrary/src/main/ncnn/${ANDROID_ABI}/lib/cmake/ncnn

GPU 版找这里：
OcrLibrary/src/main/ncnn-vulkan/${ANDROID_ABI}/lib/cmake/ncnn
```

例如 CPU + arm64-v8a 必须能找到：

```text
OcrLibrary/src/main/ncnn/arm64-v8a/lib/cmake/ncnn/ncnn.cmake
```

### 10.2 找不到 OpenCV

常见原因：`sdk` 目录放错层级。

正确路径是：

```text
OcrLibrary/src/sdk/native/jni
```

不是：

```text
OcrLibrary/src/main/sdk
OcrLibrary/sdk
OcrLibrary/src/opencv-mobile-3.4.15-android/sdk
```

### 10.3 Namespace not specified

如果当前根 `build.gradle` 使用 Android Gradle Plugin 8.x，可能会报：

```text
Namespace not specified. Specify a namespace in the module's build file
```

原因：AGP 8.x 要求每个 Android 模块在 `build.gradle` 的 `android {}` 里显式声明 `namespace`。

当前 `appjava` 已经有：

```gradle
namespace 'com.benjaminwan.ocr.java'
```

如果 `app` 或 `OcrLibrary` 报这个错，需要按 Manifest 里的包名补：

```gradle
// app/build.gradle
android {
    namespace 'com.benjaminwan.ocr.ncnn'
}

// OcrLibrary/build.gradle
android {
    namespace 'com.benjaminwan.ocrlibrary'
}
```

这属于 Gradle/AGP 版本兼容问题，不是 ncnn 或 OpenCV 问题。

### 10.4 下载依赖很慢或失败

当前项目已在 `settings.gradle` 中配置阿里云 Maven 仓库，并保留 JitPack。JitPack 不能删除，因为项目里有：

```gradle
implementation 'com.github.chrisbanes:PhotoView:2.3.0'
```

这个依赖通常需要从 JitPack 获取。

### 10.5 清理后重新编译

如果换过 ncnn、OpenCV、NDK、CMake，建议清理 native 缓存：

```powershell
Remove-Item .\OcrLibrary\.cxx -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item .\OcrLibrary\build -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item .\app\build -Recurse -Force -ErrorAction SilentlyContinue
.\gradlew.bat :app:assembleCpuRelease
```

不要删除 `OcrLibrary/src/main/ncnn`、`OcrLibrary/src/main/ncnn-vulkan`、`OcrLibrary/src/sdk`，这些是你手工准备的依赖。

## 11. 推荐的新手检查清单

打包前逐项确认：

```text
[ ] Android Studio 能打开项目
[ ] SDK API 33 已安装
[ ] NDK 已安装
[ ] CMake 3.22.1 已安装
[ ] OcrLibrary/src/sdk/native/jni 存在
[ ] OcrLibrary/src/main/ncnn/arm64-v8a/lib/cmake/ncnn/ncnn.cmake 存在
[ ] 只打 CPU 时执行 :app:assembleCpuRelease
[ ] 打 GPU 时额外准备 ncnn-vulkan
[ ] app 和 OcrLibrary 的 Build Variant 保持一致
```

第一目标只需要跑通：

```powershell
.\gradlew.bat :app:assembleCpuRelease
```

这个命令成功后，再考虑 GPU 版、AAR 或全量 Release。
