# 安全风险评审

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      不可信区域                                 │
│  - 应用进程 (JS/Native)                                         │
│  - 用户输入 (URL, 文件路径, 参数)                               │
│  - 网络数据 (流媒体)                                            │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ IPC 边界
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      可信区域                                   │
│  - Media Service (SA)                                           │
│  - 引擎实现 (HiStreamer, LPP)                                   │
│  - 系统资源 (文件, 音频设备, 摄像头)                            │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流

1. **用户输入** → N-API 参数
2. **N-API** → IPC Proxy → SA Stub
3. **SA Stub** → 引擎处理
4. **引擎** → 系统资源 (文件/设备)

## 攻击面分析

### 1. N-API 参数注入

**风险等级**: 中

**描述**: 恶意应用通过 N-API 接口传入恶意参数

**受影响模块**: 所有 N-API 接口

**代码证据**:
```cpp
// 文件: frameworks/js/player/audio_player_napi.cpp:231
uint64_t tokenId = IPCSkeleton::GetSelfTokenID();
// 参数校验在 N-API 层进行
```

**触发场景**:
- 传入非法文件描述符
- 传入恶意 URL
- 构造超长字符串缓冲区

**防护措施**:
- N-API 层参数校验
- IPC 序列化边界检查
- 服务端二次校验

**修复建议**:
1. 在 N-API 层增加参数白名单校验
2. 对字符串参数进行长度限制和字符过滤
3. 对文件描述符进行有效性检查

### 2. 权限绕过

**风险等级**: 高

**描述**: 未授权应用访问敏感媒体资源

**代码证据**:
```cpp
// 文件: services/utils/media_permission.cpp:31-40
int32_t MediaPermission::CheckMicPermission()
{
    auto callerUid = IPCSkeleton::GetCallingUid();
    if (callerUid == ROOT_UID) {
        return PERMISSION_GRANTED;
    }
    Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
    return Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenCaller, "ohos.permission.MICROPHONE");
}
```

**触发场景**:
- 未声明权限直接调用
- 伪造 Token ID
- Root 用户绕过检查

**修复建议**:
1. 增加 Token ID 有效性校验
2. 完善 Root 用户的审计日志
3. 增加权限状态变更通知机制

### 3. IPC 通信劫持

**风险等级**: 中

**描述**: 跨进程通信过程中的数据篡改

**代码证据**:
```cpp
// 文件: services/services/player/ipc/player_service_stub.cpp:311
int PlayerServiceStub::OnRemoteRequest(uint32_t code, MessageParcel &data, MessageParcel &reply, ...)
{
    // 从 data 读取参数
    sptr<IRemoteObject> object = data.ReadRemoteObject();
    // 无额外签名验证
}
```

**触发场景**:
- 恶意应用伪造 IPC 请求
- 中间人攻击
- 参数序列化/反序列化漏洞

**修复建议**:
1. 增加 IPC 请求签名验证
2. 使用 MessageParcel::Write/read 时的边界检查
3. 增加 IPC 调用的审计日志

### 4. 资源耗尽 (DoS)

**风险等级**: 中

**描述**: 恶意应用耗尽系统媒体资源

**受影响模块**:
- 播放器实例
- 录制器实例
- 文件描述符
- 内存缓冲区

**代码证据**:
```cpp
// 文件: services/services/player/client/player_client.cpp:64
sptr<IRemoteObject> object = listenerStub_->AsObject();
// 未见明确的资源限制逻辑
```

**触发场景**:
- 创建大量播放器实例
- 打开大量文件
- 分配超大内存缓冲区

**修复建议**:
1. 增加实例数量限制
2. 对文件描述符进行配额管理
3. 对内存使用设置上限

### 5. 路径遍历

**风险等级**: 中

**描述**: 恶意应用通过路径遍历访问受限文件

**触发场景**:
- 传入 `../../../etc/passwd` 类型的路径
- 使用符号链接绕过路径检查

**防护措施**:
- OpenHarmony 文件系统沙箱
- 权限检查

**修复建议**:
1. 在 N-API 层进行路径标准化
2. 使用 realpath() 解析符号链接
3. 检查最终路径是否在允许范围内

### 6. 缓冲区溢出

**风险等级**: 低

**描述**: 可能的缓冲区溢出风险

**代码证据**:
```cpp
// 文件: services/services/sa_media/ipc/media_service_stub.cpp:67
int32_t callingUid = IPCSkeleton::GetCallingUid();
// 使用标准 IPC 框架，内存安全
```

**触发场景**:
- 字符串操作
- 数组访问

**防护措施**:
- 使用 C++ 标准库
- 编译器安全选项

**修复建议**:
1. 尽可能使用现代 C++ 安全特性
2. 增加边界检查

### 7. 信息泄露

**风险等级**: 低

**描述**: 媒体元数据可能包含敏感信息

**受影响模块**:
- AVMetadataExtractor
- 文件头解析

**修复建议**:
1. 限制敏感元数据的访问权限
2. 对敏感信息进行脱敏处理

### 8. 拒绝服务攻击

**风险等级**: 中

**描述**: 恶意媒体文件导致服务崩溃

**触发场景**:
- 格式错误的媒体文件
- 超大文件头
- 恶意编解码参数

**代码证据**:
```cpp
// 文件: services/engine/histreamer/player/hiplayer_impl.cpp
// 引擎层处理文件解析
```

**修复建议**:
1. 文件解析增加超时机制
2. 限制单个文件处理的最大资源
3. 增加文件格式白名单

## 安全相关代码位置

| 功能 | 文件 | 行号 |
|------|-----|------|
| 权限检查 | `services/utils/media_permission.cpp` | 31-68 |
| IPC 调用者识别 | `services/services/player/server/player_server.cpp` | 110-114 |
| Token 验证 | `frameworks/js/avplayer/avplayer_napi.cpp` | 231 |
| 录音权限 | `frameworks/js/player/audio_player_napi.cpp` | 226 |

## 权限清单

| 权限名称 | 用途 | 保护等级 |
|---------|------|---------|
| ohos.permission.MICROPHONE | 麦克风录音 | normal |
| ohos.permission.RECORD_VOICE_CALL | 通话录音 | system_basic |
| ohos.permission.CAPTURE_SCREEN | 屏幕录制 | system_basic |
| ohos.permission.READ_MEDIA | 读取媒体文件 | normal |
| ohos.permission.WRITE_MEDIA | 写入媒体文件 | normal |
| ohos.permission.INTERNET | 网络访问 | normal |

## 安全最佳实践

### 开发者建议

1. **最小权限原则**: 只申请必要的权限
2. **参数校验**: 永远不要信任用户输入
3. **资源限制**: 设置合理的资源使用上限
4. **错误处理**: 妥善处理所有异常情况
5. **日志记录**: 记录关键安全事件

### 代码审查清单

- [ ] 所有 N-API 参数是否经过校验？
- [ ] 权限检查是否在关键路径执行？
- [ ] IPC 通信是否有足够的保护？
- [ ] 资源使用是否有上限控制？
- [ ] 敏感操作是否有审计日志？

## 相关文档

- [N-API 接口总览](05_NAPI_Overview.md)
- [故障排查指南](appendix/Troubleshooting.md)
