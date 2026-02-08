# AI Engine 安全风险评审

## 评审范围

本评审覆盖 AI Engine 代码库的以下模块：

| 模块 | 路径 | 评审状态 |
|------|------|----------|
| 客户端 SDK | `services/client/` | ✅ 已评审 |
| 服务端引擎 | `services/server/` | ✅ 已评审 |
| 通信适配层 | `services/*/communication_adapter/` | ✅ 已评审 |
| 插件系统 | `services/server/plugin/` | ✅ 已评审 |
| 平台模块 | `services/common/platform/` | ✅ 已评审 |

**未评审范围**：测试代码、第三方依赖（bounds_checking_function、NNIE SDK）

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      不可信区域                                  │
│  应用进程 ←──────────────────────────────────────────────────→ IPC │
└─────────────────────────────────────────────────────────────────┘
                               ▲
                               │ 边界
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界内                                   │
│  AI Engine 服务端进程                                              │
│  ├── 插件管理器                                                   │
│  ├── 引擎执行器                                                   │
│  └── 插件实例 (IPlugin)                                          │
└─────────────────────────────────────────────────────────────────┘
```

### 边界定义

| 边界 | 说明 |
|------|------|
| IPC 边界 | 跨进程调用，数据需要序列化/反序列化 |
| 插件边界 | 动态加载的 .so 文件，信任度低 |
| 共享内存边界 | 大数据传输通道，依赖 UID 权限 |

**证据**：`services/common/protocol/struct_definition/aie_info_define.h:25-35` UID 定义

---

## 攻击面清单

| 攻击面 | 类型 | 说明 |
|--------|------|------|
| IPC 接口 | 网络/进程 | SAMGR 通信，可能被恶意应用调用 |
| 插件加载 | 文件 | 动态加载 .so，可能加载恶意插件 |
| 共享内存 | 文件/内存 | 大数据传输，可能被篡改 |
| 配置解析 | 文件 | ai_engine_plugin.ini 配置解析 |
| 模型加载 | 文件 | .wk 模型文件解析 |

---

## 风险点与修复建议

### 高风险

#### 1. 插件代码执行

**证据**：`services/server/plugin_manager/source/plugin_manager.cpp:68-95`

```cpp
void *handle = dlopen(pluginPath.c_str(), RTLD_NOW);
if (handle == nullptr) {
    HILOGE("dlopen failed: %s", dlerror());
    return RETCODE_FAILURE;
}
IPLUGIN_INTERFACE pluginEntry = (IPLUGIN_INTERFACE)dlsym(handle, "PLUGIN_INTERFACE");
```

**风险**：插件路径可配置，恶意 .so 可执行任意代码

**触发条件**：
1. 攻击者修改 `ai_engine_plugin.ini` 配置路径
2. 替换插件 .so 文件为恶意库
3. 重启 ai_server

**影响**：完全控制服务端进程权限

**修复建议**：
- 插件签名验证：加载前校验 .so 数字签名
- 插件路径白名单：限制可加载插件的路径范围
- 代码签名策略：使用 SELinux/AppArmor 限制 dlopen

---

### 中风险

#### 2. 共享内存权限验证缺失

**证据**：`services/common/platform/os_wrapper/ipc/source/aie_ipc.cpp:96`

```cpp
shmidDs.shm_perm.uid = receiverUid; // give receiver the privilege
```

**风险**：仅依赖接收方 UID 校验，无完整性校验

**触发条件**：
1. 恶意进程猜测 shmId
2. 直接 attach 共享内存
3. 读取或篡改推理数据

**影响**：数据泄露、推理结果篡改

**修复建议**：
- 添加 HMAC 签名：共享内存数据添加校验码
- 双向认证：发送方验证接收方身份
- 随机化 shmId：避免猜测

#### 3. 输入验证不完整

**证据**：`services/common/protocol/data_channel/source/request.cpp:74-79`

```cpp
uid_t IRequest::GetClientUid() const
{
    return clientUid_;
}
```

**风险**：直接使用客户端提供的 clientUid，未验证来源

**触发条件**：
1. 伪造 IPC 请求
2. 设置任意 clientUid
3. 绕过 UID 检查

**影响**：权限提升、访问其他客户端数据

**修复建议**：
- 内核级别验证：从 IPC 元数据获取真实 UID
- 请求来源追踪：记录请求发送方进程信息

---

#### 4. 序列化漏洞

**证据**：`services/common/utils/encdec/source/data_encoder.cpp`

```cpp
int DataEncoder::EncodeData(const unsigned char *data, int len)
{
    if (data == nullptr || len <= 0) {
        return RETCODE_FAILURE;
    }
    // ... 编码逻辑
}
```

**风险**：边界检查不完整可能导致整数溢出

**触发条件**：
1. 发送超长序列化数据
2. 触发缓冲区溢出

**影响**：拒绝服务、代码执行

**修复建议**：
- 严格长度校验：len 必须小于 MAX_BUFFER_SIZE
- 使用安全函数：memcpy_s, strncpy_s
- 模糊测试：增加序列化代码的 fuzzing 测试

---

### 低风险

#### 5. 日志信息泄露

**证据**：多处使用 HILOGE/HILOGI 输出调试信息

```cpp
HILOGE("[KWSSdkImpl]AieClientInit failed. Error code[%d]", retCode);
```

**风险**：错误日志可能泄露内部路径、变量值

**触发条件**：
1. 日志级别设置为 DEBUG
2. 恶意应用获取日志

**影响**：信息收集、辅助攻击

**修复建议**：
- 日志脱敏：敏感信息（UID、路径）输出为 ***
- 日志分级：生产环境默认 INFO 及以上
- 日志隔离：不同客户端日志分开存储

#### 6. 空指针解引用

**证据**：多处使用裸指针，未统一检查

```cpp
int32_t KWSRetCode KWSSdkImpl::Create()
{
    if (kwsHandle_ != INVALID_KWS_HANDLE) {
        return KWS_RETCODE_FAILURE;
    }
    // ...
}
```

**风险**：插件实现不当可能导致空指针崩溃

**触发条件**：
1. 插件 Prepare 返回空 outputInfo
2. 后续 SyncProcess 解引用

**影响**：拒绝服务（进程崩溃）

**修复建议**：
- 统一空指针检查框架
- 插件接口增加空指针契约检查
- 崩溃监控与自动恢复

---

## 安全最佳实践

### 插件开发安全规范

1. **输入验证**：所有外部输入必须验证
2. **错误处理**：不泄露内部细节，返回通用错误码
3. **资源释放**：确保所有资源在 Release 中释放
4. **线程安全**：插件实现必须线程安全

### 服务端部署安全规范

1. **最小权限**：ai_server 以低权限用户运行
2. **文件权限**：插件目录仅服务可读写
3. **网络隔离**：禁用不必要的网络能力
4. **日志审计**：记录所有 IPC 调用

---

## 总结

| 风险等级 | 数量 | 主要风险 |
|----------|------|----------|
| 高 | 1 | 插件代码执行 |
| 中 | 3 | 共享内存安全、输入验证、序列化 |
| 低 | 2 | 日志泄露、空指针 |

**总体评估**：AI Engine 存在潜在安全风险，主要集中在插件加载和跨进程通信环节。建议在高安全场景下增加插件签名验证和 IPC 双向认证机制。
