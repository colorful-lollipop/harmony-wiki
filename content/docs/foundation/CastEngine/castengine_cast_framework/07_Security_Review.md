# 安全风险评审

> 文档版本: 1.0.0
> 最后更新: 2026-02-06
> 检查范围: 完整代码库扫描（不含测试）

## 目的

本文档提供 CastEngine 框架的安全风险评估，包括攻击面、信任边界、已发现的潜在漏洞和修复建议。

## 威胁模型概述

### 攻击面清单

CastEngine 框架暴露了多个攻击面：

| 攻击面 | 入口点 | 信任边界 |
|---------|--------|---------|
| N-API（JavaScript） | 所有公共 N-API 方法 | 应用进程 |
| IPC 通信 | 所有 IPC 接口 | 客户端进程 |
| 网络输入 | 设备发现、协议解析 | 未认证网络 |
| 文件访问 | 媒体文件加载 | 本地文件系统 |
| 系统能力 | 权限检查、Surface 设置 | OpenHarmony 系统 |

### 数据流和信任边界

```
[不受信任来源]
    │
    ├── N-API 调用（JavaScript 应用）
    │
    ├── 网络输入（设备发现消息）
    │
    ├── IPC 调用（来自客户端）
    │
    └── 文件输入（媒体文件）
         │
         ▼
┌───────────────────────────────────────┐
│   输入校验层（N-API/IPC）         │
│   • 参数类型检查                          │
│   • 参数范围检查                          │
│   • 权限验证                            │
└──────────┬────────────────────────────┘
           │
           ▼
┌───────────────────────────────────────┐
│   CastEngine 核心逻辑                │
│   • 会话管理                          │
│   • 设备管理                          │
│   • 播放器管理                        │
└──────────┬────────────────────────────┘
           │
           ▼
┌───────────────────────────────────────┐
│   敏感操作（权限保护）            │
│   • 屏幕捕获（MIRROR 权限）         │
│   • 媒体播放（STREAM 权限）          │
│   • IPC 调用（PID 检查）              │
└───────────────────────────────────────┘
```

---

## 权限系统

### 权限定义

**位置**: `service/src/session/src/utils/src/permission.cpp:39-40`

| 权限名称 | 常量 | 用途 | 检查位置 |
|-----------|------|------|----------|
| ACCESS_CAST_ENGINE_MIRROR | MIRROR_PERMISSION | 镜像投屏 | permission.cpp:75-77 |
| ACCESS_CAST_ENGINE_STREAM | STREAM_PERMISSION | 流媒体播放 | permission.cpp:80-83 |

### 权限检查实现

**检查机制**: 使用 OpenHarmony Access Token Kit

```cpp
// service/src/session/src/utils/src/permission.cpp:55-66
bool CheckPermission(const std::string &permission)
{
    AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int result = AccessTokenKit::VerifyAccessToken(callerToken, permission);
    if (result != PERMISSION_GRANTED) {
        CLOGE("%{public}s denied!", GetPermissionDescription(permission).c_str());
        return false;
    }
    return true;
}
```

**关键点**:
1. 通过 `IPCSkeleton::GetCallingTokenID()` 获取调用者令牌
2. 使用 `AccessTokenKit::VerifyAccessToken()` 验证权限
3. 检查结果是否为 `PERMISSION_GRANTED`

### PID 检查机制

**位置**: `service/src/session/src/utils/src/permission.cpp:110-126`

```cpp
bool Permission::CheckPidPermission()
{
    std::lock_guard<std::mutex> lock(pidLock_);
    pid_t pid = IPCSkeleton::GetCallingPid();
    pid_t myPid = getpid();
    if (pid == myPid) {
        return true;  // 自身进程允许
    }
    auto it = std::find_if(pids_.begin(), pids_.end(), [pid](pid_t element) { return element == pid; });
    if (it == pids_.end()) {
        CLOGE("pid(%{public}d) is illegal", pid);
        return false;  // 未授权进程拒绝
    }
    return true;  // 授权进程允许
}
```

**白名单机制**:
- 允许进程列表（`pids_`）维护
- 通过 `SavePid(pid)` 添加授权 PID
- 通过 `RemovePid(pid)` 移除授权 PID
- 非白名单进程的所有操作都会被拒绝

**问题**: 见"风险 1 - PID 白名单绕过"

---

## 输入校验分析

### N-API 参数校验

**位置**: `interfaces/kits/js/src/napi_castengine_utils.cpp`

所有 N-API 方法都使用统一的参数检查：

**类型检查**:
```cpp
template<size_t N>
bool CheckJSParamsType(napi_env env, napi_value *argv, napi_valuetype *expectedTypes)
```

**检查项**:
- ✅ 参数数量校验
- ✅ 参数类型校验（null、number、string、object、function）
- ✅ 类型不匹配时返回错误
- ❌ **未发现**字符串长度上限检查
- ❌ **未发现**字符串内容格式验证（路径遍历）
- ❌ **未发现**特殊字符过滤

**证据**: `interfaces/kits/js/src/napi_castengine_manager.cpp:84-96`

### IPC 参数校验

**Stub 层校验**:
- ✅ IPC 消息读取成功性检查
- ✅ 参数类型反序列化
- ❌ **有限** - 主要依赖类型定义

---

## 已发现的安全风险

### 风险 1: PID 白名单机制可被绕过

**严重性**: 高
**证据**: `service/src/session/src/utils/src/permission.cpp:110-126`

**问题描述**:
PID 白名单检查存在以下问题：
1. 进程可调用 `SavePid(pid)` 将自己添加到白名单
2. 之后可绕过所有权限检查
3. `SavePid` 方法本身没有权限检查
4. 只依赖客户端的善意调用

**可利用路径**:
```
恶意应用
    │
    ├── 1. 调用任何需要权限的 API（被拒绝）
    │
    ├── 2. 探测 IPC 接口，找到 SavePid 方法
    │
    └── 3. 调用 SavePid(getpid()) 将自己加入白名单
         │
         ▼
    ┌─────────────────────┐
    │ 绕过所有权限检查  │
    └─────────────────────┘
```

**影响**:
- 权限机制完全失效
- 恶意应用可执行任何操作
- 可能访问敏感功能（屏幕捕获、媒体播放）

**修复建议**:
1. 移除或加强 `SavePid` 方法的访问控制
2. 要求 SavePid 只能由系统进程调用（通过 UID 检查）
3. 或者完全移除 PID 白名单机制，仅依赖权限 Token
4. 添加 SavePid 调用的日志审计

**参考代码**: `service/src/session/src/utils/src/permission.cpp:85-92`

---

### 风险 2: 网络输入未经验证

**严重性**: 高
**证据**: 设备发现和协议解析代码

**问题描述**:
从网络接收的设备发现消息和协议数据可能包含恶意内容，但缺乏充分验证：

1. **设备发现信息**:
   - `service/src/device_manager/src/discovery_manager.cpp`
   - 设备名称、ID 等可能包含恶意字符串
   - 缺少长度检查

2. **RTSP 消息解析**:
   - `service/src/session/src/rtsp/src/rtsp_parse.cpp`
   - 解析来自网络的 RTSP 消息
   - 缓冲区溢出风险

3. **SoftBus 数据**:
   - `service/src/session/src/channel/src/softbus/softbus_connection.cpp`
   - 接收分布式软总线数据
   - 可能包含格式错误的设备信息

**可利用路径**:
```
恶意设备
    │
    ├── 发送超长设备名称
    │
    ├── 发送格式错误的 RTSP 消息
    │
    └── 尝试触发缓冲区溢出
         │
         ▼
    ┌─────────────────────┐
    │ 解析崩溃或      │
    │ 内存破坏         │
    └─────────────────────┘
```

**影响**:
- 服务崩溃
- 拒绝服务
- 可能的信息泄露（崩溃堆栈）

**修复建议**:
1. 为所有网络输入添加长度上限检查
2. 验证设备名称、ID 等的格式（仅允许安全字符）
3. 对 RTSP 解析使用安全的解析库（避免手动缓冲操作）
4. 添加输入清洗（移除特殊字符、控制字符）
5. 记录所有可疑的输入（便于审计）

**参考代码**:
- `service/src/device_manager/src/discovery_manager.cpp`
- `service/src/session/src/rtsp/src/rtsp_parse.cpp`

---

### 风险 3: 缺少路径遍历防护

**严重性**: 中
**证据**: 媒体文件加载接口

**问题描述**:
StreamPlayer 的 `load(mediaInfo)` 方法接受媒体 URL 或文件路径，但未发现路径遍历检查：

```cpp
// interfaces/kits/js/src/napi_stream_player.cpp - load 方法
// 缺少 "../", "./" 等路径遍历模式检查
```

**可利用路径**:
```
恶意应用
    │
    ├── 加载 "/data/media/../../etc/passwd"
    │
    └── 或加载 "/sdcard/./../private/data"
         │
         ▼
    ┌─────────────────────┐
    │ 读取敏感文件     │
    │ （路径遍历）      │
    └─────────────────────┘
```

**影响**:
- 信息泄露
- 读取系统敏感文件
- 违反应用沙箱边界

**修复建议**:
1. 解析路径并移除路径遍历序列（../, ./）
2. 限制文件访问到应用专属目录
3. 验证文件在允许范围内
4. 使用系统文件 API 的安全版本（自动进行路径规范化）

---

### 风险 4: 权限检查的不一致

**严重性**: 中
**证据**: 多个 N-API 方法

**问题描述**:
部分 N-API 方法存在权限检查不一致：

1. **客户端和服务端都检查**:
   - 客户端 Proxy 在 IPC 调用前检查
   - 服务端 Stub 也检查
   - 双重检查，但逻辑可能不一致

2. **权限覆盖不完整**:
   - 某些操作可能遗漏权限检查
   - 错误码映射可能不完整

**证据**:
```cpp
// client/src/cast_session_impl_proxy.cpp:90 - 客户端返回 ERR_NO_PERMISSION
if (ret != OHOS::SUCCESS) {
    return ERR_NO_PERMISSION;
}

// 但某些场景下，实际错误可能是其他类型
```

**影响**:
- 拒绝合法访问
- 用户体验问题
- 潜在的权限提升

**修复建议**:
1. 统一权限检查逻辑（只在服务端检查）
2. 确保所有敏感操作都有明确的权限要求
3. 审查所有返回 `ERR_NO_PERMISSION` 的代码路径
4. 添加权限审计日志

---

### 风险 5: 共享内存使用

**严重性**: 低
**证据**: `interfaces/inner_api/include/cast_shared_memory*.h`

**问题描述**:
共享内存接口存在以下风险：
1. 缺乏访问控制验证
2. 可能的越界读写
3. 未验证的映射

**共享内存接口**:
- `CastSharedMemory`
- `CastSharedMemoryBase`
- `CastSharedMemoryIpc`

**可利用场景**:
```
恶意应用
    │
    ├── 1. 访问共享内存对象
    │
    ├── 2. 通过偏移量越界读写
    │
    └── 3. 读取其他应用的敏感数据
         │
         ▼
    ┌─────────────────────┐
    │ 内存泄露或      │
    │ 数据破坏         │
    └─────────────────────┘
```

**影响**:
- 跨进程数据泄露
- 内存破坏
- 可能的拒绝服务

**修复建议**:
1. 验证共享内存的大小和偏移量
2. 使用权限控制访问共享内存
3. 实施共享内存的访问审计日志
4. 考虑使用 IPC 代替共享内存（如果可能）

---

## 安全机制

### 1. ASan/Sanitizer 支持

**配置**: 所有 GN targets 启用了 sanitizers

**证据**: 所有 BUILD.gn 文件中的 `sanitize` 配置

```gn
sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
}
```

**启用功能**:
- CFI (Control Flow Integrity) - 控制流完整性
- CFI cross DSO - 跨 DSO 控制流完整性
- 其他 sanitizers（UAF, Address Sanitizer 等）

**覆盖范围**: 所有主要库（.so 文件）

### 2. 编译器安全标志

**根配置**: `BUILD.gn:14-30`

```gn
cflags = [
    "-Wall",
    "-Wextra",
    "-Werror",          # 警告视为错误
    "-Wno-shadow",
    "-Wno-unused-parameter",
    "-Wno-missing-field-initializers",
    "-FS",              # 函数级链接
    "-O2",
    "-D_FORTIFY_SOURCE=2",  # 强化源代码
    "-fvisibility=hidden",        # 隐藏符号
    "-fvisibility-inlines-hidden"
]
```

**安全特性**:
- 严格的编译检查（-Werror）
- FORTIFY_SOURCE 2 - 缓冲区溢出保护
- 符号隐藏 - 减少攻击面

### 3. Branch Protector

**配置**: `branch_protector_ret = "pac_ret"`

**作用**: 返回地址保护，防止 ROP（Return-Oriented Programming）攻击

### 4. 堆栈保护

**推测**: OpenHarmony 标准编译配置包含：
- Stack Canaries
- NX 位（非可执行堆栈）
- ASLR（地址空间布局随机化）

---

## 未覆盖或局限

### 检查范围

✅ 已扫描:
- N-API 实现层（interfaces/kits/js/src/）
- 服务端实现（service/src/）
- 客户端实现（client/src/）
- 权限检查代码
- IPC 接口

❌ 未覆盖:
- 外部协议适配器（DLNA/WiFi Display/Cast+Stream）
  - 这些是独立仓库
- 测试代码
- 第三方依赖（OpenSSL、JSON 等）

### 局限性

1. **协议层安全**: 外部协议库的安全状况未知
2. **依赖漏洞**: 第三方依赖的漏洞未评估
3. **动态加载**: 某些模块可能支持动态加载协议
4. **网络认证**: 设备认证机制未深入分析

---

## 安全最佳实践

### 开发者指南

1. **始终验证输入**:
   - 检查参数类型和范围
   - 限制字符串长度
   - 清洗文件路径

2. **最小权限原则**:
   - 仅申请必要的权限
   - 限制权限的使用范围

3. **使用安全的 API**:
   - 优先使用 OpenHarmony 提供的安全 API
   - 避免不安全的字符串操作（strcpy, sprintf）

4. **错误处理**:
   - 不要吞掉错误
   - 提供有用的错误信息（不泄露内部实现）
   - 记录安全相关事件

5. **审计和日志**:
   - 记录所有安全相关操作
   - 避免记录敏感数据
   - 实施日志级别控制

### 安全测试建议

1. **模糊测试**:
   - 对网络输入进行模糊测试
   - 对 IPC 接口进行模糊测试
   - 对文件解析进行模糊测试

2. **渗透测试**:
   - 尝试绕过权限检查
   - 尝试路径遍历
   - 尝试资源耗尽攻击

3. **静态分析**:
   - 使用静态分析工具扫描代码
   - 检查常见漏洞模式
   - 验证 sanitizers 的有效性

---

## 相关文档

- [N-API 文档](03_N-API.md) - 所有对外接口和权限要求
- [内部 API](04_Internal_API.md) - 权限检查的实现位置
- [架构文档](02_Architecture.md) - 信任边界和数据流

---

**返回**: [SUMMARY.md](SUMMARY.md)
