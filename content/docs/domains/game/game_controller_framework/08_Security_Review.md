# 08_Security_Review - 安全风险评审

## 目的

本文档提供 GameController Framework 的安全风险评审，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- **所有开发人员**（必读）
- 安全审计人员
- 架构师
- 维护人员

## 攻击面清单

| 攻击面 | 风险等级 | 检查范围 | 证据 |
|---------|---------|---------|------|
| **N-API/CAPI 接口** | 高 | 参数校验、权限检查 | 04_External_CAPI.md |
| **IPC 通信** | 中 | 权限验证、消息校验 | 03_Architecture.md |
| **配置文件** | 高 | 文件权限、路径遍历、JSON 解析 | 06_GN_Targets.md |
| **插件加载** | 低 | 加载路径验证 | 02_Directory_Structure.md |
| **输入事件** | 高 | 事件校验、注入检测 | 00_Overview.md |
| **JSON 解析** | 中 | 异常处理、大小限制 | _work/NOTES.md |

## 信任边界

```
┌─────────────────────────────────────────────────────────┐
│              应用层（不可信）                  │
│  ┌──────────────┐  ┌──────────────┐     │
│  │  游戏应用   │  │  终端厂商服务│     │
│  └──────┬───────┘  └──────┬───────┘     │
│         │                     │                 │
│         ▼                     ▼                 │
│  ┌─────────────────────────────────────────┐ │
│  │      CAPI 接口（参数校验）        │ │
│  └─────────────────────────────────────────┘ │
└───────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│          Framework 层（半可信）           │
│  - 参数校验（边界检查）              │
│  - 大小限制                           │
│  - Null 检查                          │
└─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│         SA 层（可信）                   │
│  - 权限验证（IsSACall、IsSystemAppCall） │
│  - Bundle 名称验证                      │
│  - JSON 文件读写                       │
│  - 配置文件（只读）                   │
└─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│         外部系统服务（可信）           │
│  - MMI Service（多模态输入）           │
│  - Window Manager（窗口框架）           │
│  - Bundle Manager（包管理）           │
│  - System Ability Manager               │
└─────────────────────────────────────────────────┘
```

**证据**:
- `service/ipc/src/gamecontroller_server_ability.cpp:143-150`: IsSystemAppCall() + VerifyBundleNameIsValid()
- `service/common/src/permission_utils.cpp:51-62`: 权限验证工具

## 可被利用点（至少 5 条）

### 1. 整数溢出风险

**证据**:
- `frameworks/native/common/include/gamecontroller_client_model.h:28`: `MAX_SIZE = 10`
- `service/common/src/permission_utils.cpp`: 数组大小检查

**风险描述**:
在 `CheckParamValid()` 方法中，使用索引访问数组时，如果数组大小未被严格校验，可能导致越界访问。

**触发路径**:
1. 应用构造恶意 GameInfo 数组，包含超过 MAX_SIZE 个元素
2. 调用 `SyncSupportKeyMappingGames()` 或 `SyncIdentifiedDeviceInfos()`
3. 服务端未严格校验数组实际大小，直接遍历到 MAX_SIZE
4. 访问超出实际数组的内存

**影响**:
- 内存泄漏
- 信息泄露（读取未授权内存）
- 潜在的远程代码执行（如果可以控制偏移）

**修复建议**:
1. 使用 `std::vector` 并检查实际大小而非依赖 MAX_SIZE
2. 在循环中使用数组实际大小：`for (size_t i = 0; i < gameInfos.size(); i++)`
3. 添加边界检查：`if (index >= gameInfos.size()) return GAME_ERR_ARRAY_MAXSIZE;`

**文件位置**:
- `frameworks/native/common/include/gamecontroller_keymapping_model.h:416-527`: CheckParamValid 实现
- `service/ipc/src/gamecontroller_server_ability.cpp`: CheckParamValid 调用点

---

### 2. JSON 解析注入风险

**证据**:
- `service/common/src/json_utils.cpp:100`: 自定义 IsUtf8() 验证器
- `service/common/src/json_utils.cpp:132`: WriteFileFromJson() 使用 nlohmann::json::parse()

**风险描述**:
自定义的 UTF-8 验证器可能存在绕过漏洞，攻击者可以构造特制化的 JSON 字符串，包含非 UTF-8 字符或多字节序列，绕过验证。

**触发路径**:
1. 终端厂商构造恶意 JSON 配置文件，包含恶意编码
2. 调用 `SetCustomGameKeyMappingConfig()` 或 `SetDefaultGameKeyMappingConfig()`
3. `IsUtf8()` 验证被绕过
4. JSON 文件写入，可能包含恶意内容
5. 后续读取时，可能触发缓冲区溢出或命令注入

**影响**:
- 配置文件篡改
- 持久化攻击（恶意配置持久化）
- 潜在的命令执行（如果解析器存在漏洞）

**修复建议**:
1. 使用系统 UTF-8 验证函数而非自定义实现
2. 添加 JSON 深度限制和递归限制
3. 使用 nlohmann::json 的安全解析选项（如 `allow_exceptions = true`）
4. 对解析后的数据进行二次校验

**文件位置**:
- `service/common/src/json_utils.cpp:82-130`: IsUtf8() 实现
- `service/common/src/json_utils.cpp:132-153`: JSON 文件读写

---

### 3. 路径遍历风险（部分缓解）

**证据**:
- `service/common/src/json_utils.cpp:135`: `realpath()` 调用
- `service/common/src/json_utils.cpp:156`: `realpath()` 调用
- `service/common/src/json_utils.cpp:192`: `realpath()` 调用

**风险描述**:
虽然使用了 `realpath()` 进行路径规范化，但如果 `realpath()` 调用失败或返回结果未检查，可能导致路径遍历攻击。当前实现仅检查返回值是否为空。

**触发路径**:
1. 攻击者构造包含 `..` 路径的 JSON 配置
2. 调用写入配置文件操作（如 SetCustomGameKeyMappingConfig）
3. `realpath()` 可能未能正确解析（如符号链接指向受限目录）
4. 文件写入到未授权位置

**影响**:
- 配置文件被写入到系统任意位置
- 权限提升
- 系统文件篡改

**修复建议**:
1. 检查 `realpath()` 返回值，确保不以 `/` 开头且在合法目录内
2. 添加路径白名单检查，限制写入路径到 `GAME_CONTROLLER_SERVICE_ROOT`
3. 使用 `openat2()` 系列调用，基于目录文件描述符
4. 确保文件权限为 root:gamecontroller_server:rw-r-----

**文件位置**:
- `service/common/src/json_utils.cpp:132-193`: 路径处理

---

### 4. 权限绕过风险

**证据**:
- `service/ipc/src/gamecontroller_server_ability.cpp:98-100`: IsSystemServiceCall()
- `service/common/src/permission_utils.cpp:51-62`: IsSACall() 实现

**风险描述**:
权限验证依赖 AccessTokenKit 的 `IsSACall()` 和 `IsSystemAppCall()`。如果这些系统调用存在漏洞或被绕过，特权 API 可能被未授权应用调用。

**触发路径**:
1. 恶意应用调用需要系统权限的 API（如 SetCustomGameKeyMappingConfig）
2. `IsSystemServiceCall()` 使用 AccessTokenKit 验证
3. 如果 AccessTokenKit 存在漏洞或被篡改，验证可能失败
4. 未授权应用获得权限，可以写入配置文件

**影响**:
- 未授权配置修改
- 系统服务干扰
- 持久化攻击

**修复建议**:
1. 在 SA 层添加额外的 UID 检查，验证调用者 UID 匹配预期
2. 添加调用链完整性验证（如签名验证）
3. 记录所有权限失败的调用到安全日志
4. 定期审计权限验证逻辑

**文件位置**:
- `service/ipc/include/gamecontroller_server_ability.h:137-150`: 权限验证方法
- `service/common/src/permission_utils.cpp`: 权限工具实现

---

### 5. 竞争条件（TOCTOU）风险

**证据**:
- `frameworks/native/key_mapping/src/key_to_touch_manager.cpp`: FFRT queue 使用
- `frameworks/native/multi_modal_input/src/multi_modal_input_mgt_service.cpp`: mutex 使用
- `service/key_mapping_manager/src/key_mapping_config_manager.cpp`: mutex 使用

**风险描述**:
在异步操作中，如果检查和操作之间存在时间窗口，可能导致 TOCTOU（Time-Of-Check-Time-Of-Use）竞态。例如，检查条件后，状态在操作前被改变。

**触发路径**:
1. 线程 A 检查设备支持转触控（`IsSupportGameKeyMapping()`）
2. 线程 B 同时调用 `EnableGameKeyMapping()` 改变状态
3. 线程 A 基于过时的检查结果继续操作
4. 导致竞态：设备状态与实际不符

**影响**:
- 功能异常（错误启用/禁用）
- 状态不一致
- 潜在的拒绝服务

**修复建议**:
1. 使用原子操作而非 check-then-act 模式
2. 使用 `std::atomic` 或互斥锁保护临界区
3. 考虑使用乐观并发控制（如 compare_exchange）
4. 减少检查和操作之间的时间窗口

**文件位置**:
- `frameworks/native/key_mapping/include/key_mapping_service.h`: IsSupportGameKeyMapping() 定义
- `service/ipc/src/gamecontroller_server_ability.cpp`: EnableGameKeyMapping() 实现

---

### 6. 束指针悬空风险

**证据**:
- `frameworks/capi/src/game_device_proxy.cpp:29`: new(std::nothrow) 调用
- `service/common/src/json_utils.cpp`: malloc/free 使用

**风险描述**:
部分代码使用 C 风格的 `malloc/free` 或手动内存管理，如果异常发生或代码路径错误，可能导致内存泄漏或悬空指针。

**触发路径**:
1. 调用 `OH_GameDevice_GetAllDeviceInfos()` 分配内存
2. 中间发生异常或错误
3. 内存未正确释放或释放多次
4. 后续访问悬空指针

**影响**:
- 内存泄漏（长期运行后内存耗尽）
- 崩溃（访问已释放内存）
- 信息泄露

**修复建议**:
1. 全面使用智能指针（`std::unique_ptr`, `std::shared_ptr`）
2. 移除所有手动 `malloc/free` 调用
3. 使用 RAII 模式确保资源释放
4. 添加内存安全扫描工具到 CI 流程

**文件位置**:
- `frameworks/capi/src/game_device_proxy.cpp:28-36`: 内存分配
- `service/common/src/gamecontroller_utils.cpp`: 需要检查

---

### 7. DoS 拒绝服务风险

**证据**:
- `service/ipc/src/gamecontroller_server_ability.cpp`: 无速率限制

**风险描述**:
IPC 接口缺乏速率限制，恶意应用可以快速连续调用 SA，消耗系统资源或进行暴力攻击。

**触发路径**:
1. 恶意应用循环调用 `SetCustomGameKeyMappingConfig()`
2. 每次调用导致文件 I/O 和 JSON 解析
3. SA 进程 CPU 和内存被耗尽
4. 正常应用无法获得服务

**影响**:
- 拒绝服务
- 系统资源耗尽
- 可用性下降

**修复建议**:
1. 添加调用速率限制（如每秒最多 N 次调用）
2. 使用令牌桶或漏桶算法
3. 超出限制时返回特定错误码（如 GAME_ERR_RATE_LIMIT）
4. 记录异常调用模式到安全日志

**文件位置**:
- `service/ipc/include/gamecontroller_server_ability.h`: 所有 IPC 方法定义
- `service/ipc/src/gamecontroller_server_ability.cpp`: 方法实现

## 安全措施总结

### 已实现的安全措施

| 措施 | 实现位置 | 有效性 |
|------|----------|--------|
| **路径遍历防护** | `service/common/src/json_utils.cpp:135,156,192` | 部分（使用了 realpath，但检查不足）|
| **参数校验** | `frameworks/native/common/include/gamecontroller_keymapping_model.h` | 好（有边界检查）|
| **大小限制** | `frameworks/native/common/include/gamecontroller_keymapping_model.h:28-35` | 好（定义多个 MAX 常量）|
| **Null 检查** | `frameworks/capi/src/*.cpp` | 好（全面检查）|
| **权限分层** | `service/ipc/src/gamecontroller_server_ability.cpp:143-210` | 好（三层权限验证）|
| **编译安全** | 所有 BUILD.gn 文件 | 好（启用多项安全特性）|
| **UTF-8 验证** | `service/common/src/json_utils.cpp:82-130` | 中等（自定义实现）|
| **互斥锁保护** | `service/key_mapping_manager/src/*.cpp` | 好（使用 mutex）|

### 待改进的安全措施

| 措施 | 当前状态 | 优先级 |
|------|----------|--------|
| **速率限制** | 未实现 | 高 |
| **调用链完整性验证** | 未实现 | 高 |
| **路径白名单强制** | 部分（仅有 GAME_CONTROLLER_SERVICE_ROOT）| 中 |
| **JSON 深度限制** | 未实现 | 中 |
| **全面使用智能指针** | 部分（仍有 malloc/free）| 中 |
| **系统 UTF-8 验证** | 未实现（使用自定义 IsUtf8）| 中 |
| **原子操作** | 未实现 | 中 |

## 检查范围与局限性

### 已检查范围

✅ **已检查**:
1. 所有 CAPI 接口的参数校验
2. InnerAPI 权限验证机制
3. JSON 文件读写操作
4. IPC 消息处理
5. 配置文件格式和内容
6. 输入事件处理流程
7. 内存管理模式

### 未检查范围

⚠️ **未检查（限制）**:
1. **运行时行为**：未进行动态分析或模糊测试
2. **并发场景**：仅静态分析，未验证实际竞态
3. **插件系统**：未深入分析插件加载机制
4. **第三方依赖**：未分析 MMI、Window Manager、Bundle Manager 的安全性
5. **密码学强度**：未评估加密或签名机制
6. **社会工程**：未考虑用户交互界面

### 证据范围

本文档所有结论均基于代码证据：
- **文件路径**: 所有引用均包含完整文件路径
- **行号**: 关键代码引用包含行号
- **符号名**: 所有函数、类、变量名均来自代码

**检查时间**: 2026-02-06

## 关键结论

1. **发现 7 类可被利用点**，包括整溢出、JSON 注入、路径遍历、权限绕过、竞态条件、悬空指针、DoS
2. **已实现基础安全措施**（参数校验、大小限制、权限分层、编译保护）
3. **待改进项**（速率限制、调用链完整性、系统 UTF-8 验证、全面使用智能指针）
4. **信任边界清晰**：应用层（不可信）→ Framework（半可信）→ SA（可信）→ 外部服务（可信）
5. **局限**：未进行运行时测试，未深入分析第三方依赖

## 修复优先级建议

| 优先级 | 问题 | 工作量 | 影响范围 |
|---------|------|--------|---------|
| **P0（极高）** | 束指针悬空风险 | 中 | 崩溃、内存泄漏 |
| **P0（极高）** | JSON 解析注入风险 | 低 | 配置篡改 |
| **P1（高）** | DoS 拒绝服务 | 低 | 可用性 |
| **P1（高）** | 权限绕过风险 | 中 | 未授权访问 |
| **P2（中）** | 路径遍历风险增强 | 低 | 写入任意文件 |
| **P2（中）** | 竞态条件 | 中 | 功能异常 |
| **P3（低）** | 全面使用智能指针 | 高 | 长期稳定性 |

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [04_External_CAPI.md](./04_External_CAPI.md) - CAPI 接口安全
- [05_Inner_API.md](./05_Inner_API.md) - InnerAPI 安全
- [03_Architecture.md](./03_Architecture.md) - 架构与信任边界

---

**版本**: 1.0 | **更新时间**: 2026-02-06
