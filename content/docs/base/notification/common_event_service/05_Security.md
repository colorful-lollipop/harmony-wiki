# 05_安全评审

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  应用进程   │    │  SA 进程   │    │  系统服务   │     │
│  │ (untrusted) │    │  (trusted) │    │ (trusted)  │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘     │
│         │                  │                  │            │
│         ▼                  ▼                  ▼            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Binder IPC (认证通道)                   │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 攻击面

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| N-API 输入 | JS 参数解析 | 高 |
| IPC 接口 | Binder 通信 | 中 |
| 事件名称 | 字符串处理 | 中 |
| 事件数据 | 载荷传递 | 中 |
| 订阅者权限 | 权限校验 | 高 |
| 文件路径 | 配置加载 | 低 |

## 攻击面分析

### 1. N-API 参数注入

**代码证据**: `interfaces/kits/napi/napi_common_event/src/common_event_parse.cpp`

```cpp
// 事件名称解析
napi_get_value_string_utf8(env, value, buf, bufLen, &bufSize);

// 数据解析
napi_get_value_string_utf8(env, value, data, dataLen, &dataLen);
```

**风险**: 恶意 JS 代码可能传入超长字符串或特殊字符

**影响**: 缓冲区溢出、拒绝服务

**建议**:
- [ ] 实现输入长度限制 (`STR_DATA_MAX_SIZE = 64KB`)
- [ ] 添加特殊字符过滤
- [ ] 使用安全的字符串处理函数

### 2. 权限校验绕过

**代码证据**: `services/src/common_event_permission_manager.cpp`

```cpp
// 权限检查
int32_t AccessTokenHelper::CheckPermission(
    const std::string &permission,
    int32_t userId);
```

**风险**: 发布者可能绕过 `publisherPermission` 检查

**影响**: 未授权事件发布

**建议**:
- [ ] 在 IPC 层增加权限校验
- [ ] 实现完整的权限链验证
- [ ] 增加审计日志

### 3. 事件名称注入

**代码证据**: `services/src/common_event_control_manager.cpp`

```cpp
// 事件存储
bool CommonEventControlManager::PublishCommonEvent(
    const CommonEventData &data,
    const CommonEventPublishInfo &publishInfo,
    const CommonEventSubscriberPtr &subscriber) {
    // 事件名称未做严格校验
}
```

**风险**: 恶意应用可能发布系统保留事件

**影响**: 系统行为异常

**建议**:
- [ ] 白名单校验系统事件
- [ ] 事件名称格式验证
- [ ] 隔离系统与应用事件

### 4. 有序事件竞争

**代码证据**: `services/src/common_event_control_manager.cpp`

```cpp
// 有序事件处理
if (commonEventPublishInfo.IsOrdered()) {
    // 多订阅者按优先级处理
    // 可能存在竞态条件
}
```

**风险**: 有序事件处理中的竞态条件

**影响**: 事件处理顺序异常

**建议**:
- [ ] 增加同步机制
- [ ] 事件处理超时保护
- [ ] 超时事件报告 (`ORDERED_EVENT_PROC_TIMEOUT`)

### 5. 订阅者数量耗尽

**代码证据**: `services/src/common_event_subscriber_manager.cpp`

```cpp
// 订阅者注册
bool CommonEventSubscriberManager::SubscribeCommonEvent(
    const CommonEventSubscribeInfo &subscribeInfo);
```

**风险**: 恶意应用创建大量订阅者

**影响**: 资源耗尽、拒绝服务

**当前保护**:
- [ ] `hisysevent.yaml`: `SUBSCRIBER_EXCEED_MAXIMUM` 监控

**建议**:
- [ ] 增加订阅者数量限制
- [ ] 实现订阅者配额管理
- [ ] 定期清理无效订阅者

### 6. 粘性事件安全

**代码证据**: `services/src/common_event_sticky_manager.cpp`

```cpp
// 粘性事件存储
bool CommonEventStickyManager::AddStickyCommonEvent(
    const CommonEventData &data);
```

**风险**: 粘性事件被恶意覆盖或泄露

**影响**: 数据泄露、状态篡改

**建议**:
- [ ] 限制粘性事件数量
- [ ] 敏感事件禁止粘性
- [ ] 粘性事件访问审计

## 安全机制

### 权限模型

| 权限 | 说明 | 使用场景 |
|------|------|----------|
| `ohos.permission.RECEIVER_STARTUP_COMPLETED` | 接收启动完成事件 | BOOT_COMPLETED 等 |
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 管理本地账户 | USER_* 事件 |
| `ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS` | 跨账户交互 | 多用户场景 |
| `ohos.permission.GET_WIFI_INFO` | 获取 WiFi 信息 | WiFi 相关事件 |
| `ohos.permission.USE_BLUETOOTH` | 使用蓝牙 | 蓝牙相关事件 |

### 数据完整性

| 机制 | 说明 |
|------|------|
| Binder 认证 | IPC 调用方身份验证 |
| Token 校验 | 应用身份标识 |
| 签名校验 | 应用签名验证 |

### 审计日志

**文件**: `hisysevent.yaml`

| 事件类型 | 说明 |
|----------|------|
| `ORDERED_EVENT_PROC_TIMEOUT` | 有序事件处理超时 |
| `STATIC_EVENT_PROC_ERROR` | 静态事件处理错误 |
| `SUBSCRIBER_EXCEED_MAXIMUM` | 订阅者超限 |
| `PUBLISH_ERROR` | 发布错误 |
| `SUBSCRIBE/UNSUBSCRIBE` | 订阅/取消订阅统计 |

## 安全配置

### 参数配置

**文件**: `services/etc/ces.para`

```properties
# CES SA 权限检查开关
notification.ces.check.sa.permission=false
```

### DAC 配置

**文件**: `services/etc/ces.para.dac`

配置不同用户/组的访问权限。

## 安全最佳实践

### 应用开发者

1. **最小权限原则**: 只请求必要的权限
2. **输入验证**: 验证所有外部输入
3. **事件过滤**: 只订阅需要的事件
4. **回调处理**: 实现事件回调超时机制
5. **敏感数据**: 避免在事件中传递敏感信息

### 系统集成

1. **权限隔离**: 严格隔离系统与应用事件
2. **资源限制**: 实施订阅者数量限制
3. **监控告警**: 启用 HiSysEvent 监控
4. **安全更新**: 及时修补安全漏洞

## 已知安全限制

| 限制 | 说明 | 缓解措施 |
|------|------|----------|
| IPC 身份伪造 | Binder 身份可能伪造 | 启用 SELinux |
| 资源耗尽 | 大量事件/订阅者 | 实施配额 |
| 信息泄露 | 事件数据可能被截获 | 敏感数据加密 |
| 拒绝服务 | 恶意应用发送大量事件 | 速率限制 |

## 相关文档

- [概览](00_Overview.md)
- [架构](01_Architecture.md)
- [N-API 接口](02_N-API.md)
- [构建编译](04_Build.md)
- [FAQ](06_FAQ.md)
