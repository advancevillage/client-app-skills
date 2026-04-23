---
name: app-spec
description: "App 通用架构约束：分层边界、状态归属、依赖方向、错误处理、幂等重试、扩展性与可观测性。"
---

# App Spec Skill

Use this skill when designing, implementing, or reviewing App features. It defines cross-platform App architecture constraints for maintainability, extensibility, resilience, and observability. Platform-specific rules live in `app-ios` and `app-android`; App-facing API rules live in `app-api`.

---

## 一、适用场景说明

原则：
- 用于 Flutter App 的通用架构、模块边界、状态管理、错误处理和扩展性设计。
- 适用于新功能开发、旧功能重构、代码评审、VibeCoding 生成约束。

必须：
- 先应用本通用规范，再应用 `app-api`、`app-ios`、`app-android` 的专项规范。
- 当功能涉及网络、缓存、权限、支付、登录、同步、埋点时，必须检查本规范。

禁止：
- 只实现功能 happy path，不考虑失败、重试、状态归属和后续扩展。

---

## 二、核心设计目标

原则：
- App 代码目标是高内聚、低耦合、可维护、可扩展、可观测、可演进。
- 架构不是越复杂越好，而是用清晰边界控制复杂度。

必须：
- 保持职责单一：一个模块只负责一个方向的变化。
- 保持依赖可控：模块之间通过稳定且最小的接口通信。
- 保持演进能力：新增能力应尽量新增模块，而不是修改多处旧代码。

禁止：
- 为短期功能把 UI、业务规则、API、缓存、跳转、埋点混在一个文件里。
- 为没有明确变化压力的功能引入复杂抽象。

---

## 三、架构分层约束

原则：
- App 默认分为 UI、ViewModel/Provider、UseCase/Service、Repository、Infrastructure 五类职责。

必须：
- UI Component 只负责展示和交互事件派发。
- ViewModel/Provider 负责页面状态、加载状态、错误状态和交互状态。
- UseCase/Service 负责业务流程编排。
- Repository 负责数据读写抽象。
- API、Storage、SDK、Router、Analytics 通过 Adapter 或基础设施层封装。

禁止：
- Widget 直接调用 Dio、Storage、SDK 或复杂业务流程。
- Repository 反向依赖 UI、Provider 或具体页面。
- 公共 UI 组件依赖业务 API 或业务 Provider。

推荐：
- 页面调用方向保持为 `Page -> Provider/ViewModel -> UseCase/Service -> Repository -> API/Storage/SDK`。

---

## 四、数据模型约束

原则：
- API DTO、Domain Model、View Model、UI State 必须区分。

必须：
- API response 先映射为 Domain Model 或 View Model，再进入 UI。
- Mapper/Adapter 负责隔离后端字段、版本和语义变化。
- UI 只依赖自己需要的最小展示字段。

禁止：
- 后端 DTO 直接穿透到组件树。
- 组件依赖接口中的无关字段。
- 因接口字段变化导致多个页面同步修改。

推荐：
- DTO 命名使用 `XxxDto`。
- 业务实体命名表达领域含义，不表达接口来源。
- 页面展示模型只保留渲染需要的数据。

---

## 五、状态管理约束

原则：
- 每个状态必须明确归属，避免重复存储和隐式共享。

必须：
- 区分 Server State、Client State、Derived State、Persistent State。
- Server State 由 Repository 或数据缓存机制管理。
- Client State 由页面级 Provider/ViewModel 管理。
- Derived State 通过计算得到，不重复持久化。
- Persistent State 通过统一 Storage Adapter 管理。

禁止：
- 把所有状态塞进全局 Store。
- 多处维护同一个业务状态。
- 重复存储可计算的派生状态。
- 让子组件直接修改跨模块状态。

自检：
- 这个状态来自服务端、本地交互、计算结果，还是持久化存储？
- 这个状态是否已经能从其他状态推导出来？

---

## 六、依赖方向约束

原则：
- 高层依赖抽象，低层提供实现，依赖方向必须单向。

必须：
- 业务逻辑依赖 Repository 接口，不依赖具体 API 实现。
- 第三方 SDK、请求库、本地存储、路由、埋点都必须封装后使用。
- 跨模块协作通过明确接口、事件或共享抽象完成。

禁止：
- 下层模块 import 上层模块。
- Domain 层依赖 Flutter Widget、Dio、localStorage、平台 SDK。
- 模块之间读取彼此内部文件或内部状态。

推荐：
- 不稳定实现放在 data/infrastructure 层。
- 稳定业务接口放在 domain 层。

---

## 七、异步错误约束

原则：
- 网络、权限、支付、同步、推送、SDK 调用都默认可能失败。

必须：
- 所有异步流程至少处理 loading、success、empty、error、retry。
- 用户可感知操作失败时必须给出明确反馈。
- 可恢复错误必须提供重试入口。
- Repository 层统一转换底层异常为业务错误。
- UI 层只负责展示错误，不解析底层异常。

禁止：
- 只处理成功路径。
- `catch` 后静默吞掉错误。
- 在多个页面重复实现同一套错误解析。

推荐：
- 错误对象包含 code、message、cause、context。
- 慢网络、离线、无权限、服务端错误分别处理。

---

## 八、幂等重试约束

原则：
- 写操作必须假设用户会重复点击，网络会超时，响应会丢失，请求会重放。

必须：
- 提交、支付、删除、点赞、收藏、登录、发消息等写操作必须防重复提交。
- 关键写请求必须支持安全重试或明确禁止自动重试。
- 乐观更新必须具备失败回滚或补偿策略。
- 涉及订单、支付、资产、账号的操作必须以服务端最终状态为准。

禁止：
- 只依赖按钮 disabled 保证不重复提交。
- 超时后盲目重复发起非幂等请求。
- 乐观更新失败后不恢复 UI 状态。

推荐：
- 写请求携带 requestId、clientMutationId 或业务幂等键。
- 查询服务端状态确认关键操作结果。

---

## 九、扩展变化约束

原则：
- 变化点要集中管理，不能散落在页面和组件里。

必须：
- 登录方式、支付方式、角色权限、表单字段、主题、多语言、渠道差异、实验配置必须识别为扩展点。
- 扩展点优先使用策略、配置、Adapter、注册表或插件式组织。
- 新增类型时优先新增实现文件，而不是修改多处调用方。

禁止：
- 在多个页面散落相同业务 if/else。
- 把渠道、角色、类型判断硬编码进 UI。
- 为每个新增类型复制一套近似流程。

推荐：
- 使用 map/registry 替代长分支。
- 将变化维度写入命名和目录结构。

---

## 十、可观测性约束

原则：
- 看不见的系统无法调试，关键流程必须可追踪。

必须：
- 登录、注册、支付、提交、同步、权限失败、接口错误必须具备日志、埋点或错误上报。
- 错误上报必须包含页面、操作、接口、业务 ID、错误码等上下文。
- 埋点、日志、错误上报通过统一 Adapter 调用。

禁止：
- 在 UI 中散落埋点实现细节。
- 静默忽略关键业务失败。
- 只记录错误文本，不记录上下文。

推荐：
- 关键流程记录开始、成功、失败和耗时。
- 离线埋点进入本地队列，恢复网络后批量上报。

---

## 十一、简单演进约束

原则：
- 简单优先，但简单不能破坏边界；抽象必须服务于明确变化。

必须：
- 小功能可以轻量实现，但仍需保持分层和依赖方向。
- 当重复出现两次以上，或变化点已经明确时，再抽象公共能力。
- 复杂度必须有业务收益或维护收益。

禁止：
- 为模式而模式。
- 为未来不确定需求提前设计复杂框架。
- 用继承层级复用业务页面。
- 创建万能 Service、万能 Store、万能 Utils。

推荐：
- 优先组合复用，而不是继承复用。
- 优先局部封装，而不是全局化。
- 优先删除偶然复杂度。

---

## 十二、生成自检清单

生成或评审代码前必须检查：

- 这个模块是否只负责一个方向的变化？
- UI 是否只负责展示和事件派发？
- API DTO 是否被隔离在数据层？
- 状态是否明确归属，是否存在重复存储？
- 异步流程是否覆盖 loading、empty、error、retry？
- 写操作是否能安全处理重复点击和网络重试？
- 业务逻辑是否依赖了具体 SDK、Dio、Storage 或 Router？
- 扩展点是否集中，是否存在散落 if/else？
- 错误是否带上下文，是否能定位失败阶段？
- 新增能力是否可以通过新增模块完成，而不是修改多处旧代码？
- 当前抽象是否有明确重复、变化或隔离需求？
- 是否保持了高内聚、低耦合、可维护、可扩展？
