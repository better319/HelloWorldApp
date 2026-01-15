# HelloWorld App - Android编译指南

## 项目介绍

这是一个在Acode和Termux环境中编译的Android HelloWorld示例项目。本指南详细介绍了如何在Android设备上完全使用命令行工具构建APK文件，无需PC或Android Studio。

## 目录结构

```
HelloWorldApp/
├── app/
│   ├── build.gradle          # 应用模块构建配置
│   ├── src/
│   │   ├── main/
│   │   │   ├── AndroidManifest.xml  # 应用清单文件
│   │   │   ├── java/com/example/helloworld/
│   │   │   │   └── MainActivity.java  # 主活动类
│   │   │   └── res/
│   │   │       ├── layout/
│   │   │       │   └── activity_main.xml  # 主界面布局
│   │   │       ├── values/
│   │   │       │   ├── colors.xml     # 颜色定义
│   │   │       │   ├── strings.xml    # 字符串资源
│   │   │       │   └── themes.xml     # 主题样式
│   │   │       └── mipmap-hdpi/
│   │   │           └── ic_launcher.png  # 应用图标
├── build.gradle              # 项目根构建配置
├── gradle.properties         # Gradle属性配置
├── settings.gradle           # 项目设置
└── README.md                 # 本文档
```

## 编译环境要求

### 基本要求
- Android 7.0+ 设备
- Acode代码编辑器（或其他文本编辑器）
- Termux终端模拟器
- 至少2GB可用存储空间
- 稳定的网络连接（用于下载SDK和依赖）

### Termux环境准备

#### 1. 更新Termux包
```bash
pkg update && pkg upgrade -y
```

#### 2. 安装必要工具
```bash
pkg install -y openjdk-17 wget git unzip
```

#### 3. 安装Android SDK
```bash
# 创建SDK目录
mkdir -p ~/android-sdk/cmdline-tools

# 下载命令行工具
cd ~/android-sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip

# 解压并重命名
unzip commandlinetools-linux-9477386_latest.zip
mv cmdline-tools latest

# 设置目录结构
mkdir -p latest/bin
```

#### 4. 安装Gradle
```bash
# 下载Gradle
wget https://services.gradle.org/distributions/gradle-7.6-bin.zip

# 解压到/opt目录
unzip gradle-7.6-bin.zip -d /opt/

# 创建符号链接
ln -sf /opt/gradle-7.6/bin/gradle /data/data/com.termux/files/usr/bin/gradle
```

## 环境变量设置

### 临时设置（当前会话有效）
```bash
export ANDROID_HOME=$HOME/android-sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### 永久设置（推荐）
编辑Termux启动文件：`nano ~/.bashrc`

在文件末尾添加：
```bash
# Android SDK
export ANDROID_HOME=$HOME/android-sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Gradle
export GRADLE_HOME=/opt/gradle-7.6
export PATH=$PATH:$GRADLE_HOME/bin

# Java
export JAVA_HOME=/data/data/com.termux/files/usr/lib/jvm/java-17-openjdk
export PATH=$PATH:$JAVA_HOME/bin
```

使配置生效：
```bash
source ~/.bashrc
```

## SDK组件安装

### 安装必要的SDK组件
```bash
# 查看可用组件
sdkmanager --list

# 安装必需组件
sdkmanager "platform-tools" "platforms;android-33" "build-tools;33.0.0"

# 接受所有许可证
sdkmanager --licenses
```

## 编译步骤

### 1. 检查项目配置
确保你的`build.gradle`文件配置正确：

**项目根build.gradle** (`/path/to/HelloWorldApp/build.gradle`):
```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.4.2'
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

**app模块build.gradle** (`/path/to/HelloWorldApp/app/build.gradle`):
```gradle
plugins {
    id 'com.android.application'
}

android {
    compileSdk 33
    
    defaultConfig {
        applicationId "com.example.helloworld"
        minSdk 21
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.8.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
}
```

### 2. 进入项目目录
```bash
cd /path/to/HelloWorldApp
```

### 3. 清理项目（可选）
```bash
gradle clean
```

### 4. 构建项目

#### 构建Debug版本（推荐）
```bash
gradle assembleDebug
```

#### 构建Release版本
```bash
gradle assembleRelease
```

#### 构建所有版本
```bash
gradle build
```

### 5. 查看构建过程
构建过程会显示：
- 依赖下载（首次构建需要时间）
- 资源编译
- Java代码编译
- DEX文件生成
- APK打包
- 签名

## 生成的APK位置

构建完成后，APK文件位于：

### Debug版本
```
app/build/outputs/apk/debug/app-debug.apk
```

### Release版本
```
app/build/outputs/apk/release/app-release-unsigned.apk
```

### 快速查找命令
```bash
find . -name "*.apk" -type f
```

## 安装和运行

### 方法1：使用ADB安装（推荐）
```bash
# 确保设备已连接到同一网络
adb devices

# 通过USB或网络安装
adb install app/build/outputs/apk/debug/app-debug.apk

# 如果通过WiFi安装
adb connect 192.168.1.xxx:5555
adb install app/build/outputs/apk/debug/app-debug.apk
```

### 方法2：手动安装
1. 在文件管理器中找到生成的APK文件
2. 点击APK文件
3. 允许安装未知来源应用
4. 按照提示完成安装

### 运行应用
安装完成后，在应用列表中找到"HelloWorld"并点击运行。

## 常见问题解决

### 1. Java版本问题
**错误**：`Unsupported class file major version`

**解决**：
```bash
# 确认Java版本
java -version

# 应该是openjdk 17
# 如果不是，重新安装
pkg install openjdk-17 -y
```

### 2. 内存不足
**错误**：`OutOfMemoryError`

**解决**：
编辑`gradle.properties`：
```properties
org.gradle.jvmargs=-Xmx2048m -XX:MaxMetaspaceSize=512m
```

### 3. 网络问题
**错误**：连接超时或下载失败

**解决**：
```bash
# 使用国内镜像（清华大学）
echo "systemProp.http.proxyHost=mirrors.tuna.tsinghua.edu.cn"
echo "systemProp.http.proxyPort=80"

# 或者在build.gradle中添加镜像源
repositories {
    maven { url 'https://mirrors.cloud.tencent.com/nexus/repository/maven-public/' }
    google()
    mavenCentral()
}
```

### 4. 权限问题
**错误**：`Permission denied`

**解决**：
```bash
# 给Termux存储权限
termux-setup-storage

# 检查文件权限
chmod +x gradlew
```

### 5. SDK路径问题
**错误**：`Android SDK not found`

**解决**：
```bash
# 检查环境变量
echo $ANDROID_HOME

# 如果为空，重新设置
export ANDROID_HOME=$HOME/android-sdk
```

### 6. ADB连接问题
**错误**：`device not found`

**解决**：
```bash
# 开启开发者选项
# 1. 设置 > 关于手机 > 连续点击"版本号"7次
# 2. 返回设置 > 开发者选项 > 开启USB调试

# 如果通过WiFi连接
adb tcpip 5555
adb connect YOUR_DEVICE_IP:5555
```

### 7. 构建缓存问题
```bash
# 清理Gradle缓存
gradle clean
rm -rf ~/.gradle/caches/
```

### 8. 签名问题（Release版本）
如果需要签名Release版本：
```bash
# 生成密钥库
keytool -genkey -v -keystore my-release-key.keystore -alias alias_name

# 签名APK
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 -keystore my-release-key.keystore app-release-unsigned.apk alias_name

# 对齐优化
zipalign -v 4 app-release-unsigned.apk app-release.apk
```

## 优化建议

### 1. 使用Gradle Wrapper（推荐）
项目根目录下创建`gradle/wrapper/gradle-wrapper.properties`：
```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-7.6-bin.zip
networkTimeout=10000
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

然后使用`./gradlew`代替`gradle`命令。

### 2. 并行构建
在`gradle.properties`中添加：
```properties
org.gradle.parallel=true
org.gradle.configureondemand=true
```

### 3. 增量构建
使用`gradle assembleDebug --continuous`进行持续构建，自动检测代码变化。

## 资源

- [Android开发者文档](https://developer.android.com/docs)
- [Gradle用户指南](https://docs.gradle.org/current/userguide/userguide.html)
- [Termux Wiki](https://wiki.termux.com/wiki/Main_Page)

## 许可证

本项目仅用于学习目的。

---

**注意**：在Android设备上编译APK可能需要较长时间，建议在设备充电时进行，并确保有足够的存储空间。

最后更新：2024年
# HelloWorldApp
# HelloWorldApp
# HelloWorldApp
# HelloWorldApp
