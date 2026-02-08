# 安全风险评审

## 概述

本文档对 hiviewdfx_cangjie_wrapper 进行安全风险评审，识别潜在攻击面、信任边界和安全风险点。

**评审范围**: `base/hiviewdfx/hiviewdfx_cangjie_wrapper`  
**评审时间**: 2026-02-06

---

## 攻击面分析

### 输入源分类

| 输入类型 | 来源 | 处理位置 |
|---------|------|---------|
| 用户字符串参数 | Cangjie 应用传入 | HiLog, HiAppEvent |
| 格式字符串 | Cangjie 应用传入 | HiLog |
| 事件参数 | Cangjie 应用传入 | HiAppEvent |
| 用户属性 | Cangjie 应用传入 | HiAppEvent |
| Watcher 配置 | Cangjie 应用传入 | HiAppEvent |

### 攻击面清单

| 序号 | 攻击面 | 模块 | 严重程度 |
|------|--------|------|---------|
| 1 | 格式字符串注入 | HiLog | 中 |
| 2 | 事件参数溢出 | HiAppEvent | 中 |
| 3 | 标签/名称长度溢出 | HiLog, HiAppEvent | 低 |
| 4 | 隐私数据泄露 | HiLog | 低 |
| 5 | 资源耗尽 | HiAppEvent | 中 |

---

## 信任边界

### 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                     Cangjie Application                          │
│  (可信: 开发者编写的业务代码)                                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   hiviewdfx_cangjie_wrapper                      │
│  (半可信: FFI 边界，类型安全检查)                                  │
│  - 参数长度检查                                                   │
│  - 格式字符串解析                                                  │
│  - 异常封装                                                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FFI Bridge Layer                              │
│  (边界: 类型转换，内存管理)                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Native Services                                 │
│  (可信: 系统服务，权限管控)                                        │
│  - HiLog Service                                                │
│  - HiAppEvent Service                                           │
│  - HiTrace Service                                              │
└─────────────────────────────────────────────────────────────────┘
```

### 边界说明

| 边界 | 说明 | 安全措施 |
|------|------|---------|
| 应用 → Cangjie 封装 | Cangjie 字符串传入 | 类型安全，边界检查 |
| Cangjie 封装 → FFI | 跨语言调用 | CString 转换，内存管理 |
| FFI → Native | Native 服务调用 | 服务端校验 |

---

## 安全风险点

### 风险1: 格式字符串注入

**描述**: HiLog 的格式字符串解析可能受到注入攻击。

**代码位置**: `ohos/hilog/hilog.cj:288-340`

**触发条件**:
```cj
// 恶意构造的格式字符串
let maliciousFormat = "%s%s%s%s%s%n%n%n%n%n"
Hilog.info(domain, tag, maliciousFormat, args)
```

**代码证据**:

```cj
// hilog.cj:288-340
func parseLogContent(formatStr: String, args: Array<String>): String {
    var logContent: String = ""
    // ... 解析逻辑
    while (pos < len) {
        // 只检查 %public 和 %private
        if (formatStr[(pos + PROPERTY_POS)..(pos + PROPERTY_POS + PUBLIC_LEN)] == "public") {
            // ... 处理
        }
        // 未检查其他格式说明符
    }
}
```

**影响**:
- 可能的拒绝服务
- 内存读取异常
- 格式化字符串泄露

**修复建议**:
1. 白名单验证格式说明符
2. 拒绝 `%n`, `%p`, `%h` 等危险说明符
3. 添加参数数量校验

**风险等级**: 中

---

### 风险2: 事件参数数量溢出

**描述**: HiAppEvent 的参数数量超过 32 个时静默丢弃。

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:400`

**触发条件**:
```cj
let params = HashMap<String, EventValueType>()
for (i in 0..50) {  // 超过 32 个参数
    params.add("param${i}", StringValue("value"))
}
let info = AppEventInfo("domain", "name", EventType.Behavior, params)
HiAppEvent.write(info)  // 超出的参数被丢弃，无警告
```

**代码证据**:

```cj
// cj_event.cj:399-401
// The maximum number of parameters is 32.
// If this limit is exceeded, excess parameters will be discarded.
```

**影响**:
- 数据丢失
- 业务逻辑异常
- 难以调试的问题

**修复建议**:
1. 参数超过限制时抛出异常
2. 添加日志警告
3. 限制参数添加时检查

**风险等级**: 中

---

### 风险3: 字符串长度溢出

**描述**: 标签、域名、事件名等字符串超长时静默截断。

**代码位置**: `ohos/hilog/hilog.cj:123`

**触发条件**:
```cj
// 标签超过 32 字节
let longTag = "A" * 100
Hilog.info(domain, longTag, "message", [])  // 静默截断
```

**代码证据**:

```cj
// hilog.cj:123-124
// @param { String } tag - Identifies the log tag,
// length cannot exceed 32 bytes, the excess part will be truncated.
```

**影响**:
- 标签混乱
- 日志过滤失效
- 调试困难

**修复建议**:
1. 超长时抛出异常
2. 添加日志警告

**风险等级**: 低

---

### 风险4: 隐私数据泄露风险

**描述**: `%{private}s` 格式可能被绕过。

**代码位置**: `ohos/hilog/hilog.cj:324`

**触发条件**:
```cj
// 隐私开关关闭时内容泄露
Hilog.info(domain, tag, "Password: %{private}s", [password])
// 如果隐私开关关闭，密码被打印
```

**代码证据**:

```cj
// hilog.cj:324
info.isPriv = showPriv && unsafe { IsPrivateSwitchOn() }

if (info.isPriv) {
    logContent += "<private>"
} else {
    logContent += args[info.count]  // 可能泄露隐私
}
```

**影响**:
- 敏感数据泄露
- 隐私合规问题

**修复建议**:
1. 隐私开关作为系统级配置
2. 敏感数据不应通过日志传输

**风险等级**: 低

---

### 风险5: 存储配额绕过

**描述**: ConfigOption.maxStorage 验证不严格。

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:495-499`

**触发条件**:
```cj
let config = ConfigOption(maxStorage: "999999999999999999999999G")
HiAppEvent.configure(config)  // 超大配额可能绕过检查
```

**代码证据**:

```cj
// cj_event.cj:495-499
// The quota value must meet the following requirements:
// The quota value consists of only digits and a unit...
// It is recommended that the quota be less than or equal to 10 MB.
```

**影响**:
- 存储耗尽
- 拒绝服务

**修复建议**:
1. 严格验证配额格式
2. 添加最大值限制
3. 拒绝过大配额

**风险等级**: 中

---

### 风险6: Watcher 回调注入风险

**描述**: Watcher 的 onTrigger/onReceive 回调可能被恶意覆盖。

**代码位置**: `ohos/hiviewdfx/hi_app_event/cj_event.cj:990-1001`

**触发条件**:
```cj
let watcher = Watcher("test")
watcher.onTrigger = Some(maliciousCallback)  // 回调可能被替换
```

**代码证据**:

```cj
// cj_event.cj:990
public var onTrigger: Option<(Int32, Int32, AppEventPackageHolder) -> Unit>

// cj_event.cj:1001
public var onReceive: Option<(String, Array<AppEventGroup>) -> Unit>
```

**影响**:
- 回调劫持
- 数据泄露

**修复建议**:
1. 回调在构造时固定
2. 避免公开的 setter

**风险等级**: 低

---

## 安全最佳实践

### 对于开发者

1. **日志安全**
   - 避免在日志中输出敏感信息
   - 使用 `%{private}s` 保护隐私数据
   - 不要将密码、Token 等放入日志

2. **事件参数**
   - 限制单个事件的参数数量（< 32）
   - 控制字符串参数长度（< 8KB）
   - 避免敏感数据作为事件参数

3. **存储管理**
   - 合理设置 maxStorage
   - 定期调用 clearData() 清理数据

### 对于集成方

1. **系统能力配置**
   ```json
   {
       "syscap": [
           "SystemCapability.HiviewDFX.HiLog",
           "SystemCapability.HiviewDFX.HiAppEvent",
           "SystemCapability.HiviewDFX.HiTrace"
       ]
   }
   ```

2. **隐私开关控制**
   - 正确配置系统隐私策略
   - 避免关闭隐私保护

---

## 风险汇总

| 风险 | 严重程度 | 可能性 | 风险等级 | 建议状态 |
|------|---------|--------|---------|---------|
| 格式字符串注入 | 中 | 低 | 中 | 需修复 |
| 事件参数溢出 | 中 | 中 | 中 | 需修复 |
| 字符串溢出 | 低 | 低 | 低 | 建议改进 |
| 隐私数据泄露 | 低 | 低 | 低 | 建议改进 |
| 存储配额绕过 | 中 | 低 | 中 | 需修复 |
| Watcher回调注入 | 低 | 低 | 低 | 建议改进 |

---

## 检查范围与局限性

### 已检查范围

| 模块 | 文件 | 检查内容 |
|------|------|---------|
| HiLog | hilog.cj | 格式字符串、隐私处理 |
| HiLog | hilog_channel.cj | 渠道配置 |
| HiTraceMeter | hi_trace_meter.cj | 打点参数 |
| HiAppEvent | hi_app_event.cj | 事件写入、配置 |
| HiAppEvent | cj_event.cj | 参数定义、类型安全 |

### 局限性说明

1. **Native 层未检查**: FFI 调用后的 Native 服务安全逻辑未在本次评审范围内
2. **运行时行为**: 部分安全措施依赖运行时配置，静态分析无法覆盖
3. **跨模块风险**: 模块间的交互风险可能未完全识别

---

## 相关文档

| 文档 | 描述 |
|------|------|
| [03_Architecture.md](03_Architecture.md) | 架构说明 |
| [04_External_API.md](04_External_API.md) | API 参考 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题 |
