# MovieMoment

收集来源于网络的一些资源。

跨平台 Flutter 应用，目标平台：

- iOS
- Android
- Windows
- macOS

当前通过 [FVM](https://fvm.app/) 锁定 Flutter **3.47.0**（stable）。

## 本地开发

```bash
# 按 .fvmrc 安装并启用本项目 Flutter 版本
fvm install
fvm use

# 依赖、分析、测试
fvm flutter pub get
fvm flutter analyze
fvm flutter test

# 运行
fvm flutter run
```

本地 Release 构建：

```bash
fvm flutter build apk --release
fvm flutter build appbundle --release
fvm flutter build ios --release --no-codesign
fvm flutter build macos --release
# Windows 需在 Windows 机器上：
fvm flutter build windows --release
```

VS Code / Cursor 会读取 `.vscode/settings.json`，自动使用 `.fvm/flutter_sdk`。

本地打 iOS 包需要 Xcode 已安装对应 **iOS 平台组件**（Xcode → Settings → Components）。当前机器若缺少 iOS SDK，本地 `flutter build ios` 会失败，GitHub Actions 的 `macos-latest` 自带 SDK，不受影响。

## GitHub 一次性打包四个平台

工作流：[`.github/workflows/release.yml`](.github/workflows/release.yml)

一次触发会并行构建：

| 平台 | Runner | 产物 |
| --- | --- | --- |
| Android | `ubuntu-latest` | `MovieMoment-<version>-android.apk`、`.aab` |
| iOS | `macos-latest` | `MovieMoment-<version>-ios-unsigned.ipa` |
| macOS | `macos-latest` | `MovieMoment-<version>-macos.zip` |
| Windows | `windows-latest` | `MovieMoment-<version>-windows.zip` |

构建前会先跑 `flutter analyze` 和 `flutter test`。CI 使用的 Flutter 版本来自 `.fvmrc`。

### 怎么触发

1. **手动一次打四个包**  
   GitHub → Actions → **Release** → **Run workflow**。默认会在四个平台都成功后创建一个 GitHub Release。
2. **打 tag 自动发版**

   ```bash
   git tag v1.0.0+1
   git push origin v1.0.0+1
   ```

产物会出现在该次 Workflow 的 Artifacts，以及对应的 GitHub Release 附件里。

### 签名说明

- **Android**：未配置密钥时，Release APK 使用 debug 签名，方便 CI 先跑通。若要上传商店，在仓库 Secrets 中配置：
  - `ANDROID_KEYSTORE_BASE64`：`.jks` / `.keystore` 的 base64
  - `ANDROID_KEYSTORE_PASSWORD`
  - `ANDROID_KEY_ALIAS`
  - `ANDROID_KEY_PASSWORD`
- **iOS**：CI 产出的是 **未签名 IPA**，不能直接上架 App Store，需要本机或后续接入 Apple 证书后再签名导出。
- **macOS**：CI 关闭代码签名，得到可分发的 `.app` 压缩包；正式分发还需公证（notarization）。
- **Windows**：输出 `Release` 目录下的可执行文件及依赖 DLL。
