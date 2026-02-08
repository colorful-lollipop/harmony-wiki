# 安全风险评审

## 概述

本文档分析 distributed_audio 组件的安全风险，包括攻击面、信任边界和可利用点。

## 攻击面分析

### 1. IPC 接口攻击面

**接口**: `IDAudioSource` (SA 4805), `IDAudioSink` (SA 4806)

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 未授权调用 | 高 | 恶意应用尝试调用 IPC 接口 |
| 参数注入 | 中 | JSON 参数注入攻击 |
| 接口令牌伪造 | 中 | 尝试伪造 IPC 接口令牌 |
| **DAudioNotify 无权限** | **高** | `DAudioNotifyInner` 接口缺少权限检查 |
| **权限逻辑错误** | **高** | `UpdateWorkMode` 权限检查逻辑反转 |

**防护措施**:
```cpp
// services/audiomanager/servicesource/src/daudio_source_stub.cpp:55-92
int DAudioSourceStub::OnRemoteRequest(...) {
    // 1. 接口令牌验证
    if (descriptor != GetDescriptor()) {
        return ERR_DH_AUDIO_SA_INVALID_INTERFACE_TOKEN;
    }
    
    // 2. 权限检查
    if (!VerifyPermission()) {
        return ERR_DH_AUDIO_SA_PERMISSION_CHECK_FAIL;
    }
    // ...
}

bool DAudioSourceStub::VerifyPermission() {
    AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t result = AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, AUDIO_PERMISSION_NAME);
    return result == AccessToken::PERMISSION_GRANTED;
}
```

**⚠️ 已知缺陷**:
- `DAudioNotifyInner` 方法（Source 和 Sink）**未调用** `VerifyPermission()`
- `UpdateDAudioWorkModeInner` 方法使用了错误的逻辑 `!VerifyPermission()`

### 2. 输入验证攻击面

**数据来源**: IPC 调用、软总线消息、HDF 回调

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| JSON 炸弹 | 中 | 超大 JSON 数据消耗内存 |
| 格式字符串 | 低 | 字符串格式化漏洞 |
| 整数溢出 | 低 | 大小参数溢出 |

**防护措施**:
```cpp
// common/src/daudio_util.cpp:279-360
bool CJsonParamCheck(cJSON *jsonObj, const ParamInfo *paramInfos, int32_t paramNum) {
    // 验证 JSON 参数类型和范围
}

// services/audiomanager/managersource/src/daudio_source_dev.cpp
if (event.content.length() > DAUDIO_MAX_JSON_LEN) {
    // 拒绝超大 JSON
    return ERR_DH_AUDIO_SA_PARAM_INVALID;
}
```

### 3. 内存操作攻击面

**数据来源**: 音频数据缓冲区、共享内存

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 缓冲区溢出 | 高 | 音频数据拷贝溢出 |
| 共享内存攻击 | 中 | ashmem 滥用 |
| 内存泄漏 | 低 | 长期运行内存增长 |

**防护措施**:
```cpp
// common/src/daudio_ringbuffer.cpp:101,158
int32_t ret = memcpy_s(array_ + writePos_, len, data, len);
CHECK_AND_RETURN_RET_LOG(ret != EOK, ERR_DH_AUDIO_FAILED, "memcpy_s error.");

// services/audiomanager/managersource/src/dspeaker_dev.cpp:421
if (ashmemLength >= ASHMEM_MAX_LEN) {
    return ERR_DH_AUDIO_FAILED;
}
```

### 4. 文件系统攻击面

**数据来源**: 调试转储文件

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 路径遍历 | 中 | 文件名包含 ../ |
| 文件覆盖 | 低 | 写入系统文件 |

**防护措施**:
```cpp
// common/src/daudio_util.cpp:537-551
std::string SaveFile(const std::string &fileName, ...) {
    if (fileName.length() > PATH_MAX) {
        return "";
    }
    // 使用 realpath 解析规范路径
    char resolvedPath[PATH_MAX];
    if (realpath(fileName.c_str(), resolvedPath) == nullptr) {
        return "";
    }
    // ...
}
```

### 5. 网络/传输攻击面

**数据来源**: 软总线传输

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 中间人攻击 | 中 | 传输数据篡改 |
| 重放攻击 | 中 | 重复发送控制指令 |
| 拒绝服务 | 中 | 洪水攻击 |

**防护措施**:
- 软总线层提供加密和认证
- 设备间需要建立可信关系
- ACL 访问控制验证

## 信任边界

### 边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界分析                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│   │   应用层     │────▶│  音频框架    │────▶│   HDF 驱动   │      │
│   │  (不信任)   │     │  (半信任)   │     │  (半信任)   │      │
│   └─────────────┘     └─────────────┘     └──────┬──────┘      │
│                                                   │              │
│                    信任边界 1: HDF 接口           │              │
│                                                   ▼              │
│                                          ┌─────────────┐        │
│                                          │  音频管理器  │        │
│                                          │ (核心服务)  │        │
│                                          └──────┬──────┘        │
│                                                 │                │
│                    信任边界 2: IPC 接口        │                │
│                                                 ▼                │
│                                          ┌─────────────┐        │
│                                          │  软总线传输  │        │
│                                          │ (跨设备)   │        │
│                                          └──────┬──────┘        │
│                                                 │                │
│                    信任边界 3: 设备边界        │                │
│                                                 ▼                │
│                                          ┌─────────────┐        │
│                                          │  远端设备   │        │
│                                          └─────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 信任边界检查点

| 边界 | 位置 | 检查内容 |
|------|------|----------|
| **IPC 边界** | `DAudioSourceStub::OnRemoteRequest()` | 接口令牌、调用者权限 |
| **设备访问** | `DAudioSourceDev::CheckAclRight()` | ACL 访问控制 |
| **JSON 输入** | `CJsonParamCheck()` | 参数类型、长度 |
| **内存操作** | 所有 `memcpy_s` 调用 | 目标缓冲区大小 |
| **文件操作** | `SaveFile()` | 路径规范化 |

## 可利用点分析

### 1. IPC 接口未授权访问

**位置**: `services/audiomanager/servicesource/src/daudio_source_stub.cpp:84-92`

**证据**:
```cpp
bool DAudioSourceStub::VerifyPermission() {
    AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t result = AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, OHOS_PERMISSION_DISTRIBUTED_DATASYNC);
    return result == AccessToken::PERMISSION_GRANTED;
}
```

**触发条件**: 调用 IPC 接口时

**影响**: 高 - 可控制分布式音频设备

**修复建议**: ✅ 已实现权限检查

### 2. JSON 参数长度溢出

**位置**: `services/audiomanager/managersource/src/daudio_source_dev.cpp:588,775,855,873,928,976`

**证据**:
```cpp
if (event.content.length() > DAUDIO_MAX_JSON_LEN) {
    DHLOGE("Content length exceeds maximum.");
    return ERR_DH_AUDIO_SA_PARAM_INVALID;
}
```

**触发条件**: 传入超大 JSON 数据

**影响**: 中 - 内存消耗或解析性能问题

**修复建议**: ✅ 已实现长度限制

### 3. 设备 ID 格式验证

**位置**: `common/src/daudio_util.cpp:436-450`

**证据**:
```cpp
bool CheckDevIdIsLegal(const std::string &devId) {
    if (devId.length() > DAUDIO_MAX_DEVICE_ID_LEN || devId.empty()) {
        return false;
    }
    // 验证字符范围
    for (const auto &c : devId) {
        if (!isalnum(c) && c != '_' && c != '-') {
            return false;
        }
    }
    return true;
}
```

**触发条件**: 传入非法设备 ID

**影响**: 低 - 可能导致逻辑错误

**修复建议**: ✅ 已实现格式验证

### 4. 共享内存大小验证

**位置**: `services/audiomanager/managersource/src/dspeaker_dev.cpp:421`

**证据**:
```cpp
if (ashmemLength >= ASHMEM_MAX_LEN) {
    DHLOGE("Ashmem length exceeds maximum.");
    return ERR_DH_AUDIO_FAILED;
}
```

**触发条件**: 创建超大共享内存

**影响**: 中 - 内存耗尽

**修复建议**: ✅ 已实现大小限制

### 5. 文件路径遍历

**位置**: `common/src/daudio_util.cpp:540`

**证据**:
```cpp
if (fileName.length() > PATH_MAX) {
    return "";
}
char resolvedPath[PATH_MAX];
if (realpath(fileName.c_str(), resolvedPath) == nullptr) {
    return "";
}
```

**触发条件**: 调试转储文件操作

**影响**: 中 - 可能覆盖系统文件

**修复建议**: ✅ 已使用 realpath 规范化

### 6. 内存拷贝安全

**位置**: `common/src/daudio_ringbuffer.cpp:101,158`

**证据**:
```cpp
int32_t ret = memcpy_s(array_ + writePos_, len, data, len);
CHECK_AND_RETURN_RET_LOG(ret != EOK, ERR_DH_AUDIO_FAILED, "memcpy_s error.");
```

**触发条件**: 音频数据拷贝

**影响**: 高 - 缓冲区溢出

**修复建议**: ✅ 已使用 memcpy_s 安全拷贝

### 7. 动态库加载

**位置**: `services/audiomanager/managersource/src/daudio_source_manager.cpp:514`

**证据**:
```cpp
void *handler = dlopen("libdistributed_av_sender.z.so", RTLD_LAZY | RTLD_NODELETE);
```

**触发条件**: 加载 AV 引擎库

**影响**: 中 - 可能加载恶意库

**修复建议**: ⚠️ 建议验证库签名

### 8. DAudioNotifyInner 接口缺少权限检查

**位置**: 
- Source: `services/audiomanager/servicesource/src/daudio_source_stub.cpp:174-183`
- Sink: `services/audiomanager/servicesink/src/daudio_sink_stub.cpp:166-175`

**证据**:
```cpp
int32_t DAudioSourceStub::DAudioNotifyInner(MessageParcel &data, MessageParcel &reply, 
    MessageOption &option)
{
    // ⚠️ 无权限检查！直接读取参数
    std::string networkId = data.ReadString();
    std::string dhId = data.ReadString();
    int32_t eventType = data.ReadInt32();
    std::string eventContent = data.ReadString();
    
    DAudioNotify(networkId, dhId, eventType, eventContent);
    return DH_SUCCESS;
}
```

**触发条件**: 任何进程均可调用 DAudioNotify IPC 接口

**影响**: **高** - 恶意应用可伪造音频事件，干扰分布式音频状态机，可能导致：
- 音频设备异常断开
- 伪造音量/焦点事件
- 干扰正常音频传输

**触发路径**:
```
恶意应用 -> IPC 调用 (DAUDIO_NOTIFY) -> DAudioNotifyInner() -> 
  -> DAudioNotify() -> 处理伪造事件 -> 设备状态异常
```

**修复建议**: 🚨 **必须添加权限检查**
```cpp
int32_t DAudioSourceStub::DAudioNotifyInner(MessageParcel &data, MessageParcel &reply,
    MessageOption &option)
{
    // 添加权限检查
    if (!VerifyPermission()) {
        DHLOGE("Permission verification failed for DAudioNotify.");
        return ERR_DH_AUDIO_SA_PERMISSION_FAIED;
    }
    // ... 原有逻辑
}
```

### 9. UpdateDAudioWorkModeInner 权限检查逻辑错误

**位置**: `services/audiomanager/servicesource/src/daudio_source_stub.cpp:188-189`

**证据**:
```cpp
int32_t DAudioSourceStub::UpdateDAudioWorkModeInner(MessageParcel &data, MessageParcel &reply,
    MessageOption &option)
{
    int32_t ret = 0;
    do {
        // 🐛 BUG: 使用了 !VerifyPermission()，逻辑完全相反！
        CHECK_AND_RETURN_RET_LOG(!VerifyPermission(), ERR_DH_AUDIO_SA_PERMISSION_FAIED,
            "Permission verification fail.");
        // ...
    } while (0);
    // ...
}
```

**触发条件**: 调用 UpdateWorkMode IPC 接口

**影响**: **高** - 权限检查逻辑完全反转：
- 有权限的调用者会被拒绝（返回 PERMISSION_FAIED）
- 无权限的调用者会被允许继续执行

**后果**: 未授权应用可修改分布式音频工作模式（共享内存参数、场景设置等）

**修复建议**: 🚨 **立即修复逻辑错误**
```cpp
// 删除 ! 运算符
CHECK_AND_RETURN_RET_LOG(VerifyPermission(), ERR_DH_AUDIO_SA_PERMISSION_FAIED,
    "Permission verification fail.");
```

## 安全建议

### 1. 已实施的良好实践

✅ IPC 接口权限检查  
✅ JSON 参数长度限制  
✅ 设备 ID 格式验证  
✅ 共享内存大小限制  
✅ 文件路径规范化  
✅ 安全内存拷贝（memcpy_s）  
✅ 边界检查编译选项  
✅ CFI（控制流完整性）保护  

### 2. 建议改进

🚨 **立即修复（高危）**

**DAudioNotifyInner 权限缺失**
- 当前: `DAudioNotifyInner` 接口无权限检查
- 建议: 添加 `VerifyPermission()` 检查，与其他 IPC 接口保持一致

**UpdateDAudioWorkModeInner 逻辑错误**
- 当前: `!VerifyPermission()` 逻辑完全反转
- 建议: 删除 `!` 运算符，修正为 `VerifyPermission()`

⚠️ **动态库签名验证**
- 当前: 直接 dlopen 加载库
- 建议: 验证库文件签名后再加载

⚠️ **更细粒度的权限控制**
- 当前: 单一 DISTRIBUTED_DATASYNC 权限
- 建议: 区分播放/录音/控制权限

⚠️ **输入数据白名单**
- 当前: 黑名单过滤（长度检查）
- 建议: JSON Schema 验证

⚠️ **审计日志**
- 当前: 仅错误日志
- 建议: 安全事件审计日志

## 安全检查清单

### 代码审查检查项

- [ ] 所有 IPC 入口是否有权限检查
- [ ] 所有字符串输入是否有长度检查
- [ ] 所有内存拷贝是否使用安全函数
- [ ] 所有文件操作是否规范化路径
- [ ] 所有 JSON 解析是否有异常处理
- [ ] 所有动态库加载是否验证来源

### 运行时检查项

- [ ] 权限配置是否正确
- [ ] SELinux 策略是否生效
- [ ] 进程运行用户是否正确（daudio）
- [ ] 文件权限是否正确

## 参考

- [OpenHarmony 安全开发指南](https://gitee.com/openharmony/docs)
- [IPC 安全最佳实践](https://gitee.com/openharmony/ability_ability_runtime)
- [SELinux 策略配置](https://gitee.com/openharmony/security_selinux)

---

*文档生成时间: 2025-02-06*
