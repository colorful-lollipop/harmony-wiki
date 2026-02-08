# 安全风险评估

## 1. 攻击面分析

### N-API 入口点

| 攻击面 | 文件位置 | 风险等级 |
|--------|----------|----------|
| `publish` API | `frameworks/js/napi/src/publish.cpp` | **高** |
| `subscribe` API | `frameworks/js/napi/src/subscribe.cpp` | **高** |
| `cancel` API | `frameworks/js/napi/src/cancel.cpp` | **中** |
| `setBadgeNumber` API | `frameworks/js/napi/src/manager/napi_*.cpp` | **中** |

### IPC 接口

| 攻击面 | SA ID | 风险等级 |
|--------|-------|----------|
| IAnsManager | 3203 | **高** |
| IAnsSubscriber | callback | **高** |

---

## 2. 信任边界

```
┌─────────────────────────────────────────────────────┐
│                    不可信区域                          │
│  ┌─────────────────────────────────────────────┐   │
│  │  应用进程 (沙箱内)                            │   │
│  │  - JS/N-API 调用                             │   │
│  │  - 通知数据构造                              │   │
│  └─────────────────────────────────────────────┘   │
│                        ↑                            │
│                        ↓ IPC                        │
│  ┌─────────────────────────────────────────────┐   │
│  │  Foundation 进程                             │   │
│  │  - SA 验证与授权                             │   │
│  │  - 数据过滤与清洗                            │   │
│  │  - 持久化存储                                │   │
│  └─────────────────────────────────────────────┘   │
│                        ↑                            │
│                        ↓ 内部API                     │
│  ┌─────────────────────────────────────────────┐   │
│  │  系统服务 ( privileged)                      │   │
│  │  - 核心逻辑                                 │   │
│  │  - 分布式同步                               │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## 3. 已识别风险点

### 风险1: 通知内容注入

**证据**: `services/ans/src/advanced_notification_publish_service.cpp`

**触发条件**:
```
应用构造 NotificationRequest → 发布通知 → 内容显示
```

**影响**: 可能显示恶意构造的内容（如特殊字符、格式化字符串）

**修复建议**:
- 在 `PrepareNotificationRequest()` 中添加内容过滤
- 对用户输入进行 XSS/Escape 处理
- 限制可包含的 HTML/Markup 内容

---

### 风险2: 权限提升

**证据**: `services/ans/src/access_token_helper.cpp`

**触发条件**:
```
应用调用 publishAsBundle() → 伪装成其他应用发布
```

**影响**: 应用可能绕过权限检查，以其他应用身份发布通知

**修复建议**:
- 验证 `representativeBundle` 的授权
- 检查调用者与目标 bundle 的一致性
- 记录跨应用发布的审计日志

---

### 风险3: 拒绝服务 (DoS)

**证据**: `services/ans/src/advanced_notification_flow_control_service.cpp`

**触发条件**:
```
大量 publish 请求 → 超出处理能力 → 服务响应变慢
```

**影响**: 通知服务不可用或响应超时

**修复建议**:
- 实现更严格的速率限制
- 对单应用通知数量设上限
- 使用 FFRT 队列隔离处理

---

### 风险4: 路径遍历

**涉及文件**: `services/ans/src/common/file_utils.cpp`

**触发条件**:
```
通知附带图片路径 → 图片加载 → 构造路径访问敏感文件
```

**影响**: 可能读取系统私有文件

**修复建议**:
- 使用 `realpath()` 规范化路径
- 检查路径是否在允许目录内
- 使用沙箱化的图片解码服务

---

### 风险5: 订阅信息泄露

**证据**: `services/ans/src/notification_subscriber_manager.cpp`

**触发条件**:
```
任意应用调用 subscribe() → 获取其他应用的订阅信息
```

**影响**: 泄露用户隐私（哪些应用被订阅）

**修复建议**:
- 区分系统应用和普通应用的查询权限
- 对订阅信息进行脱敏处理
- 添加 `subscribe` 操作的审计日志

---

### 风险6: 徽章数字溢出

**涉及文件**: `services/ans/src/badge_manager/badge_manager.cpp`

**触发条件**:
```
应用设置超大徽章数字 → 整数溢出 → 显示异常
```

**影响**: UI 显示错误或崩溃

**修复建议**:
- 对徽章数字进行范围检查 (0-99)
- 使用安全的整数类型
- 捕获溢出异常

---

### 风险7: 分布式同步数据篡改

**证据**: `services/distributed/`

**触发条件**:
```
设备A发布通知 → 同步到设备B → 中间人篡改
```

**影响**: 跨设备通知内容被篡改

**修复建议**:
- 实现端到端加密
- 添加数字签名验证
- 使用 Dsoftbus 安全通道

---

## 4. 安全控制措施

### 访问控制

| 控制点 | 实现位置 | 说明 |
|--------|----------|------|
| 应用签名验证 | `access_token_helper.cpp` | 验证应用身份 |
| 权限检查 | `permission_filter.cpp` | 检查 NOTIFICATION_CONTROLLER |
| 用户隔离 | `os_account_manager_helper.cpp` | 多用户隔离 |

### 数据验证

| 验证点 | 实现位置 | 说明 |
|--------|----------|------|
| 通知请求校验 | `advanced_notification_publish_service.cpp` | 必填字段检查 |
| 通道类型校验 | `advanced_notification_slot_service.cpp` | 有效通道类型 |
| 字符串长度限制 | `notification_request.cpp` | 防止缓冲区溢出 |

### 加密措施

| 加密项 | 实现位置 | 说明 |
|--------|----------|------|
| 偏好设置加密 | `notification_preferences.cpp` | AES-GCM 加密 |
| 分布式数据传输 | `distributed_manager/` | Dsoftbus 加密 |

**证据**: `services/ans/src/common/aes_gcm_helper.cpp`

---

## 5. 安全建议

### 高优先级

1. **输入验证强化**: 对所有 N-API 参数进行严格校验
2. **权限最小化**: 移除不必要的 `ohos.permission` 声明
3. **审计日志**: 记录关键操作（发布、取消、订阅）

### 中优先级

1. **沙箱化**: 图片等资源加载应通过沙箱处理
2. **速率限制**: 完善 DoS 防护机制
3. **代码审计**: 定期进行安全代码审查

### 低优先级

1. **模糊测试**: 增加 fuzzing 测试覆盖
2. **依赖扫描**: 定期更新第三方依赖版本
