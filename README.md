# APK Universal Converter

将 Google Play 或第三方 APK 下载工具获取的 **APK / APKS / APKM / XAPK** 转换为一个可以直接安装的 **Universal APK**。

本项目使用 GitHub Actions 自动完成：

1. 获取输入 APK 包
2. 使用 [APKEditor](https://github.com/REAndroid/APKEditor) 合并 Split APK
3. 将必要的 ABI、语言、屏幕密度及其他配置合并到一个 APK
4. 重新签名
5. 将最终 APK 作为 GitHub Actions Artifact 提供下载

> **项目定位：**
>
> 本项目不负责从 Google Play 下载应用。
>
> 你可以使用自己喜欢的 Google Play 下载工具或第三方网站获取 `.apk`、`.apks`、`.apkm` 或 `.xapk`，然后交给本项目转换。

---

## ✨ Features

* ✅ 支持 `.apk`
* ✅ 支持 `.apks`
* ✅ 支持 `.apkm`
* ✅ 支持 `.xapk`
* ✅ 自动合并 Split APK
* ✅ 尽可能将 ABI、语言、屏幕密度等 Split 合并到单一 APK
* ✅ 生成单独的 `.apk` 文件
* ✅ 自动重新签名
* ✅ 无需在本地安装 Java、APKEditor、Android SDK
* ✅ 完全通过 GitHub Actions 运行
* ✅ 可以 Fork 本项目后直接使用
* ✅ 不需要在 GitHub Actions 中配置 Google Play 账号或其他第三方账号

---

# 📦 什么是 Universal APK？

很多 Android 应用现在通过 Android App Bundle（AAB）发布。

Google Play 不一定直接提供一个包含所有资源的单一 APK，而是根据设备生成多个 Split APK，例如：

```text
base.apk
config.arm64_v8a.apk
config.armeabi_v7a.apk
config.xxhdpi.apk
config.zh.apk
config.en.apk
...
```

或者下载工具会将这些 Split APK 打包成：

```text
xxx.apks
xxx.apkm
xxx.xapk
```

普通情况下，需要安装这些 Split APK 才能完整安装应用。

本项目的目标是将这些 Split APK 合并成：

```text
Universal APK
```

即最终得到：

```text
xxx-universal.apk
```

理论上不再需要同时安装一堆 Split APK。

---

# ⚠️ Universal APK 的含义

本项目生成的 Universal APK 是一个**实际的单一 APK 文件**，而不是简单提取：

```text
base.apk
```

对于包含 ABI Split 的应用，例如：

```text
arm64-v8a
armeabi-v7a
```

APKEditor 会尝试将相关 Split 合并到最终 APK。

因此最终 APK 可以包含多个 ABI 的 native libraries，例如：

```text
lib/
├── arm64-v8a/
└── armeabi-v7a/
```

同时也会尝试合并其他必要的资源 Split，例如：

```text
语言
屏幕密度
地区资源
其他 configuration split
```

因此最终得到的是一个真正的 standalone APK，而不是单纯的 `base.apk`。

> 注意：最终 APK 的具体内容取决于输入包本身包含哪些 Split。
>
> 如果原始下载包本身没有某个 ABI 或资源，转换工具无法凭空生成该资源。

---

# 🚀 快速开始

## 1. Fork 本项目

首先点击 GitHub 页面右上角：

**Fork**

将本项目 Fork 到自己的 GitHub 账号。

Fork 后，你会拥有自己的仓库，例如：

```text
https://github.com/你的用户名/apk-universal-converter
```

---

# 2. 准备 APK 包

使用你喜欢的 Google Play APK 下载工具获取应用。

本项目不限制具体下载来源。

可以是：

```text
xxx.apk
xxx.apks
xxx.apkm
xxx.xapk
```

例如：

```text
Browser_s1.0.111_apkcube.apks
```

---

# 3. 创建 GitHub Release

进入你 Fork 后的仓库：

```text
Releases
```

创建一个新的 Release。

建议 Tag 使用：

```text
input
```

例如：

```text
Tag:
input
```

然后将准备好的 APK 文件上传到这个 Release 的 **Assets** 区域。

例如：

```text
input
└── Browser_s1.0.111_apkcube.apks
```

### 为什么要使用 Release Asset？

GitHub Actions 的 `workflow_dispatch` 本身没有直接上传 APK 文件作为运行参数的功能。

因此本项目使用 GitHub Release Asset 作为输入文件暂存位置。

这样做还有一个好处：

* APK 不需要提交到 Git 仓库
* 不会污染 Git 历史
* 大文件更适合放在 Release Asset
* Actions 可以通过 Release Tag + 文件名获取输入

---

# 4. 打开 Actions

进入：

```text
Actions
```

找到：

```text
Convert to Universal APK
```

点击：

```text
Run workflow
```

---

# 5. 填写参数

会看到两个输入框。

## Release tag

如果你按照上面的方式创建：

```text
input
```

就填写：

```text
input
```

默认值也是：

```text
input
```

所以通常不需要修改。

---

## Asset filename

填写你上传到 Release 的**完整文件名**。

例如：

```text
Browser_s1.0.111_apkcube.apks
```

注意必须和 Release Asset 的文件名完全一致。

例如：

```text
Browser_s1.0.111_apkcube.apks
```

和：

```text
browser_s1.0.111_apkcube.apks
```

不是同一个文件名。

---

# 6. 运行 Action

填写完成后点击：

```text
Run workflow
```

GitHub Actions 会依次执行：

```text
Download APKEditor
        ↓
Download input package
        ↓
Validate input
        ↓
Merge APKs
        ↓
Create temporary signing key
        ↓
Sign Universal APK
        ↓
Verify APK
        ↓
Upload Universal APK
```

---

# 7. 下载最终 APK

Action 成功后进入对应的 Workflow Run。

页面底部会看到：

```text
Artifacts
```

其中：

```text
universal-apk
```

就是最终生成的 Universal APK。

下载后解压即可得到：

```text
universal.apk
```

将这个 APK 传输到 Android 设备即可安装。

---

# 🧩 支持的输入格式

| 格式      | 支持 | 说明                           |
| ------- | -- | ---------------------------- |
| `.apk`  | ✅  | 单 APK                        |
| `.apks` | ✅  | APK Set / Split APK 容器       |
| `.apkm` | ✅  | APKM Split APK 容器            |
| `.xapk` | ✅  | XAPK 容器                      |
| `.aab`  | ❌  | 本项目不是 AAB → Universal APK 工具 |

其中：

```text
.apks
.apkm
.xapk
```

通常包含多个 Split APK。

本项目通过 APKEditor 对这些 Split APK 进行合并。

---

# 🔧 技术原理

核心转换由：

**APKEditor**

完成。

官方项目：

https://github.com/REAndroid/APKEditor

APKEditor 提供：

```text
merge
```

功能，可以处理：

```text
XAPK
APKM
APKS
Split APK directory
```

例如：

```bash
java -jar APKEditor.jar m -i input.apks -o output.apk
```

本项目在此基础上进一步：

```text
Input
  │
  ├── .apk
  ├── .apks
  ├── .apkm
  └── .xapk
       │
       ▼
   APKEditor
       │
       ▼
  merged.apk
       │
       ▼
  temporary keystore
       │
       ▼
    apksigner
       │
       ▼
  universal.apk
```

---

# 🔐 关于重新签名

合并 APK 后，原始 APK 的签名通常无法继续直接使用。

因此 GitHub Actions 会自动生成一个临时 keystore，并使用 Android `apksigner` 对最终 APK 重新签名。

生成的签名信息仅用于本次转换。

例如：

```text
Keystore:
universal.keystore

Alias:
universal
```

签名密码由 Workflow 内部临时使用。

### 重要

重新签名意味着：

**转换后的 APK 与原 Google Play APK 的签名不同。**

因此，如果设备已经安装了同一个应用的官方版本，可能无法直接覆盖安装。

通常需要：

```text
卸载原版本
        ↓
安装 Universal APK
```

或者使用与原应用相同签名的 APK 进行升级。

---

# ⚠️ 重要注意事项

## 1. Universal APK 可能非常大

Universal APK 包含多个 ABI 和资源，因此通常明显大于针对单一设备生成的 APK。

例如一个应用可能原本分别提供：

```text
arm64-v8a
armeabi-v7a
x86
x86_64
```

合并之后文件体积会增加。

这是 Universal APK 的正常特性。

---

## 2. 不保证所有应用都能成功转换

Android 应用的 Split 结构并不完全相同。

某些应用可能包含：

* Dynamic Feature
* 特殊 Split
* DRM
* Google Play Licensing
* 特殊安装逻辑
* App Bundle 特殊配置
* 依赖 Google Play Services 的功能

这些情况下，即使 APK 成功合并，也不代表所有功能都一定正常。

---

## 3. APK 成功生成 ≠ 应用一定可以正常运行

本项目主要解决：

```text
Split APK
        ↓
Single APK
```

而不是保证应用本身可以脱离 Google Play 正常运行。

例如应用可能在运行时检查：

```text
Google Play Services
Google Play Licensing
Play Integrity
设备认证
服务器授权
```

这些问题不属于 APK 合并本身。

---

## 4. 输入包必须尽可能完整

如果下载工具只提供：

```text
base.apk
```

那么本项目无法凭空恢复其他 Split。

如果希望得到完整 Universal APK，建议使用包含完整 Split 的下载包。

例如：

```text
xxx.apks
xxx.apkm
xxx.xapk
```

---

# 📁 Repository Structure

项目结构非常简单：

```text
apk-universal-converter/
│
├── .github/
│   └── workflows/
│       └── convert.yml
│
└── README.md
```

核心逻辑全部位于：

```text
.github/workflows/convert.yml
```

因此 Fork 后通常不需要安装任何本地软件。

---

# ⚙️ GitHub Actions 配置

Workflow 使用：

```yaml
workflow_dispatch
```

因此可以手动运行。

主要参数：

```text
asset_name
release_tag
```

例如：

```text
release_tag:
input

asset_name:
Browser_s1.0.111_apkcube.apks
```

---

# 🛠️ 使用的主要工具

## APKEditor

用于：

```text
Split APK → Single APK
```

项目：

https://github.com/REAndroid/APKEditor

本项目当前使用：

```text
APKEditor 1.4.9
```

---

## Java

GitHub Actions 使用：

```text
Java 17
```

---

## Android apksigner

用于对最终 APK 进行 Android APK 签名和验证。

---

# 🔒 隐私

本项目本身不会：

* 上传 APK 到第三方服务器
* 保存 APK
* 分析 APK 内容
* 使用 Google Play 账号
* 要求 Google Play 登录

APK 处理全部发生在 GitHub Actions Runner 中。

不过需要注意：

**GitHub Actions 的运行环境属于 GitHub。**

如果处理的是具有隐私、商业机密或其他敏感内容的 APK，请根据自己的安全需求决定是否使用 GitHub Actions。

---

# 📜 License

本项目的 Workflow 脚本和 README 可以根据本仓库实际 License 使用。

本项目本身不包含 APKEditor 的源码。

APKEditor 的许可证及相关版权信息请参见：

https://github.com/REAndroid/APKEditor

---

# ❓ FAQ

### Q: 我必须使用 Release tag `input` 吗？

不必须。

例如可以创建：

```text
release-001
```

然后在 Run workflow 时：

```text
release_tag:
release-001
```

即可。

---

### Q: 文件一定要叫 `Browser_s1.0.111_apkcube.apks` 吗？

不需要。

任何合法文件名都可以。

例如：

```text
Telegram.apks
YouTube.apkm
Example.xapk
MyApp.apk
```

只要在 `asset_name` 中填写完全相同的文件名即可。

---

### Q: 能不能一次转换多个 APK？

当前 Workflow 一次处理一个 Release Asset。

如果需要转换多个应用，可以分别运行多次 Workflow。

---

### Q: 为什么不用 bundletool？

`bundletool` 更主要用于：

```text
AAB → APK Set / Universal APK
```

而本项目的输入已经是：

```text
APK
APKS
APKM
XAPK
```

即已经存在 Split APK。

因此这里直接使用 APKEditor 的 Merge 功能更加直接。

---

### Q: 最终 APK 是不是只有 `base.apk`？

不是。

本项目的目的不是提取：

```text
base.apk
```

而是通过 APKEditor 将输入包中的 Split APK 合并为一个 standalone APK。

---

### Q: 为什么最终 APK 的签名和原版不同？

因为合并 APK 后需要重新签名。

本项目会自动生成临时签名密钥并重新签名。

因此：

```text
官方 APK
```

和：

```text
本项目生成的 APK
```

签名不同。

---

# ⭐ 推荐使用流程

最简单的完整流程：

```text
Google Play / 第三方下载工具
              │
              ▼
      Browser_xxx.apks
              │
              ▼
       GitHub Release
              │
              ▼
        Run workflow
              │
              ▼
          APKEditor
              │
              ▼
       Merge Split APK
              │
              ▼
          apksigner
              │
              ▼
        universal.apk
              │
              ▼
          Android
```

---

## 💡 Why this project?

很多下载工具可以很好地获取 Google Play 的 Split APK，但最终得到的往往是：

```text
.apks
.apkm
.xapk
```

而部分 Android 设备、文件管理器、第三方安装环境更适合直接使用：

```text
.apk
```

本项目就是提供一个简单的：

```text
Split APK → Universal APK
```

GitHub Actions 工作流。

**Fork → 上传 Release Asset → Run workflow → 下载 APK。**
