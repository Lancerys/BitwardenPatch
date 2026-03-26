# BitwardenPatch

## 解锁 Bitwarden 官方客户端 TOTP 功能

本指南/项目旨在解锁 Bitwarden 官方客户端的 **TOTP（双重身份验证令牌）** 功能，支持以下平台：

- **浏览器扩展** (Browser Extension)
- **安卓端** (Android)

**提示：** 本项目仅供学习与技术交流使用。

---

一个基于 Bitwarden 上游项目的自动化构建仓库。

本仓库不直接维护完整源码，而是通过 GitHub Actions 在构建时拉取上游官方仓库、应用本地 patch，并自动生成可用产物。

目前包含两套工作流：

- Android 客户端自动构建
- Browser 扩展自动构建

## 项目思路

本仓库主要保存两类内容：

- GitHub Actions workflow
- 自定义 patch

构建流程不是直接在本仓库里编译现成源码，而是：

1. 在 GitHub Actions 中拉取 Bitwarden 官方上游仓库
2. 获取对应最新 tag
3. 应用本仓库中的 patch
4. 执行上游原有构建流程
5. 将构建产物上传到 GitHub Releases

这样做的好处是：

- 不需要长期同步完整上游源码
- 更容易跟进上游新版本
- patch 和构建逻辑分离，更方便维护

## 工作流说明

### Android 工作流

Android 工作流会自动：

- 拉取 `bitwarden/android`
- 获取最新 Android 对应 tag
- 应用 `patches/android/hasPremium.patch`
- 配置 JDK、Ruby、Gradle、Fastlane
- 使用签名配置生成 release APK
- 上传到 GitHub Releases

对应文件：

```text
.github/workflows/build-android.yml
````

Android 构建依赖 GitHub Secrets 中的签名和 Firebase 配置，因此 fork 后如果要自行构建，需要先补齐相关 secrets。

### Browser 工作流

Browser 工作流会自动：

* 拉取 `bitwarden/clients`
* 获取最新 Browser 对应 tag
* 应用 `patches/browser/hasPremiumPersonally.patch`
* 构建浏览器扩展源码包
* 分别构建 Chrome / Edge / Firefox / Opera 版本
* 上传 artifact 和 GitHub Releases

对应文件：

```text
.github/workflows/build-browser.yml
```

Browser 工作流不依赖 Android 那套签名配置，fork 后可直接运行。

## Patch 说明

当前 patch 主要用于修改客户端本地的 Premium 状态判断逻辑，从而解锁 TOTP 等依赖 Premium 状态的本地功能。

仓库中的 patch 只用于本地状态处理和自动构建，不直接托管上游完整源码。

## 可自行 fork 构建

本项目支持自行 fork 后独立构建。

如果你不想直接使用当前仓库发布的 release，或者你想自己控制构建流程、签名文件和发布内容，建议直接 fork 本仓库后自行运行 GitHub Actions。

### fork 构建的基本步骤

1. fork 本仓库到你自己的 GitHub 账号
2. 在你自己的仓库中启用 GitHub Actions
3. 根据需要配置 Android 构建所需 secrets
4. 手动运行 workflow，或通过 push / schedule 自动构建
5. 在你自己的 Releases 页面获取产物

### Android fork 构建需要的 Secrets

Android workflow 需要以下 secrets：

* `SIGN_KEYSTORE_BASE64`
* `GOOGLE_SERVICES_JSON`
* `KEYSTORE_PASSWORD`
* `KEY_PASSWORD`

这些内容都需要配置在你自己 fork 后的仓库中，原仓库的 secrets 不会自动继承到 fork。

### Browser fork 构建

Browser workflow 通常不需要 Android 那些签名 secrets，配置成本更低。
如果只是想自己构建浏览器扩展，fork 后直接运行对应 workflow 即可。

## 为什么建议自行 fork 构建

自行 fork 构建有几个好处：

* 可以自己审查 workflow 和 patch
* 可以自己控制签名材料和 secrets
* 可以确认 release 产物来自你自己的 workflow
* 后续更新时更方便按自己的需求调整

如果你对 release 产物的来源、构建环境或发布方式有顾虑，自行 fork 构建会更合适。

## 仓库结构

```text
.github/workflows/
  build-android.yml
  build-browser.yml

patches/
  android/
    hasPremium.patch
  browser/
    hasPremiumPersonally.patch
```

## 说明

本仓库的定位是：

* 自动跟踪上游版本
* 应用少量 patch
* 自动构建发布

更适合作为自用构建仓库或 fork 后二次使用的模板仓库。

好用的话给个⭐⭐，蟹蟹

```
```
