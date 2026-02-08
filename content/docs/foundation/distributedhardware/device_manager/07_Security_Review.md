# 安全风险评审

## 1. 评审范围

### 1.1 评估模块

| 模块 | 范围 | 优先级 |
|-----|------|-------|
| N-API 层 | `interfaces/kits/js4.0/` | 高 |
| Inner SDK | `interfaces/inner_kits/` | 高 |
| Service 实现 | `services/implementation/` | 高 |
| SA 配置 | `sa_profile/` | 中 |
| 工具库 | `utils/` | 低 |

### 1.2 排除范围

- 测试代码（`test/` 目录）
- 文档文件（`.md`, `.png`）
- 构建脚本（`BUILD.gn` 除外）

## 2. 威胁模型

### 2.1 外部输入

| 输入源 | 类型 | 风险等级 |
|-------|------|---------|
| JS 应用调用 | API 参数 | 高 |
| 设备发现结果 | 网络数据 | 高 |
| 认证数据 | PIN 码/凭据 | 高 |
| 用户 UI 操作 | UI 事件 | 中 |
| 系统事件 | 广播事件 | 低 |

### 2.2 敏感操作

| 操作 | 说明 | 风险等级 |
|-----|------|---------|
| 设备认证 | 建立信任关系 | 高 |
| 凭据存储 | 存储认证凭据 | 高 |
| 设备发现 | 扫描周边设备 | 中 |
| 状态变更 | 修改设备状态 | 中 |
| 日志输出 | 记录敏感信息 | 低 |

## 3. 攻击面分析

### 3.1 N-API 攻击面

**入口点**：`native_devicemanager_js.cpp`

**风险操作**：
- 参数解析：`napi_get_cb_info`
- 字符串操作：`std::string`
- 内存分配：`new`, `malloc`

**当前防护**：
- ✅ `CHECK_NULL_VOID_RET_LOG` 空指针检查
- ✅ `NAPI_CALL` 错误码检查
- ✅ 参数数量验证

> 证据来源：`native_devicemanager_js.cpp:37-55`

### 3.2 IPC 攻击面

**入口点**：`ipc_skeleton.h`

**风险操作**：
- `MessageParcel` 数据读取
- `Write/read` 序列化
- 跨进程调用

**当前防护**：
- ✅ `MessageParcel` 边界检查
- ✅ 权限校验
- ✅ 身份验证

### 3.3 设备发现攻击面

**入口点**：`softbus_connector.cpp`

**风险操作**：
- 设备信息解析
- 发现结果处理
- 过滤规则应用

**潜在风险**：
- ⚠️ 恶意设备伪造发现结果
- ⚠️ 大规模发现请求 DoS

### 3.4 设备认证攻击面

**入口点**：`hichain_connector.cpp`

**风险操作**：
- PIN 码处理
- 凭据交换
- 认证结果验证

**当前防护**：
- ✅ HiChain 内部安全机制
- ✅ 加密通信
- ✅ 凭据安全存储

## 4. 信任边界

### 4.1 边界定义

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  DeviceManager Service (SA 进程)                   │   │
│  │  - IPC 通信                                         │   │
│  │  - 设备认证                                          │   │
│  │  - 凭据管理                                          │   │
│  └─────────────────────────────────────────────────────┘   │
│                              ▲                              │
│                              │ IPC                         │
│                              ▼                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  应用进程（JS 调用）                                 │   │
│  │  - 参数验证                                          │   │
│  │  - 回调处理                                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
         │                              │
         ▼                              ▼
┌──────────────────┐        ┌──────────────────┐
│   DSoftBus       │        │   DeviceAuth     │
│   (可信子系统)     │        │   (可信子系统)    │
└──────────────────┘        └──────────────────┘
```

### 4.2 边界验证

| 边界 | 验证机制 | 实现位置 |
|-----|---------|---------|
| JS → Native | 参数校验 | `native_devicemanager_js.cpp` |
| App → Service | 权限检查 | `permission/` |
| Service → Subsystems | 身份验证 | `ipc_skeleton.h` |

## 5. 安全风险清单

### 5.1 高风险项

#### 风险 1：参数校验不完整

**证据**：`native_devicemanager_js.cpp:37-41`

```cpp
#define GET_PARAMS(env, info, num)    \
    size_t argc = num;                \
    napi_value argv[num] = {nullptr}; \
    napi_value thisVar = nullptr;     \
    NAPI_CALL(env, napi_get_cb_info(env, info, &argc, argv, &thisVar, nullptr))
```

**问题**：仅验证参数数量，未验证参数类型和内容

**触发条件**：
```javascript
// 传入非法类型参数
dmClass.startDiscovering(null);  // 传入 null
dmClass.startDiscovering(123);   // 传入数字
dmClass.startDiscovering({});    // 传入空对象
```

**影响**：可能导致空指针解引用、类型混淆

**修复建议**：
```cpp
// 添加参数类型和内容校验
napi_value arg = argv[0];
napi_valuetype type;
NAPI_CALL(env, napi_typeof(env, arg, &type));
if (type != napi_object) {
    napi_throw_type_error(env, nullptr, "Expected object");
    return;
}
```

---

#### 风险 2：设备 ID 未验证路径遍历

**证据**：`dm_device_info.h`（TODO：需确认具体文件）

**问题**：设备 ID 作为字符串直接用于文件路径或网络请求

**触发条件**：
```javascript
// 恶意构造的设备 ID
const maliciousDeviceId = "../../../etc/passwd";
dmClass.unbindTarget(maliciousDeviceId);
```

**影响**：路径遍历攻击、文件读取

**修复建议**：
```cpp
// 设备 ID 白名单校验
static const std::regex DEVICE_ID_PATTERN("^[a-zA-Z0-9-]{1,64}$");
if (!std::regex_match(deviceId, DEVICE_ID_PATTERN)) {
    return ERR_INVALID_DEVICE_ID;
}
```

---

#### 风险 3：认证回调未验证来源

**证据**：`hichain_connector.cpp`（TODO：需确认）

**问题**：认证结果回调未验证调用者身份

**触发条件**：
```cpp
// 恶意进程伪造认证结果
void OnAuthResult(const std::string& authId,
                 int32_t result,
                 const std::string& data)
```

**影响**：中间人攻击、认证绕过

**修复建议**：
```cpp
// 验证调用者 UID/PID
int32_t callerUid = IPCSkeleton::GetCallingUid();
if (callerUid != TRUSTED_SERVICE_UID) {
    return ERR_PERMISSION_DENIED;
}
```

---

#### 风险 4：设备发现结果未加密传输

**证据**：`softbus_connector.cpp`（TODO：需确认）

**问题**：设备发现结果在 DSoftBus 传输中可能被截获

**触发条件**：
- 中间人监听网络流量
- 恶意设备伪造发现响应

**影响**：设备信息泄露、伪造设备攻击

**修复建议**：
- ✅ DSoftBus 已实现传输加密（需验证）
- 增加设备证书校验

---

#### 风险 5：PIN 码明文处理

**证据**：`pin_auth.cpp`（TODO：需确认）

**问题**：PIN 码在内存中可能以明文形式存在

**触发条件**：
- 内存 Dump 攻击
- 进程 Attach 调试

**影响**：PIN 码泄露

**修复建议**：
```cpp
// 使用安全字符串
SecureString pinCode(input, length);
// 及时清理内存
memset_s(buffer, size, 0, size);
```

### 5.2 中风险项

#### 风险 6：日志信息泄露

**证据**：`dm_log.h`

**问题**：`LOGE`, `LOGI` 可能输出敏感信息

**当前状态**：
- ✅ 日志级别可配置
- ⚠️ 敏感信息（设备 ID、PIN 码）可能被记录

**建议**：添加敏感信息脱敏

---

#### 风险 7：并发竞态条件

**证据**：`native_devicemanager_js.cpp:87-95`

```cpp
std::mutex g_deviceManagerMapMutex;
std::mutex g_initCallbackMapMutex;
// ...
```

**问题**：多线程访问共享数据存在竞态风险

**触发条件**：多个 JS 线程同时调用 API

**影响**：数据竞争、资源泄露

**当前防护**：使用互斥锁保护

---

#### 风险 8：设备数量无上限

**证据**：`native_devicemanager_js.cpp:72-73`

```cpp
const int32_t DM_MAX_DEVICE_SIZE = 100;
const uint32_t DM_MAX_DEVICESLIST_SIZE = 50;
```

**问题**：设备列表上限可能导致拒绝服务

**触发条件**：
```javascript
// 短时间内发现大量设备
for (let i = 0; i < 10000; i++) {
    startDiscovering({...});
}
```

**影响**：内存耗尽、服务崩溃

**当前防护**：已设置上限，但未验证实现

---

### 5.3 低风险项

#### 风险 9：配置路径硬编码

**证据**：`device_manager.cfg:5-7`

```json
"mkdir /data/service/el1/public/database/distributed_device_manager_service 02770 device_manager ddms"
```

**问题**：路径硬编码不易配置

**影响**：路径冲突、权限配置不灵活

**建议**：支持环境变量配置

---

#### 风险 10：错误信息泄露

**证据**：`native_devicemanager_js.cpp`

**问题**：错误码可能泄露内部实现细节

**建议**：
- 统一错误码
- 避免泄露堆栈信息

## 6. 权限清单

### 6.1 已申请权限

**配置文件**：`sa_profile/device_manager.cfg`

| 权限 | 用途 | 风险等级 |
|-----|------|---------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 | 中 |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 软总线中心 | 高 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取包信息 | 中 |
| `ohos.permission.ENABLE_DISTRIBUTED_HARDWARE` | 使能分布式硬件 | 高 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 管理安全设置 | 高 |
| `ohos.permission.ACCESS_BLUETOOTH` | 访问蓝牙 | 中 |
| ... | ... | ... |

### 6.2 权限使用点

| API | 需要的权限 | 验证位置 |
|-----|----------|---------|
| `startDiscovering` | 无（4.0+） | Inner SDK |
| `bindTarget` | `DISTRIBUTED_DATASYNC` | Service |
| `getTrustedDeviceList` | 无 | Inner SDK |

## 7. 安全建议

### 7.1 输入校验

| 校验项 | 建议 | 优先级 |
|-------|------|-------|
| 参数类型 | 严格校验 napi_valuetype | 高 |
| 字符串长度 | 限制最大长度（64-256） | 高 |
| 特殊字符 | 过滤路径遍历字符 | 高 |
| JSON 解析 | 使用安全 JSON 库 | 中 |

### 7.2 输出脱敏

| 敏感数据 | 脱敏方式 | 示例 |
|---------|---------|------|
| 设备 ID | 部分隐藏 | `ABCD...1234` |
| PIN 码 | 禁止日志输出 | 不打印 |
| 认证凭据 | 加密存储 | 加密后保存 |

### 7.3 内存安全

| 措施 | 实现 | 优先级 |
|-----|------|-------|
| 敏感数据清理 | `memset_s` | 高 |
| 栈保护 | `-fstack-protector-strong` | 高 |
| 堆保护 | ASan/MSan | 中 |

> 证据来源：`services/implementation/BUILD.gn:162-167`

### 7.4 运行时保护

| 保护项 | 配置 | 验证 |
|-------|------|------|
| SELinux | `u:r:device_manager:s0` | ✅ 已配置 |
| seccomp | 系统调用过滤 | 建议添加 |
| ASLR | 随机化基地址 | 系统默认 |

> 证据来源：`sa_profile/device_manager.cfg:59`

## 8. 总结

### 8.1 风险统计

| 风险等级 | 数量 | 已缓解 | 需关注 |
|---------|-----|-------|-------|
| 高风险 | 5 | 2 | 3 |
| 中风险 | 3 | 1 | 2 |
| 低风险 | 2 | 0 | 2 |

### 8.2 总体评估

**评估结论**：`DeviceManager` 在架构上采用了多层次防护机制，包括：

- ✅ N-API 参数校验（需加强）
- ✅ IPC 权限控制（已实现）
- ✅ Service 沙箱隔离（SELinux）
- ✅ 编译时安全加固（sanitize flags）
- ⚠️ 输入校验（需完善）
- ⚠️ 敏感数据保护（需加强）

**建议优先级**：
1. 加强 N-API 参数校验
2. 完善设备 ID 验证
3. 增加认证回调来源验证
4. 添加敏感信息脱敏
5. 实施运行时保护

### 8.3 后续跟进

| 任务 | 负责人 | 预计时间 |
|-----|-------|---------|
| 参数校验加强 | 安全团队 | Sprint 1 |
| 设备 ID 验证 | 开发团队 | Sprint 2 |
| 敏感信息脱敏 | 开发团队 | Sprint 2 |
| 安全测试 | QA 团队 | Sprint 3 |
