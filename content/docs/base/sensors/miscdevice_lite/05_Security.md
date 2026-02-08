# sensors_miscdevice_lite 安全风险评审

## ⚠️ 重要声明

**本文档基于标准系统实现和通用架构推断**。

当前仓库**不包含源代码实现**，因此无法进行精确的代码级安全审计。本评审基于：

1. `bundle.json` 元数据
2. OpenHarmony 标准组件架构
3. 主实现仓库 `sensors_miscdevice` 的 API 规范
4. 通用小器件控制的安全风险模式

实际安全评审应以主实现仓库代码为准。

---

## 5.1 攻击面分析

### 5.1.1 外部输入点

| 输入类型 | 来源 | 处理层级 | 风险等级 |
|----------|------|----------|----------|
| **JS API 参数** | 用户应用 | N-API 层 | 中 |
| **IPC 消息** | 其他进程 | Service 层 | 中 |
| **HDI 调用** | 系统服务 | 驱动层 | 低 |
| **配置文件** | 文件系统 | Service 层 | 中 |
| **硬件事件** | 物理世界 | 驱动层 | 低 |

### 5.1.2 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      不可信区域                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            用户应用 (第三方应用)                       │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      信任边界 (应用框架)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌────────────┐    │
│  │   N-API 层     │  │ Service 层     │  │ HDF 驱动   │    │
│  │  (参数校验)    │  │ (业务逻辑)     │  │ (硬件控制) │    │
│  └────────────────┘  └────────────────┘  └────────────┘    │
├─────────────────────────────────────────────────────────────┤
│                      信任边界 (系统内核)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐   │
│  │              硬件 (振动马达、LED)                      │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 5.2 潜在安全风险

### ⚠️ 风险 1：振动时长过长导致拒绝服务

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **类型** | 资源耗尽 / DoS |
| **攻击面** | `startVibration(effect, attribute)` |
| **影响等级** | 中 |

**证据来源**: API 规范推断

```typescript
// 潜在攻击场景
vibrator.startVibration(
    { type: 'time', duration: 3600000 },  // 1 小时振动
    { usage: { scenario: vibrator.VibratorScenario.VIBRATOR_SCENATION_NOTIFICATION } }
);
```

**触发条件**:
1. 应用调用 `startVibration` 时长超过合理范围
2. 设备持续振动导致：
   - 电池快速耗尽
   - 设备发热
   - 用户无法使用设备

**修复建议**:
```cpp
// 在 N-API 层添加时长限制
const int32_t MAX_VIBRATION_DURATION = 10000;  // 10 秒

if (duration > MAX_VIBRATION_DURATION) {
    return NAPI_ERR_PARAMETER_INVALID;
}
```

**缓解措施**:
- 系统层设置全局振动时长上限
- 用户可随时取消振动
- 低电量模式下自动限制

---

### ⚠️ 风险 2：路径遍历攻击（自定义振动配置）

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **类型** | 路径遍历 / 任意文件读取 |
| **攻击面** | `VibrateFromFile` 类型 |
| **影响等级** | 中 |

**证据来源**: API 规范（`VibrateFromFile` 支持 hapticFd）

**触发条件**:
1. 应用使用 `type: 'file'` 模式
2. 提供的文件描述符指向敏感文件
3. 服务读取并解析配置文件

**修复建议**:
```cpp
// 只允许读取指定目录的配置文件
const char* ALLOWED_HAPTIC_DIR = "/system/etc/haptic/";

if (!strncmp(filePath, ALLOWED_HAPTIC_DIR, strlen(ALLOWED_HAPTIC_DIR))) {
    // 允许读取
} else {
    // 拒绝访问
}
```

---

### ⚠️ 风险 3：权限滥用（振动控制）

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-003 |
| **类型** | 未授权访问 |
| **攻击面** | `ohos.permission.VIBRATE` |
| **影响等级** | 中 |

**当前权限模型**:
```json
{
    "name": "ohos.permission.VIBRATE",
    "level": "system_grant"
}
```

**潜在问题**:
- `system_grant` 级别权限，应用安装时自动授予
- 恶意应用可能滥用振动功能进行：
  - 骚扰攻击（频繁振动）
  - 电池耗尽攻击
  - 用户干扰

**修复建议**:
```typescript
// 增加使用场景限制
interface VibratorAttribute {
    usage?: {
        scenario: VibratorScenario;  // 必须指定用途
        flags?: number;              // 额外限制
    };
}

// 限制可调用的场景
const ALLOWED_SCENARIOS = [
    'notification',
    'alarm',
    'interaction'
];

// 拒绝不合理的场景组合
if (!ALLOWED_SCENARIOS.includes(scenario)) {
    return NAPI_ERR_PERMISSION_DENIED;
}
```

---

### ⚠️ 风险 4：竞态条件（启动/停止振动）

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **类型** | 竞态条件 |
| **攻击面** | `startVibration` / `stopVibration` |
| **影响等级** | 低 |

**触发场景**:
```
时间线:
T0: 合法应用 A 调用 startVibration()
T1: 恶意应用 B 调用 startVibration() 覆盖
T2: 应用 A 调用 stopVibration() 停止了自己的振动
     (但意图是只停止自己的)
T3: 应用 B 的振动继续（被意外停止了？）
```

**修复建议**:
```cpp
// 使用会话 ID 管理振动生命周期
struct VibrationSession {
    int sessionId;
    pid_t callingPid;
    int64_t startTime;
};

napi_value StartVibration(napi_env env, napi_callback_info info) {
    // 1. 验证调用者权限
    // 2. 创建唯一会话 ID
    // 3. stopVibration 时验证会话 ID 归属
}
```

---

### ⚠️ 风险 5：LED 亮度超出安全范围

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **类型** | 硬件安全 / 过载 |
| **攻击面** | `setLedBrightness()` |
| **影响等级** | 低 |

**触发条件**:
1. 应用调用 LED 控制 API
2. 亮度值超过 LED 硬件限制
3. 可能导致 LED 过热或损坏

**修复建议**:
```cpp
// 在驱动层限制亮度范围
const int32_t MIN_BRIGHTNESS = 0;
const int32_t MAX_BRIGHTNESS = 255;

if (brightness < MIN_BRIGHTNESS || brightness > MAX_BRIGHTNESS) {
    return HDF_ERR_INVALID_PARAM;
}
```

---

## 5.3 安全最佳实践

### 5.3.1 输入验证清单

| 检查点 | 验证内容 | 处理方式 |
|--------|----------|----------|
| **时长校验** | `duration` 在 [0, MAX] 范围内 | 拒绝异常值 |
| **效果ID校验** | `effectId` 格式和长度 | 白名单验证 |
| **文件路径校验** | `hapticFd` 指向安全目录 | 路径规范化 |
| **权限校验** | 调用者持有必要权限 | 权限检查 |
| **频率限制** | 防止频繁调用 | 速率限制 |

### 5.3.2 权限模型建议

| 权限 | 当前级别 | 建议级别 | 说明 |
|------|----------|----------|------|
| `ohos.permission.VIBRATE` | system_grant | system_grant | 保持当前 |
| `ohos.permission.LED_CONTROL` | 无 | system_grant | 新增 |

### 5.3.3 安全编码规范

```cpp
// 1. 参数边界检查
int32_t StartVibration(int32_t duration) {
    if (duration < 0 || duration > MAX_VIBRATION_MS) {
        return ERROR_INVALID_PARAM;
    }
}

// 2. 权限验证
bool CheckVibratePermission(pid_t uid) {
    // 检查是否持有 VIBRATE 权限
}

// 3. 资源限制
const int MAX_CONCURRENT_VIBRATIONS = 1;
const int64_t MAX_TOTAL_VIBRATION_PER_MINUTE = 5000;  // ms
```

---

## 5.4 数据流与敏感数据

### 5.4.1 数据流图

```
用户应用 (参数: duration, effectId)
    ↓
N-API 层 (参数校验, 权限检查)
    ↓
Native Framework (会话管理)
    ↓
IPC (可选，跨进程)
    ↓
Service (业务逻辑)
    ↓
HDI (硬件调用)
    ↓
HDF Driver (硬件操作)
    ↓
Hardware (振动马达/LED)
```

### 5.4.2 敏感数据识别

| 数据类型 | 敏感级别 | 说明 |
|----------|----------|------|
| 振动模式配置 | 低 | 纯控制指令 |
| LED 亮度值 | 低 | 纯控制指令 |
| 硬件状态 | 低 | 设备状态信息 |
| 用户行为日志 | 中 | 可能泄露使用习惯 |

---

## 5.5 审计日志

### 5.5.1 建议记录内容

| 事件 | 记录字段 | 敏感度 |
|------|----------|--------|
| 振动启动 | 时间戳, PID, 时长, 场景 | 低 |
| 振动停止 | 时间戳, PID, 持续时长 | 低 |
| 权限检查失败 | 时间戳, PID, 缺少权限 | 中 |
| 参数异常 | 时间戳, PID, 异常参数 | 中 |

### 5.5.2 日志示例

```log
[2025-02-06 06:15:00.123] I/VIBRATOR: VibrationStart pid=1234 uid=5678 duration=1000 scenario=notification
[2025-02-06 06:15:01.456] I/VIBRATOR: VibrationStop pid=1234 uid=5678 duration=1000
[2025-02-06 06:15:02.789] W/VIBRATOR: InvalidParam pid=9999 uid=0000 reason=duration_too_long
```

---

## 5.6 安全相关代码位置（推断）

### 5.6.1 关键文件（待确认）

| 文件路径（推断） | 安全相关功能 |
|-----------------|--------------|
| `interfaces/plugin/napi_vibrator.cpp` | 参数校验、权限检查 |
| `frameworks/native/vibrator_manager.cpp` | 会话管理、资源限制 |
| `services/miscdevice_service/permission.cpp` | 权限验证 |
| `utils/security/*` | 安全工具 |

### 5.6.2 相关系统组件

| 组件 | 用途 |
|------|------|
| `abilityAccessCtrl` | 权限管理 |
| `permissionManager` | 权限检查 |
| `hdf` | 驱动框架安全 |

---

## 5.7 检查范围与局限性

### 5.7.1 已检查范围

| 类别 | 状态 | 说明 |
|------|------|------|
| N-API 参数校验 | ✅ 已评估 | 基于 API 规范推断 |
| 权限模型 | ✅ 已评估 | 基于标准权限系统 |
| 硬件访问控制 | ⚠️ 部分 | 需实际代码确认 |
| 驱动层安全 | ❌ 未检查 | 无源码访问权限 |

### 5.7.2 局限性声明

1. **无源代码**: 本评审基于元数据和 API 规范推断，未进行代码级审计
2. **推断架构**: 组件结构基于 OpenHarmony 标准模式，实际实现可能有差异
3. **未验证**: 实际安全漏洞需要通过渗透测试验证

---

## 5.8 相关文档

### 官方资源
- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/security-guidelines.md)
- [权限管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/runtime-permissions-0000001774280914)
- [sensors_miscdevice GitHub](https://github.com/openharmony/sensors_miscdevice)

### 本地文档
- [02_Architecture.md](./02_Architecture.md) - 架构说明
- [03_API.md](./03_API.md) - API 文档
- [06_Troubleshooting.md](./06_Troubleshooting.md) - 常见问题
