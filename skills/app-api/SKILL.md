---
name: app-api
description: "App HTTP API 设计约束：RESTful 资源建模、统一响应、分页过滤、错误码、鉴权、幂等与版本兼容。"
---

# App API Design Skill

Use this skill when designing, reviewing, or implementing App-facing HTTP APIs. The API style must be RESTful, predictable for mobile clients, and consistent with Flutter App Repository-based calling.

## 一、默认约定规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 使用 HTTPS | App 与服务端通信必须使用 HTTPS，拒绝明文 HTTP。 |
| 使用 JSON | 请求和响应默认使用 JSON。 |
| 使用 Bearer Token | 登录态默认通过 `Authorization: Bearer <token>` 传递。 |
| 使用 `/v1` | API 版本号默认放在 URL 路径中。 |
| 统一响应结构 | 所有接口统一使用 `code/message/data`。 |

### 允许

```http
GET https://api.example.com/v1/users/123
Authorization: Bearer <access_token>
Accept: application/json
```

### 拒绝

```http
GET http://api.example.com/users/123
GET https://api.example.com/users/123
```

```text
同一项目混用多种响应结构
业务接口直接返回 HTML/XML
```

### 示例

```http
GET https://api.example.com/v1/users/me
Authorization: Bearer xxx
Accept: application/json
```

## 二、地址版本规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 地址完整 | API 地址必须包含协议、域名、版本号、资源路径。 |
| 版本入径 | 版本号默认放在 URL 路径中，便于调试、网关路由和兼容。 |
| 资源挂载 | 资源路径必须挂在 `/v1` 之后。 |

### 允许

```http
https://api.example.com/v1/users
https://api.example.com/v1/orders/123
```

### 拒绝

```http
http://api.example.com/users
https://api.example.com/users
https://api.example.com/api/users
```

### 示例

```http
GET https://api.example.com/v1/orders?page=1&page_size=20
```

## 三、资源命名规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 资源名词 | URI 表达资源，不表达动作。 |
| 复数集合 | 集合资源使用复数名词。 |
| 方法表达 | 操作类型由 HTTP Method 表达。 |
| 字段统一 | 路径和参数字段默认使用 `snake_case`。 |
| 原子命名 | 资源名是最小业务名词，不带场景/角色/消费方前缀。 |
| 禁用别名 | 资源详情用 `:id`，禁止 `me / self / current` 等魔法别名。 |

### 允许

```http
/users
/orders
/products
/messages
/payments
/codes
/versions
/users/:userId
```

### 拒绝

```http
/getUsers
/createOrder
/updateProduct
/deleteMessage
/payOrder
/sms-verification-codes
/app-versions
/users/me
```

### 示例

```http
GET /v1/users
GET /v1/users/:userId
POST /v1/orders
PATCH /v1/orders/:orderId
DELETE /v1/messages/:messageId
```

## 四、资源关系规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 父子资源 | 一对多关系可以使用父资源/子集合表达。 |
| 集合过滤 | 同一关系也可以通过集合查询参数表达。 |
| 层级限制 | URI 层级最多推荐到集合/资源/子集合。 |
| 关联链接 | 资源详情可以返回关联资源 ID 或关联资源 URL。 |
| 复杂拆分 | 多级复杂关系应拆成多个独立资源接口。 |

### 允许

```http
GET /v1/users/123/orders
GET /v1/orders?user_id=123&page=1&page_size=20
GET /v1/orders/456/items
GET /v1/orders/456/payments
```

### 拒绝

```http
GET /v1/users/123/orders/456/items/789/products/111
GET /v1/getUserOrders
GET /v1/queryOrdersByUser
```

### 示例

父资源子集合：

```http
GET /v1/users/123/orders
```

集合过滤：

```http
GET /v1/orders?user_id=123&page=1&page_size=20
```

资源关联字段：

```json
{
  "id": "order_456",
  "user_id": "user_123",
  "items_url": "/v1/orders/456/items",
  "payments_url": "/v1/orders/456/payments"
}
```

## 五、请求方法规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| GET 查询 | `GET` 用于查询资源或资源集合。 |
| POST 创建 | `POST` 用于在集合下创建新资源。 |
| PUT 全量 | `PUT` 用于全量替换资源。 |
| PATCH 局部 | `PATCH` 用于局部更新资源。 |
| DELETE 删除 | `DELETE` 用于删除资源。 |
| 动作转资源 | 取消、支付、验证码等动作型业务必须转成业务结果资源。 |

### 允许

```http
GET /v1/orders
POST /v1/orders
PUT /v1/orders/456
PATCH /v1/orders/456
DELETE /v1/orders/456
```

### 拒绝

```http
POST /v1/getOrders
POST /v1/createOrder
POST /v1/updateOrder
POST /v1/deleteOrder
```

### 示例

```http
GET /v1/orders
GET /v1/orders/456
POST /v1/orders
PATCH /v1/orders/456
DELETE /v1/orders/456
```

动作型业务转资源：

```http
POST /v1/orders/:orderId/cancelations
POST /v1/orders/:orderId/payments
POST /v1/codes
POST /v1/messages/:messageId/read-receipts
```

## 六、分页查询规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 默认分页 | 列表接口默认使用 `page + page_size`。 |
| 首页规则 | `page` 从 1 开始，`page=1` 表示第一页。 |
| 大小限制 | 服务端必须限制 `page_size` 最大值。 |
| 列表必分页 | 返回集合的接口必须支持分页。 |
| 游标例外 | 聊天、Feed、通知流允许使用 cursor 分页，但必须单独声明。 |

### 允许

```http
GET /v1/orders?page=1&page_size=20
GET /v1/users?page=2&page_size=50
```

### 拒绝

```http
GET /v1/orders
GET /v1/orders?page=0&page_size=20
GET /v1/orders?p=1&size=20
```

### 示例

请求：

```http
GET /v1/orders?page=1&page_size=20
```

响应：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [],
    "page": 1,
    "page_size": 20,
    "total": 100,
    "has_more": true
  }
}
```

## 七、过滤排序规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 过滤入参 | 过滤条件放在 query 参数中。 |
| 排序字段 | 排序默认使用 `order_by`。 |
| 多重排序 | 多个排序字段使用英文逗号分隔。 |
| 字段命名 | 过滤和排序字段使用 `snake_case`。 |

### 允许

```http
GET /v1/orders?status=paid&page=1&page_size=20
GET /v1/orders?status=paid&order_by=created_at desc
GET /v1/orders?order_by=created_at desc,amount asc
```

### 拒绝

```http
POST /v1/getOrdersByStatus
GET /v1/orders/status/paid
GET /v1/orders?s=1&t=2
```

### 示例

```http
GET /v1/orders?status=paid&type=normal&order_by=created_at desc&page=1&page_size=20
```

## 八、公共参数规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| Header 承载 | App 公共参数通过 Header 携带。 |
| Body 纯净 | 业务 body 只传业务参数。 |
| 拦截注入 | 公共 Header 由 HTTP 拦截器统一注入。 |
| 请求追踪 | 每个请求必须携带 `X-Request-Id` 用于链路追踪。 |

### 允许

```http
Authorization: Bearer <access_token>
X-App-Id: client-app
X-App-Version: 1.0.0
X-Platform: ios
X-Device-Id: device-id
X-Channel-Id: appstore
X-Request-Id: uuid
X-Tenant-Id: tenant-id
Accept-Language: zh-CN
Accept: application/json
Content-Type: application/json
```

### 拒绝

```json
{
  "app_version": "1.0.0",
  "device_id": "device-id",
  "channel_id": "appstore",
  "business_id": "123"
}
```

```text
每个接口手动拼公共 Header
每个业务请求 body 重复传公共参数
```

### 示例

```http
GET /v1/users/me
Authorization: Bearer xxx
X-App-Version: 1.0.0
X-Platform: ios
X-Device-Id: abc
X-Request-Id: req-123
```

## 九、响应结构规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 统一 JSON | 所有接口使用统一 JSON 响应结构。 |
| 状态分层 | HTTP 状态码表达请求状态，业务 `code` 表达业务结果。 |
| 消息可读 | `message` 表达可读说明，不暴露服务端内部异常。 |
| 数据归口 | 成功或失败的业务数据统一放入 `data`。 |

### 允许

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

### 拒绝

```json
{
  "success": true,
  "result": {}
}
```

```json
{
  "error": "SQL syntax error"
}
```

### 示例

成功：

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": "user_123",
    "nickname": "Tom"
  }
}
```

失败：

```json
{
  "code": "TOKEN_EXPIRED",
  "message": "登录已过期，请重新登录",
  "data": null
}
```

## 十、状态码表规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 表达状态 | HTTP 状态码用于表达请求是否成功、是否鉴权失败、是否资源不存在。 |
| 统一处理 | App 必须根据状态码执行统一处理。 |
| 区分业务 | HTTP 状态码表达大类，业务 `code` 表达具体原因。 |

### 允许

| 状态码 | 名称 | 错误理由 | App 处理 |
|---:|---|---|---|
| 200 | OK | 请求成功 | 正常解析 data |
| 201 | Created | 资源创建成功 | 正常解析 data |
| 202 | Accepted | 异步任务已接收 | 进入轮询或等待 |
| 204 | No Content | 删除成功且无响应体 | 直接视为成功 |
| 400 | Bad Request | 请求参数格式错误 | 展示参数错误 |
| 401 | Unauthorized | 未登录或 Token 无效 | 清登录态并跳登录 |
| 403 | Forbidden | 已登录但无权限 | 展示无权限 |
| 404 | Not Found | 资源不存在 | 展示空态或已删除 |
| 409 | Conflict | 资源状态冲突 | 提示刷新后重试 |
| 422 | Unprocessable Entity | 业务校验失败 | 展示校验错误 |
| 429 | Too Many Requests | 请求过于频繁 | 提示稍后重试 |
| 500 | Internal Server Error | 服务端异常 | 展示服务异常 |

### 拒绝

```text
所有错误都返回 200
所有成功都返回 201
Token 失效返回 500
参数错误返回 200
```

### 示例

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
```

```json
{
  "code": "TOKEN_EXPIRED",
  "message": "登录已过期，请重新登录",
  "data": null
}
```

## 十一、错误码表规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 稳定可枚举 | 错误码必须稳定，不能随意变更。 |
| 英文编码 | 错误码使用英文大写蛇形命名，不使用中文。 |
| 可文档化 | 所有错误码必须能在接口文档中查询。 |
| 驱动逻辑 | App 可以根据错误码执行固定处理逻辑。 |

### 允许

| 错误码 | 错误理由 | HTTP 状态码 | App 处理 |
|---|---|---:|---|
| SUCCESS | 请求成功 | 200 | 正常解析 |
| INVALID_ARGUMENT | 参数格式错误 | 400 | 展示参数错误 |
| VALIDATION_ERROR | 业务校验失败 | 422 | 展示字段错误 |
| UNAUTHORIZED | 未登录 | 401 | 跳转登录 |
| TOKEN_EXPIRED | Token 已过期 | 401 | 清登录态并跳登录 |
| FORBIDDEN | 无访问权限 | 403 | 展示无权限 |
| NOT_FOUND | 资源不存在 | 404 | 展示空态或已删除 |
| CONFLICT | 资源状态冲突 | 409 | 提示刷新后重试 |
| TOO_MANY_REQUESTS | 请求过于频繁 | 429 | 提示稍后重试 |
| SERVER_ERROR | 服务端异常 | 500 | 展示服务异常 |
| NETWORK_ERROR | 网络不可用 | 本地错误 | 展示网络错误 |
| TIMEOUT | 请求超时 | 本地错误 | 展示超时重试 |

### 拒绝

```json
{
  "code": "登录过期",
  "message": "token expired"
}
```

```json
{
  "code": 50001,
  "msg": "error"
}
```

### 示例

```json
{
  "code": "VALIDATION_ERROR",
  "message": "参数校验失败",
  "data": {
    "fields": {
      "phone": "手机号格式错误"
    }
  }
}
```

## 十二、错误理由规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 指导处理 | 错误理由必须能指导 App 做正确处理。 |
| 用户可读 | `message` 面向用户或前端展示。 |
| 屏蔽内部 | 服务端内部异常不得直接暴露给 App。 |
| 细节入 data | 字段级错误、当前状态等细节放入 `data`。 |

### 允许

| 场景 | 错误码 | 错误理由 | message |
|---|---|---|---|
| Token 过期 | TOKEN_EXPIRED | 登录态失效 | 登录已过期，请重新登录 |
| 参数缺失 | INVALID_ARGUMENT | 请求参数缺失 | 缺少必要参数 |
| 字段非法 | VALIDATION_ERROR | 字段校验失败 | 手机号格式错误 |
| 无权限 | FORBIDDEN | 当前用户无权限 | 暂无访问权限 |
| 资源不存在 | NOT_FOUND | 目标资源不存在 | 内容不存在或已删除 |
| 状态冲突 | CONFLICT | 资源状态不可操作 | 当前状态不支持该操作 |
| 请求频繁 | TOO_MANY_REQUESTS | 触发限流 | 操作过于频繁，请稍后再试 |
| 服务异常 | SERVER_ERROR | 服务端未预期异常 | 服务异常，请稍后重试 |

### 拒绝

```text
SQL syntax error
NullPointerException
Redis connection failed
Stack trace
数据库字段名泄露
```

### 示例

```json
{
  "code": "CONFLICT",
  "message": "当前订单状态不支持取消",
  "data": {
    "current_status": "completed"
  }
}
```

## 十三、应用调用规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 分层调用 | App 不直接调用底层 HTTP。 |
| 仓储隔离 | 业务页面通过 Provider/Repository 获取数据。 |
| 统一拦截 | 公共 Header、错误处理、401 处理由 HTTP 层统一完成。 |
| DTO 隔离 | UI 层只使用 Entity，不直接依赖接口 DTO。 |

### 允许

```text
Page
-> AsyncNotifier
-> Repository
-> API
-> Dio
```

### 拒绝

```text
Widget 直接 new Dio
Widget 直接解析 Response
页面各自处理 401
页面各自拼 Header
模块内创建 Dio 单例
```

### 示例

```text
lib/common/http/
lib/modules/order/presentation/
lib/modules/order/domain/
lib/modules/order/data/
```

## 十四、接口检查规范

### 原则

| 原则项 | 含义解释 |
|---|---|
| 先审后联 | 每个接口设计完成后必须通过检查清单。 |
| 文档同步 | 接口进入联调前必须更新 OpenAPI 或接口文档。 |
| 规范阻断 | 未通过检查的接口不得进入联调。 |

### 允许

```text
有 HTTPS
有 /v1
URI 是资源名词
集合使用复数
HTTP Method 正确
列表支持 page/page_size
错误码已定义
状态码已定义
公共 Header 已说明
响应结构统一
OpenAPI 已更新
```

### 拒绝

```text
无文档直接联调
接口路径含动作词
分页参数不统一
错误响应不统一
业务 body 混入公共参数
Widget 直接调用接口
```

### 示例

```text
GET /v1/orders?page=1&page_size=20

检查结果：
HTTPS：通过
版本号：通过
资源命名：通过
分页：通过
响应结构：通过
错误码：通过
```
