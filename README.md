# Flutter 开发笔记

---

## Lottie 预览

- 预览地址：https://lottiefiles.com/preview
- 代理规则：`DOMAIN-SUFFIX,lottiefiles.com,自主选择`

---

## Flutter Rive 集成

### 方式一：MCP 集成（实验性，官方不推荐）

> 参考文档：https://rive.app/docs/editor/mcp/integration

1. 安装 **Rive Early Access 桌面版**并保持运行
2. 添加 MCP：

```bash
# 方式 A（Codex）
codex mcp add rive --url http://localhost:9791/sse

# 方式 B（Claude）
claude mcp add --transport sse rive http://localhost:9791/sse
```

### 方式二：AI Agent（官方推荐）

> 参考文档：https://rive.app/docs/editor/ai-agent/ai-agent
>
> 价格：**$17/月**

### 方式三：RiveMCP 第三方服务

> 地址：https://rivemcp.stunning.gg
>
> 价格：**$10/月**

### 参考资料

- Rive 中文文档：https://www.rive.org.cn/docs/guide/introduction

---

## App Rive Skill

新增 `skills/app-rive/SKILL.md`，用途：

- 判断 Pencil 设计稿中的图片组件是否适合转为 Rive
- 给 Rive 新手输出一步一步的 Rive Editor 操作指导
- 约束 `.riv` 资源在 Flutter App 中的接入边界

---

## Flutter 在 Mac 上运行 iPhone 真机

### 一、环境准备

```bash
# 确认 Flutter 环境正常
flutter doctor
```

确保 **Flutter** 和 **Xcode** 都是 ✅

---

### 二、Xcode 配置

```bash
# 安装 Xcode Command Line Tools
sudo xcode-select --install

# 接受 Xcode 协议
sudo xcodebuild -license accept
```

---

### 三、连接 iPhone

1. USB 线连接 iPhone 到 Mac
2. iPhone 弹出 **"信任此电脑？"** → 点击**信任** → 输入密码
3. 开启**开发者模式**：
   - 设置 → 隐私与安全性 → 开发者模式 → 开启 → 重启 → 确认

---

### 四、Xcode 签名配置

```bash
open ios/Runner.xcworkspace
```

在 Xcode 中：

1. 左侧点击 **Runner**
2. 选择 **Signing & Capabilities** tab
3. 勾选 **Automatically manage signing**
4. **Team** 选择你的 Apple 账号
   - 没有则登录：Xcode → Settings → Accounts → 添加 Apple ID
5. **Bundle Identifier** 改成唯一字符串，如 `com.yourname.appname`

---

### 五、运行

```bash
# 确认设备已识别
flutter devices

# 运行到 iPhone
flutter run
```

---

### 常见报错速查

| 报错 | 解决方案 |
|------|----------|
| 设备未显示 | 重插 USB，点信任，检查开发者模式 |
| Bundle ID 已被注册 | 改成唯一 Bundle ID |
| Xcode 版本不支持当前 iOS | 升级 Xcode 到最新版 |
| No provisioning profile | Xcode 中重新选择 Team |
| pod install 失败 | `cd ios && pod install && cd ..` |

---

### 附：快速检查命令

```bash
# 查看已连接设备
flutter devices

# 查看详细环境信息
flutter doctor -v

# 重新安装 CocoaPods 依赖
cd ios && pod install && cd ..

# 清理构建缓存
flutter clean && flutter run
```
