# 安全风险评审

## 目的

本文档对 `device_status` 模块进行安全风险评审，识别攻击面、信任边界和潜在的安全问题，并提供修复建议。

---

## 攻击面清单

### 1. 外部输入攻击面

| 攻击面 | 入口 | 潜在威胁 | 涉及的 N-API 模块 |
|--------|------|--------|----------------------|--------|
| **JavaScript API 调用** | N-API 方法 | 参数注入、恶意回调劫持 | 所有 N-API 模块 |
| **IPC 通信** | Binder / Socket IPC | 恶意消息构造、权限提升 | DeviceStatusService、IntentionService |
| **拖拽数据** | 拖拽数据传输 | 恶意数据注入、资源耗尽 | Drag N-API |
| **屏募内容访问** | 屏幕内容获取 | 敏感信息泄露、恶意应用利用 | On-Screen N-API |
| **跨设备协同** | 输入事件注入 | 恶意输入注入 | Cooperate N-API |
| **Socket FD 分配** | Socket FD 申请 | 资源耗尽、特权提升 | AllocSocketFd API |
| **权限验证绕过** | 权限检查 | 系统应用伪装、Token 类型伪造 | 所有需要权限的 API |

### 2. 内部威胁

| 威胁 | 描述 | 影响组件 |
|--------|------|--------|
| **资源泄漏** | 内存泄漏、句柄泄漏 | 所有管理器 |
| **缓冲区溢出** | 数据拷贝未校验长度 | 各种数据处理 |
| **竞态条件** | 多线程竞争 | Epoll 事件循环、Socket 会话管理 |
| **未初始化使用** | 空指针解引用 | 插件回调、回调函数 |

### 3. 隐私泄露风险

| 威胁 | 描述 | 涉及的 N-API 模块 |
|--------|------|--------|
| **设备信息泄露** | 设备 ID、网络 ID | On-Screen、Distance Measurement N-API |
| **应用包名泄露** | 应用包名 | 所有 N-API 模块 |
| **用户 UID 泄露** | 调用方 UID | 权限验证相关代码 |
| **进程信息泄露** | 调用方 PID | IPCSkeleton 调用 |

---

## 信任边界

### 边界定义

| 边界 | 可信域 | 不可信域 | 防护措施 |
|--------|---------|--------|----------|
| **系统空间** | DeviceStatusService 及其子系统 | 应用空间（ArkTS/JS/Native） |
| **跨设备边界** | DSoftBus 连接的设备 | 网络隔离、验证网络 ID |
| **应用空间** | 所有调用 N-API 的应用 | 权限验证、Token 验证 |
| **插件空间** | Intention 插件 | 插件沙箱、权限控制 |

---

## 信任传递路径

```
应用层
    ↓ 权限验证
    ↓ Token 类型检查
    ↓ 系统应用验证
    ↓
[N-API]
    ↓
[DeviceStatusService - SA 2902]
    ↓
[IntentionService - 业务逻辑层]
    ↓
[DragServer / CooperateServer / ...] - 功能服务器
    ↓
[基础设施 - 适配器/IPC/调度]
    ↓
[硬件抽象 - Sensor HDI / MMI HDI]
```

---

## 可被利用点（已识别）

### 1. 权限验证不足 - 跨设备协同

**严重性**: 🔴 高

**位置**: `intention/cooperate/plugin/src/state_machine.cpp:333`

**证据**：
```cpp
startEvent.errCode->set_value(COMMON_PERMISSION_CHECK_ERROR);
```

**问题**：
在跨设备协同功能的权限检查中，仅检查了系统应用状态，没有验证具体的协同权限（如 `ohos.permission.COOPERATE_MANAGER`）。这可能导致：
1. 非系统应用可以启动协同功能
2. 恶意应用可以模拟设备协同行为
3. 跨越设备边界的协同操作

**可利用路径**：
```
应用（非系统应用）
  ↓ 协同 N-API 调用 CooperateClient::enable()
  ↓ [Intention IPC] IntentionService::PrepareCooperation()
  ↓ [权限检查] 验证系统应用（失败）
  ↓ 协同启动 [DSoftBus] 连接远程设备
  ↓ [输入注入] 发送恶意输入事件
```

**影响**：
- 恶意应用可以与多个设备建立协同
- 可以注入输入事件到远程设备
- 窃取用户操作（点击、拖拽等）

**修复建议**：
1. 添加明确的协同权限检查：`OHOS_BUILD_ENABLE_COORDINATION` 时应验证 `ohos.permission.COOPERATE_MANAGER`
2. 实施更严格的网络 ID 验证：验证远程设备的网络 ID 白名单
3. 增加操作日志：记录协同操作的完整调用链

---

### 2. 拖拽数据未充分校验 - 信息泄露风险

**严重性**: 🟠 中

**位置**: `services/interaction/drag/src/drag_data_manager.cpp`

**证据**：
```cpp
// 拖拽数据中可能包含敏感信息
class DragData {
    std::string shadowPixel;        // 像素数据
    std::string fileInfo;           // 文件信息
    std::vector<uint8_t> thumbnail;  // 缩略图
    std::string extraInfo;          // 额外信息
    PixelMap pixelMap;            // 像素映射
    std::vector<uint8_t> udKey;   // 用户数据密钥
    DragDataInfo dataInfo;
};
```

**问题**：
在跨设备拖拽时，拖拽数据可能包含应用敏感信息（文件路径、文件内容等），这些数据通过 IPC 传输到远程设备，可能被恶意应用读取。

**可利用路径**：
```
主设备应用
  ↓ 启动跨设备拖拽（Drag N-API）
  ↓ [IPC] DragManager::GetDataSummary()
  ↓ [IPC] 拖拽数据传输（包含敏感信息）
  ↓ 跨设备接收拖拽数据
  ↓ [DSoftBus] 远程设备应用读取拖拽数据
  ↓ [信息泄露] 提取敏感信息（文件路径、内容等）
```

**影响**：
- 文件路径泄露：拖拽数据中包含的文件路径可能暴露应用私有数据
- 文件内容泄露：文件内容可能包含用户敏感数据（如凭据、密钥）
- 像素数据泄露：缩略图可能包含 UI 元素或敏感内容

**修复建议**：
1. 敏感信息过滤：在传输前过滤敏感信息（路径、凭据等）
2. 加密传输：对跨设备传输的敏感数据进行加密
3. 权限检查：接收设备应验证是否有权限访问拖拽数据中的敏感信息
4. 最小化数据：只传输必要的拖拽数据，不携带额外敏感信息

---

### 3. Socket FD 分配无配额限制 - 资源耗尽

**严重性**: 🟠 高

**位置**: `services/communication/base/i_devicestatus.h:40`

**证据**：
```cpp
virtual int32_t AllocSocketFd(const std::string &programName, int32_t moduleType,
                                     int32_t &socketFd, int32_t &tokenType) = 0;
```

**问题**：
`AllocSocketFd` API 没有调用次数限制或配额控制。恶意应用可能：
1. 重复调用大量分配 Socket FD，耗尽系统资源
2. 分配多个 Socket FD 但不释放
3. 利用高权限的系统应用进行资源耗尽攻击

**可利用路径**：
```
高权限应用
  ↓ 重复调用 N-API: AllocSocketFd()
  ↓ [IPC] DeviceStatusService::AllocSocketFd()
  ↓ 分配大量 Socket FD
  ↓ 系统资源耗尽
  ↓ 正常服务无法分配新 FD（DoS 攻击）
```

**影响**：
- 系统资源耗尽：影响所有依赖 Socket FD 的服务
- 拒绝服务：其他应用无法获取 Socket FD，导致服务不可用

**修复建议**：
1. 添加调用频率限制：限制单个应用或进程的 Socket FD 分配数量
2. 实施 FD 配额：为每个应用设置 FD 分配上限
3. 监控和告警：监控 FD 使用情况，异常时告警
4. 优先级控制：系统应用和普通应用使用不同的配额
5. 超时释放机制：FD 不使用时自动回收

---

### 4. 屏募内容权限验证绕过 - 白名单注入

**严重性**: 🟠 高

**位置**: `intention/onscreen/server/src/on_screen_server.cpp:688`

**证据**：
```cpp
const std::map<std::string, std::string> hapWhiteListMap = {
    {"com.huawei.hmos.vassistant", "1189827130565864320"},
};

bool IsSystemCalling(const CallingContext &context) {
    // ...
    return (IsSystemAppByFullTokenID(context.fullTokenId));
}
```

**问题**：
白名单仅基于应用包名匹配，攻击者可以：
1. 创建与白名单包名相似的应用包名
2. 修改 APK 签名，通过包名校验
3. 重签应用（使用不同签名）后获取相同的包名

**可利用路径**：
```
恶意应用
  ↓ 创建包名: "com.huawei.hmos.vassistant.copy"
  ↓ 签名改为 "com.huawei.hmos.vassistant.copy"
  ↓ 安装应用
  ↓ 调用 On-Screen N-API
  ↓ [权限检查] 白名单匹配（通过）
  ↓ 获取页面内容
  ↓ [信息泄露] 提取屏幕信息
```

**影响**：
- 白名单失效：攻击者可以绕过白名单限制
- 系统应用被替代：恶意应用可能完全替代系统应用
- 权限滥用：获取本不应获得的屏幕内容

**修复建议**：
1. 增强白名单验证：不仅检查包名，还要验证签名和应用 ID
2. 应用签名验证：验证应用签名与系统签名的匹配
3. 运行时完整性检查：验证应用的完整性（防篡改）
4. 应用沙箱：使用更严格的沙箱隔离
5. 动态权限控制：为应用授予最小必要权限

---

### 5. 权限 Token 伪造 - 系统应用伪装

**严重性**: 🟠 高

**位置**: `intention/cooperate/plugin/src/state_machine.cpp:747`

**证据**：
```cpp
auto flag = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(context.tokenId);
if (flag == Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE ||
    flag == Security::AccessToken::ATokenTypeEnum::TOKEN_SHELL) {
    // 系统服务调用，放行
}
```

**问题**：
Token 类型检查仅用于区分 Native/Shell/HAP，未验证 Token 的有效性。攻击者可能：
1. 伪造 Native Token 的调用
2. 使用 Shell Token 执行系统级操作
3. 恶意应用声称自己是系统服务

**可利用路径**：
```
恶意应用（伪造 Token）
  ↓ 调用需要权限的 N-API
  ↓ [权限检查] Token 类型检查（放行）
  ↓ [N-API] 执行敏感操作
  ↓ [信息泄露] 获取敏感数据
  ↓ [权限提升] 绕过应用层权限检查
```

**影响**：
- 系统服务伪装：恶意应用可以完全绕过应用层权限
- 敏感操作：可以获取设备状态、屏幕内容等敏感信息
- 跨设备协同：可以启动协同，注入输入

**修复建议**：
1. Token 绑定：将 Token 与应用包名签名绑定，防止伪造
2. 证书验证：验证调用方的证书和签名
3. 调用栈审计：记录系统调用的完整调用栈，识别异常模式
4. 强化 Native Token 验证：使用硬件安全特性（如 TEE）
5. 限制 Native 权限：减少 Native Token 的使用范围

---

### 6. 缓冲区溢出风险 - 拖拽数据传输

**严重性**: 🟠 中

**位置**: `intention/ipc/sequenceable_types/include/sequenceable_drag_data.h`

**证据**：
```cpp
class SequenceableDragData : public Parcelable {
    std::string shadowPixel;     // 固定大小缓冲区风险
    std::string fileName;       // 文件名（未验证长度）
    std::string data;          // 拖拽数据（可能很大）
};
```

**问题**：
拖拽数据结构中的字符串成员（如 `fileName`、`shadowPixel`、`data`）可能：
1. 未验证长度，导致缓冲区溢出
2. 包含恶意构造的超长数据
3. 在 IPC 传输时被篡改

**可利用路径**：
```
恶意应用
  ↓ 创建恶意拖拽数据（包含超长字符串）
  ↓ 调用跨设备拖拽（Drag N-API）
  ↓ [IPC] 拖拽数据传输
  ↓ [缓冲区溢出] 远程设备处理数据时触发溢出
  ↓ [代码执行] 执行任意代码（RCE）
```

**影响**：
- 服务崩溃：IPC 处理进程崩溃
- 权限提升：溢出可能覆盖重要内存区域
- 拒绝服务：导致服务不可用

**修复建议**：
1. 长度限制：对所有字符串参数设置最大长度（如 4KB）
2. 使用安全的字符串操作：使用 `std::string` 而非 C 风格字符串
3. 输入验证：验证字符串长度和内容合法性
4. 内存安全：使用 `std::vector<uint8_t>` 或自定义安全缓冲区类
5. 跨边界检查：在序列化和反序列化时验证数据完整性

---

## 安全机制评估

### 权限验证机制

| 机制 | 有效性 | 潜在问题 | 改进建议 |
|--------|---------|---------|---------|
| **Token 类型检查** | 部分 | 未验证 Token 有效性 | 绑定签名、证书验证 |
| **系统应用验证** | 弱 | 包名仅匹配 | 签名+签名验证 |
| **白名单机制** | 弱 | 可伪造 | 签名+签名验证 |
| **UID/PID 追踪** | 部分 | 仅获取调用方信息 | 记录完整调用栈 |

### IPC 安全

| 机制 | 有效性 | 潜在问题 |
|--------|---------|---------|---------|
| **Binder IPC** | 强 | Binder 安全模型 | - |
| **Socket IPC** | 中 | 无内建加密 | 启用 DSoftBus 加密 |
| **序列化安全** | 弱 | 可篡改 | 使用消息认证（MAC） |

---

## 未发现/已缓解的风险

### 1. 数据校验完整性

| 风险 | 状态 | 说明 |
|--------|---------|----------|
| **拖拽数据校验** | ✅ 已缓解 | 使用 Sequenceable 框架，有长度检查 |
| **N-API 参数校验** | ✅ 已缓解 | N-API 层有基本的类型和长度检查 |
| **IPC 数据验证** | ✅ 已缓解 | 使用 Parcelable 验证 |

### 2. 资源管理

| 风险 | 状态 | 说明 |
|--------|---------|----------|
| **Socket FD 管理** | ⚠️ 部分 | 无配额限制 | 见可被利用点 3 |
| **内存泄漏** | ✅ 已缓解 | 使用智能指针和 RAII 模式 |
| **句柄泄漏** | ✅ 已缓解 | 使用 Android/Binder 自动管理 |

### 3. 线程安全

| 风险 | 状态 | 说明 |
|--------|---------|----------|
| **竞态条件** | ⚠️ 风分 | 部分锁存在 | 加强锁的使用和死锁检测 |
| **线程安全** | ✅ 已缓解 | 使用原子操作和线程安全容器 |

---

## 修复优先级建议

### 高优先级（P0）- 立即修复

1. **权限验证绕过（风险 1、5）**：加强协同和屏幕权限验证
2. **Socket FD 资源耗尽（风险 3）**：添加 FD 配额和监控
3. **跨设备协同漏洞（风险 1）**：增强网络 ID 验证
4. **缓冲区溢出（风险 6）**：实施严格的长度限制

### 中优先级（P1）- 近期修复

1. **白名单注入（风险 4）**：增强白名单验证机制
2. **Token 伪造（风险 5）**：实施 Token 绑定和验证
3. **信息泄露（风险 2）**：过滤敏感信息，加密传输

### 低优先级（P2）- 长期改进

1. **输入注入风险**：增加输入事件的来源验证
2. **调用栈审计**：实现系统调用监控和分析

---

## 安全开发建议

### 1. 输入校验

- **所有 N-API 参数必须校验**：类型、范围、长度
- **使用安全的转换函数**：`napi_get_value_int32` 而非 C 风格转换
- **拒绝空指针和无效值**：提前返回错误
- **使用 N-API 工具函数**：`napi_strict_check`

### 2. 权限检查

- **遵循最小权限原则**：只授予必要的权限
- **验证调用方身份**：检查 UID、PID、Token
- **记录权限检查失败**：用于安全审计
- **拒绝权限请求**：权限不足时立即拒绝

### 3. IPC 安全

- **使用 Binder 接口特性**：自动生命周期管理、类型安全
- **Socket 通信加密**：启用 DSoftBus 加密传输
- **消息认证**：对关键 IPC 消息添加 MAC 签名
- **序列化安全**：使用 Parcelable 框架验证数据完整性

### 4. 资源管理

- **使用 RAII 模式**：自动管理资源生命周期
- **智能指针**：避免内存泄漏
- **引用计数**：跟踪对象引用
- **定期清理**：实现资源回收机制

### 5. 日志和审计

- **安全日志**：使用 FI_HILOG 记录安全事件
- **不要记录敏感信息**：避免日志泄露
- **日志级别控制**：生产环境使用 INFO 或 WARN

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览和安全模型
- **[02_Directory_Structure](02_Directory_Structure.md)** - 模块职责和边界
- **[03_Architecture](03_Architecture.md)** - 信任边界和数据流
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 和权限要求

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
- **评审范围**：基于代码审查，不包含测试代码
