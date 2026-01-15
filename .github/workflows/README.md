# GitHub Actions 构建说明

## 注意事项
- 项目中需要 `gradlew` (Gradle Wrapper) 才能运行 GitHub Actions
- 如果 GitHub Actions 失败，请先在本地生成 Gradle Wrapper:
  ```bash
  gradle wrapper
  ```
- 或者从模板项目复制标准的 Gradle Wrapper 文件

## 生成的构建产物
首次成功构建后，您可以在以下位置找到生成的文件：
- Debug APK: `app/build/outputs/apk/debug/app-debug.apk`
- Release APK: `app/build/outputs/apk/release/app-release.apk`
- App Bundle: `app/build/outputs/bundle/release/app-release.aab`

这些文件将通过 Actions Artifacts 提供下载。
