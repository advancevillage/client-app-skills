# app-ios

iOS-specific constraints for Flutter projects.

---

## Push Notifications (APNs)

- 使用 `firebase_messaging` 插件（负责 APNs 桥接）
- Xcode Capabilities 启用：`Push Notifications` + `Background Modes → Remote notifications`
- `AppDelegate.swift` 中初始化 Firebase：`FirebaseApp.configure()`
- device token 每次启动上报后端（token 可能轮换）
- 前台收到消息：`FirebaseMessaging.onMessage` 监听处理
- 后台/终止状态：`onBackgroundMessage` 静态处理函数（独立 isolate，不可访问 UI）
- 本地通知展示配合 `flutter_local_notifications` 使用

---

## Version Update

- 使用 `package_info_plus` 获取当前版本：
  ```dart
  final info = await PackageInfo.fromPlatform();
  // info.version → CFBundleShortVersionString
  ```
- 启动时与 API 返回版本比对
- 强制更新：跳转 App Store URL，阻断 UI（`PopScope` 拦截返回手势）
- 软更新：可关闭 Dialog
- App Store 链接格式：`https://apps.apple.com/app/id{APP_ID}`

---

## Widget 结构规范（Green Frame）

Flutter 文件内按以下顺序组织（与 CLAUDE.md 通用规范一致）：

1. import（`dart:` → `package:` → 相对路径，各组空一行）
2. 文件顶层常量
3. Widget 类
   - 私有 State 变量（`_xxx`）
   - `initState()` / `dispose()`
   - `handle*` 事件方法
   - `_buildXxx` 辅助构建方法
   - `build()`

**Platform Channel（仅需调用原生功能时使用）：**
- Swift 桥接代码放 `ios/Runner/Channels/`
- Channel 命名格式：`com.yourapp.module/feature`
- 每个 Channel 对应独立 Swift 文件，在 `AppDelegate.swift` 中注册

---

## Module / Asset Organization

**Flutter 层（业务代码）：**

`lib/modules/` 按功能组织，见 CLAUDE.md 目录规范。

**iOS 原生层（仅桥接）：**

```
ios/Runner/
├── AppDelegate.swift          # Firebase 初始化、推送注册、Channel 注册
├── GeneratedPluginRegistrant  # 自动生成，勿修改
└── Channels/                  # 自定义 MethodChannel Swift 实现
```

**资源文件：**
- 图片等资源放 `assets/`，在 `pubspec.yaml` 中声明
- 不在 `ios/` 下放业务图片
- App 图标使用 `flutter_launcher_icons` 生成

---

## iOS 配置 (`ios/Runner/Info.plist`)

权限描述（中文，符合 App Store CN 审核要求）：

| Key | 描述示例 |
|-----|---------|
| `NSCameraUsageDescription` | 用于拍摄头像和扫码 |
| `NSMicrophoneUsageDescription` | 用于语音通话和视频录制 |
| `NSLocationWhenInUseUsageDescription` | 用于附近功能定位 |
| `NSPhotoLibraryUsageDescription` | 用于选择和保存图片 |

最低 iOS 版本：`ios/Podfile` 中 `platform :ios, '13.0'`

---

## Build & Release

- 开发：`flutter run`
- Release 构建：`flutter build ios --release`
  → Xcode：Product → Archive → Distribute App → TestFlight
- 版本管理统一在 `pubspec.yaml`：
  ```yaml
  version: 1.0.0+1   # versionName+versionCode
  ```
  每次 TestFlight 上传前递增 `+` 后的数字（build number）
- 依赖同步：`flutter pub get`（无需手动 `pod install`，flutter 自动处理）
- Scheme：Debug 用于开发，Release 用于 Archive
