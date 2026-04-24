# App Development Design Spec

## 技能优先顺序

- 通用 App 架构先遵守 `skills/app-spec/SKILL.md`
- App HTTP API 设计遵守 `skills/app-api/SKILL.md`
- iOS 平台能力遵守 `skills/app-ios/SKILL.md`
- Android 平台能力遵守 `skills/app-android/SKILL.md`
- 若本文与 skill 冲突，以更具体的 skill 为准

---

## 核心规则清单

- 分层边界：Widget 只负责 UI，业务流程走 Provider/UseCase，数据访问走 Repository。
- 数据隔离：API DTO 不直接进入 UI，必须映射为 Entity/ViewModel。
- 状态明确：区分 server/client/derived/persistent state，禁止重复存储派生状态。
- 依赖单向：上层依赖抽象，底层实现细节不得反向依赖 UI/业务页面。
- 动画边界：复杂互动动画使用 Rive；Rive 仅限 `presentation/` 层使用。
- 错误完整：异步流程必须覆盖 loading/success/empty/error/retry。
- 写入幂等：提交、支付、删除、点赞等写操作必须防重复和支持安全重试。
- 扩展集中：支付方式、登录方式、角色、渠道、主题等变化点集中管理。
- 可观测性：登录、支付、提交、同步、权限失败、接口错误必须可追踪。
- 简单演进：不过度设计，但不能破坏边界；抽象必须服务于重复、变化或隔离。

---

## 技术栈选型表

| Layer            | Choice                          |
|------------------|---------------------------------|
| 跨平台框架        | Flutter                         |
| 语言              | Dart                            |
| 状态管理          | Riverpod                        |
| HTTP             | Dio + interceptors              |
| 路由              | GoRouter                        |
| 不可变模型        | freezed + json_serializable     |
| 权限              | permission_handler              |
| 互动动画          | Rive（rive）                    |

---

## 目录结构规范

```
lib/
├── modules/                  # 业务模块（按功能划分）
│   ├── splash/               # 启动屏：版本展示、广告、更新检查
│   ├── login/
│   └── home/
│       ├── chat/
│       └── player/
├── common/
│   ├── widgets/              # 纯 UI 组件（无业务逻辑）
│   ├── providers/            # 跨模块共享 Riverpod providers
│   ├── http/                 # Dio 单例 + 拦截器
│   ├── analytics/            # Analytics adapter 接口 + 实现
│   └── permissions/          # permission_handler 统一封装
├── router/                   # GoRouter 配置
└── main.dart

# 每个 modules/xxx/ 内部三层结构：
modules/login/
├── presentation/             # UI 层：Widget + AsyncNotifier
│   ├── login_page.dart
│   └── login_provider.dart
├── domain/                   # 领域层：Repository 接口 + 业务实体
│   ├── login_repository.dart # 抽象接口
│   └── user_entity.dart      # freezed 实体
└── data/                     # 数据层：Repository 实现 + API + DTO
    ├── login_repository_impl.dart
    ├── login_api.dart
    └── login_dto.dart        # json_serializable DTO
```

**Rules:**
- 每个 `modules/xxx/` 包含：`presentation/`、`domain/`、`data/` 三层
- 资源文件（图片）放模块内 `assets/` 目录
- 模块内 Rive 资源放 `assets/rive/`
- 跨模块共享 Rive 资源放 `common/assets/rive/`
- `common/widgets/` 必须是纯 UI — 无 API 调用，无业务 provider 依赖

---

## 命名约定规范

| 目标               | 规范                        | 示例                         |
|--------------------|-----------------------------|------------------------------|
| Widget / Class     | PascalCase                  | `LoginPage`, `UserCard`      |
| 文件名              | snake_case                  | `login_page.dart`            |
| 函数 / 变量         | camelCase                   | `getUserInfo`, `isLoading`   |
| 私有成员            | `_` 前缀                    | `_controller`, `_timer`      |
| 常量               | lowerCamelCase              | `maxRetryCount`              |
| Provider           | camelCase + Provider 后缀   | `loginProvider`              |
| Repository 接口     | PascalCase + Repository     | `LoginRepository`            |
| DTO                | PascalCase + Dto            | `LoginDto`                   |
| 图片文件            | `module_feature_desc.png`   | `uc_user_icon.png`           |
| CSS class          | 不适用（Flutter 无 CSS）    | —                            |

- 使用完整单词；通用缩写除外（msg, init, img, nav, btn, bg）
- 禁止单字母变量（循环索引除外）

---

## 代码质量约束

- Single file: max **800 lines**
- Single method/function: max **80 lines**
- Methods between blank lines: **1 blank line**
- Code blocks separated by: **1 blank line**
- Delete unused: resources, methods, commented-out code, meaningless comments
- Add comments at: class properties, complex methods, large code blocks

---

## 模块组织规范

Organize by **business function**, not by file type.

```
# Good
modules/
  login/
    presentation/
    domain/
    data/
    assets/

# Bad
widgets/
  login_form.dart
pages/
  login.dart
```

- Shared code/assets across modules → `common/`
- A module that grows into a reusable library → extract to `packages/`

---

## 组件结构规范

Group methods by function block in this order:

**StatelessWidget:**
```
1. 构造函数（const 优先）
2. build()
```

**ConsumerStatefulWidget（Riverpod）:**
```
1. 私有 State 变量（_xxx）
2. initState()
3. didChangeDependencies()
4. didUpdateWidget()
5. dispose()
6. 事件处理方法（handle*）
7. 异步/API 调用（通过 ref.read(provider.notifier).method()）
8. 辅助 build 方法（_buildXxx）
9. build()
```

**AsyncNotifier（Provider 层）:**
```
1. build()          // 初始数据加载
2. 业务方法         // 返回 AsyncValue 或抛出异常
3. 私有辅助方法
```

**Repository 实现:**
```
1. 构造函数 + 依赖注入（Dio、本地存储）
2. 接口方法实现
3. 私有数据映射方法（DTO → Entity）
```

- 生命周期方法按**实际执行顺序**排列
- 事件处理方法统一以 `handle` 前缀命名
- Rive 初始化放 `onInit`，复杂控制逻辑不写在 `build()` 中
- build 方法保持精简 — Widget 树超过 ~50 行则提取子 Widget

---

## 状态管理规范

- 所有 Provider 定义在模块 `presentation/xxx_provider.dart`
- 使用 AsyncNotifier 管理异步状态，UI 通过 AsyncValue 处理三态：
  - `data`：正常渲染
  - `loading`：骨架屏 / 加载指示器
  - `error`：错误提示 + 重试入口
- Provider 作用域：模块内 provider 不得被其他模块直接引用
  → 跨模块共享状态放 `common/providers/`
- 禁止在 Widget 内直接调用 API — 必须经由 Notifier 方法
- Rive 动画状态由 Widget 或 Provider/Notifier 驱动，禁止传入 `domain/data`
- 使用 `@riverpod` 注解生成 provider，禁止手动 new Provider

---

## Rive 动画接入规范

- 统一使用官方 `rive` 包
- 简单过渡动画优先 Flutter 原生动画；复杂互动动画再使用 Rive
- 优先使用 `RiveAnimation.asset(...)` 加载 `.riv` 文件
- 互动场景优先使用 State Machine
- `.riv` 文件名使用 `snake_case`
- Widget 负责展示与事件绑定；Provider/Notifier 负责动画状态
- 禁止 `domain/`、`data/`、Repository、DTO、Entity 依赖 Rive 类型

---

## 错误处理规范

统一错误类型：

```dart
class AppError {
  final String code;
  final String message;
  final Object? cause;
  const AppError({required this.code, required this.message, this.cause});
}
```

- **Repository 层**：捕获 DioException，转换为 AppError 抛出
- **Notifier 层**：通过 AsyncValue.error 向上传递，不做额外处理
- **UI 层**：统一 ErrorWidget 展示，提供重试按钮
- **全局**：`runZonedGuarded` 包裹 `runApp`，捕获未处理异常上报
- 网络错误由全局 Dio 拦截器统一 toast 提示，业务层不重复处理

---

## 网络请求规范

- 全局单例 Dio，通过 Riverpod Provider 注入（禁止模块内 `new Dio()`）
- Request 拦截器：注入 token → 添加公共 headers（appId、channelId）
- Response 拦截器：
  → 解析业务错误码 → 抛出 AppError
  → 401 → 清除 token → 跳转登录
  → 网络超时/断网 → 全局 toast
- 超时配置：connectTimeout 10s，receiveTimeout 30s
- 所有 API 调用通过 Repository 层封装，Widget 不直接使用 Dio

---

## 代码生成规范

- 实体/DTO：使用 `freezed` + `json_serializable`
- Provider：使用 `@riverpod` 注解（riverpod_generator）
- 运行生成命令：
  ```bash
  dart run build_runner build --delete-conflicting-outputs
  ```
- 生成文件（`*.g.dart`、`*.freezed.dart`）**提交到 git**，避免 CI 依赖生成步骤

---

## 静态检查规范

继承 `flutter_lints`，额外启用：

```yaml
linter:
  rules:
    - always_use_package_imports    # 禁止跨模块相对路径引用
    - prefer_const_constructors
    - avoid_print                   # 使用 logger 替代 print
    - require_trailing_commas       # 保持 diff 整洁
```

---

## 埋点分析规范

```dart
Analytics.init(appId, userId, channelId);       // 应用启动时调用
Analytics.sendEvent(eventId, attr);             // 用户行为事件
Analytics.sendPageBegin(eventId, attr);         // 页面进入
Analytics.sendPageEnd(eventId, attr);           // 页面离开
```

- Offline queue: cache events locally when network unavailable, batch-upload on reconnect
- Backend provides both single-event and batch-event endpoints
- Underlying implementation must be swappable (adapter pattern)

**Required events (minimum):**

| Event | Priority |
|-------|----------|
| App launch | High |
| User registered | High |
| Payment started | High |
| Payment completed | High |
| User logged in | Medium |
| Promotion clicked | Medium |
| App exit | Low |
| Page enter/leave | Low |

---

## 权限处理规范

使用 `permission_handler` 包，统一封装在 `common/permissions/`。

Handle three states for every permission request:

1. **Never asked** → request permission
2. **Denied** → show dialog explaining why → link to system settings
3. **Granted** → proceed

Permissions to cover: camera, microphone, location, storage, network.

---

## 必备功能清单

| Feature | Priority | Notes |
|---------|----------|-------|
| Version check on launch | High | API call on splash, dialog to update |
| Push notifications | High | See `skills/app-ios/SKILL.md` and `skills/app-android/SKILL.md` |
| Splash version label | Medium | Display current app version |
| Splash ad | Low | Configurable via API |

---

## 测试覆盖清单

Every feature must cover:

- **Network**: normal, offline, slow, no permission
- **Permissions**: first request, denied, re-request after denial
- **Memory**: repeated navigation, repeated data refresh, image/video loops
- **Background/foreground**: repeated switching, long background → resume
- **Input**: max length, overflow display (ellipsis), special characters
- **Share**: all platforms, link tap, QR code scan
- **Rive**: asset missing fallback, state machine trigger, repeated enter/exit, background resume

---

## 分支管理规范

```
master          production only, never commit directly
  └── hotfix/*  cut from master, merge back to master + develop
  └── develop   integration branch, never commit directly
        └── function/*  feature branches, merge to develop when done
```

- Developer works on `function/your-name-feature` or personal branch
- Develop → Master only after QA sign-off
