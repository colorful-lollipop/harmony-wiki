# 安全评审

> **目的**: 分析 Bluetooth 模块的安全风险、攻击面与修复建议  
> **适用范围**: 安全审计、代码审查、漏洞修复

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码目录** | `frameworks/` (不含测试) |
| **接口类型** | N-API、C API、IPC 接口 |
| **评审日期** | 2025-02-06 |
| **评审人** | Wiki 生成工具 |

## 威胁模型

### 信任边界

```mermaid
graph TB
    subgraph "可信区域 (Trusted)"
        KERNEL["Linux Kernel<br/>蓝牙驱动"]
        BT_SVC["Bluetooth Service<br/>SA 1130"]
    end
    
    subgraph "边界"
        IPC["IPC 通信<br/>SAMGR"]
    end
    
    subgraph "半可信区域"
        FW["Bluetooth Framework<br/>frameworks/inner/"]
    end
    
    subgraph "不可信区域 (Untrusted)"
        APP["应用层<br/>ArkTS/JS/C 应用"]
    end
    
    APP -->|N-API 调用| FW
    FW -->|IPC| IPC
    IPC -->|IPC| BT_SVC
    BT_SVC -->|HCI| KERNEL
    
    Note over FW,APP: 边界：N-API 参数校验
    Note over BT_SVC,FW: 边界：IPC 数据校验
```

### 数据流与敏感操作

| 数据流 | 敏感操作 | 风险等级 |
|--------|----------|----------|
| 用户输入 → N-API | 设备地址解析、参数校验 | 中 |
| N-API → IPC Proxy | 接口令牌校验 | 低 |
| IPC → SAMGR | 消息序列化 | 低 |
| SAMGR → Bluetooth Service | 路由到服务进程 | 低 |

## 攻击面分析

### 攻击面清单

| 攻击面 | 类型 | 暴露位置 | 说明 |
|--------|------|----------|------|
| **N-API 参数** | 输入验证 | `native_module_*.cpp` | JS 参数到 C 的转换 |
| **设备地址** | 字符串解析 | `bluetooth_host.cpp` | MAC 地址格式校验 |
| **Profile 连接** | IPC 调用 | `bluetooth_*_proxy.cpp` | 远程设备交互 |
| **权限检查** | 鉴权 | `napi_*.cpp` | 应用权限验证 |
| ** Parcel 反序列化** | 数据解析 | `ipc/parcel/*.cpp` | IPC 数据校验 |
| **广播数据** | BLE 广播 | `bluetooth_ble_advertiser.cpp` | 恶意广播包 |

### 输入验证点

| 验证点 | 文件:行号 | 验证类型 |
|--------|-----------|----------|
| 蓝牙地址格式 | `bluetooth_host.cpp:842` | 正则校验 |
| UUID 格式 | `uuid.cpp` | 格式校验 |
| 参数边界 | 各 `native_module_*.cpp` | 范围检查 |
| 空指针 | 各实现文件 | 空值检查 |

## 已识别风险

### 风险 1：设备地址未严格校验

**风险 ID**: BT-SEC-001  
**风险等级**: 中

**描述**: 蓝牙设备地址（MAC 地址）在解析时可能未进行严格格式校验，导致潜在的路径遍历或格式字符串攻击。

**证据**: `bluetooth_host.cpp:842`
```cpp
bool BluetoothHost::IsValidBluetoothAddr(const std::string &addr)
{
    // TODO: 实现地址校验逻辑
    return true;
}
```

**触发条件**:
1. 应用传入格式异常的蓝牙地址
2. 地址校验返回 `true`
3. 后续代码基于错误地址进行 IPC 调用

**影响**:
- 可能的 IPC 调用失败
- 资源泄露

**修复建议**:
```cpp
bool BluetoothHost::IsValidBluetoothAddr(const std::string &addr)
{
    // MAC 地址格式: XX:XX:XX:XX:XX:XX (17字符)
    if (addr.length() != 17) {
        return false;
    }
    
    // 验证格式和字符集
    for (size_t i = 0; i < addr.length(); ++i) {
        if (i % 3 == 2) {
            if (addr[i] != ':') {
                return false;
            }
        } else {
            if (!isxdigit(addr[i])) {
                return false;
            }
        }
    }
    return true;
}
```

---

### 风险 2：UUID 解析可能越界

**风险 ID**: BT-SEC-002  
**风险等级**: 中

**描述**: UUID 解析过程中可能存在缓冲区越界风险。

**证据**: `uuid.cpp` (需要进一步代码审查确认)

**触发条件**:
1. 应用传入超长 UUID 字符串
2. 解析函数未检查长度

**影响**:
- 缓冲区读取溢出
- 潜在的内存信息泄露

**修复建议**:
```cpp
// 在解析 UUID 前检查长度
const size_t MAX_UUID_LEN = 36; // 标准 UUID 长度
if (input.length() > MAX_UUID_LEN) {
    return ERR_INVALID_PARAM;
}
```

---

### 风险 3：N-API 参数校验不完整

**风险 ID**: BT-SEC-003  
**风险等级**: 低

**描述**: 部分 N-API 函数在参数转换过程中未进行完整的类型和边界校验。

**证据**: 需要审查各 `native_module_*.cpp` 中的参数解析逻辑

**触发条件**:
1. JS 传入异常类型参数
2. `napi_get_value_*` 函数可能返回意外值
3. 未检查 `napi_get_value_*` 的返回值

**影响**:
- 未定义行为
- 潜在崩溃

**修复建议**:
```cpp
napi_status status = napi_get_value_string_utf8(env, value, buffer, len, &written);
if (status != napi_ok) {
    napi_throw_error(env, "INVALID_ARG", "Failed to parse string argument");
    return nullptr;
}
```

---

### 风险 4：权限检查可能绕过

**风险 ID**: BT-SEC-004  
**风险等级**: 低

**描述**: 权限检查依赖于应用声明，但可能存在声明绕过或权限提升风险。

**证据**: `bundle.json` 中的权限声明

**触发条件**:
1. 应用声明了敏感权限
2. 但实际使用可能超出声明范围

**影响**:
- 权限滥用
- 数据泄露

**修复建议**:
- 在 IPC 层增加运行时权限校验
- 记录敏感操作的审计日志

---

### 风险 5：Parcel 反序列化风险

**风险 ID**: BT-SEC-005  
**风险等级**: 低

**描述**: IPC Parcel 数据反序列化时可能存在恶意数据攻击。

**证据**: `ipc/parcel/bluetooth_*.parcel.h/.cpp`

**触发条件**:
1. 恶意进程发送构造的 IPC 消息
2. Parcel 反序列化代码未检查数据范围

**影响**:
- 缓冲区溢出
- 拒绝服务

**修复建议**:
```cpp
// 在读取数据前检查长度
uint32_t size = data.ReadUint32();
if (size > MAX_ALLOWED_SIZE) {
    return ERR_INVALID_DATA;
}
```

---

## 安全最佳实践

### 1. 输入验证

**必须遵循**:
- 所有外部输入（JS 参数、IPC 数据、用户数据）必须经过验证
- 使用白名单验证而非黑名单
- 验证类型、长度、格式、范围

**示例**:
```cpp
// Good: 白名单验证
if (transport != BT_TRANSPORT_BREDR && transport != BT_TRANSPORT_LE) {
    return ERR_INVALID_PARAM;
}

// Bad: 仅类型检查
if (transport < 0) {
    return ERR_INVALID_PARAM;
}
```

### 2. 内存安全

**必须遵循**:
- 避免使用 `strcpy`, `sprintf` 等不安全函数
- 使用 `snprintf`, `strncpy_s` 等安全函数
- 始终检查 `new`/`malloc` 返回值
- 正确配对资源分配与释放

**示例**:
```cpp
// Good: 使用安全函数
snprintf(buffer, sizeof(buffer), "%s", str);

// Good: 检查返回值
char* data = new char[size];
if (data == nullptr) {
    return ERR_NO_MEMORY;
}
```

### 3. 权限控制

**必须遵循**:
- 敏感操作前检查应用权限
- 记录权限相关操作的审计日志
- 使用最小权限原则

**示例**:
```cpp
// 检查权限
if (!PermissionCheck(callingUid, "ohos.permission.USE_BLUETOOTH")) {
    return ERR_PERMISSION_DENIED;
}

// 审计日志
HILOG_SECURITY("Bluetooth connect: uid=%{public}d, device=%{public}s", 
               callingUid, deviceAddr.c_str());
```

### 4. 错误处理

**必须遵循**:
- 不泄漏敏感信息到错误消息
- 避免在错误路径中执行复杂逻辑
- 正确清理资源（RAII）

**示例**:
```cpp
// Good: 不泄漏敏感信息
try {
    ProcessData(sensitiveData);
} catch (const std::exception& e) {
    // 只记录通用错误，不输出敏感数据
    HILOG_ERROR("Failed to process data");
}

// Good: RAII 自动清理
class ScopedResource {
public:
    ScopedResource() { resource_ = Acquire(); }
    ~ScopedResource() { Release(resource_); }
};
```

## 安全相关配置

### 日志配置

| 日志标签 | 说明 | 文件位置 |
|----------|------|----------|
| `bt_napi_native_module` | N-API 模块日志 | `native_module.cpp:16` |
| `bt_fwk_host` | 主机框架日志 | `bluetooth_host.cpp:16` |
| `bt_fwk_ble` | BLE 框架日志 | `bluetooth_ble_*.cpp` |

### 安全相关头文件

| 头文件 | 说明 |
|--------|------|
| `bluetooth_log.h` | 日志宏定义 |
| `bluetooth_errorcode.h` | 错误码定义 |

## 审计清单

### 代码审查要点

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 所有外部输入已验证 | 待审查 | 需要检查各入口点 |
| 敏感操作已授权 | 待审查 | 权限检查点 |
| 内存操作安全 | 待审查 | 缓冲区使用 |
| 错误处理正确 | 待审查 | 资源清理 |
| 审计日志完整 | 待审查 | 敏感操作记录 |

### 渗透测试建议

| 测试项 | 测试方法 |
|--------|----------|
| 参数边界 | 传入超长/异常参数 |
| 并发安全 | 多线程并发调用 |
| 权限绕过 | 未授权应用调用 |
| IPC 注入 | 构造恶意 IPC 消息 |
| 状态篡改 | 修改运行时状态 |

---

**下一步**: [问题排查](06_Troubleshooting.md) → 常见问题与解决方案
