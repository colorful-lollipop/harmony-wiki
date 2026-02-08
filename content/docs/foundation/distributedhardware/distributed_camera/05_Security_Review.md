# 安全风险评审

## 审计范围

| 范围 | 包含 | 不包含 |
|------|------|--------|
| **代码范围** | `common/`, `interfaces/`, `services/`, `channel/`, `data_process/` | `test/`, `wiki/` |
| **外部依赖** | cJSON, softbus, camera_framework | 下游 HDF 驱动 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不可信区域                               │
│    外部应用 / 网络攻击者 / 恶意设备                              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                     IPC 接口边界                                 │
│    DistributedCameraSourceStub / DistributedCameraSinkStub       │
│    - InterfaceToken 校验                                        │
│    - AccessToken 权限校验                                       │
│    - 参数长度校验 (DID_MAX_SIZE/PARAM_MAX_SIZE)                   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                     半可信区域                                    │
│    分布式相机服务进程 (dcamera)                                  │
│    - SELinux 隔离 (u:r:dcamera:s0)                              │
│    - Token 传递链                                                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                     网络传输边界                                 │
│    SoftBus 通道                                                  │
│    - 数据包完整性校验                                            │
│    - 软总线层加密                                                │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│                       可信区域                                   │
│    Camera Framework / HDF / 硬件                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 攻击面清单

### 1. IPC 接口越权调用

| 风险等级 | 高 |
|----------|-----|
| **位置** | `*Stub::OnRemoteRequest()` |
| **触发** | 无权限应用通过 binder 调用特权接口 |
| **证据** | `distributed_camera_source_stub.cpp:47-54` |
| **防护** | `HasEnableDHPermission()` + AccessTokenKit 校验 |

**代码证据**：
```cpp
// distributed_camera_source_stub.cpp:47-54
int32_t DistributedCameraSourceStub::HasEnableDHPermission()
{
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}
```

**修复建议**：防护措施已完善，建议定期审查权限声明的最小化。

---

### 2. IPC 参数注入

| 风险等级 | 中 |
|----------|-----|
| **位置** | `*Inner()` 方法 |
| **触发** | 超长字符串 / 畸形数据 |
| **证据** | `distributed_camera_constants.h: DID_MAX_SIZE=256, PARAM_MAX_SIZE=4096` |
| **防护** | 长度检查 + 空值检查 |

**代码证据**：
```cpp
// distributed_camera_constants.h
const size_t DID_MAX_SIZE = 256;
const size_t PARAM_MAX_SIZE = 4096;

// 校验示例
if (devId.empty() || devId.size() > DID_MAX_SIZE) {
    return DCAMERA_BAD_VALUE;
}
```

**修复建议**：防护措施已完善，建议增加格式校验（如设备 ID 格式）。

---

### 3. 跨设备数据包伪造

| 风险等级 | 中 |
|----------|-----|
| **位置** | `dcamera_softbus_session.cpp` |
| **触发** | 中间人攻击 / 数据篡改 |
| **证据** | `dcamera_softbus_session.cpp` 数据包分片处理 |
| **防护** | 数据包校验 + 软总线层加密 |

**代码证据**：
```cpp
// 数据包组装校验
int32_t DCameraSoftbusSession::CheckUnPackBuffer(const char* data, int32_t dataLen)
{
    if (data == nullptr || dataLen <= 0 || dataLen > DCAMERA_MAX_RECV_DATA_LEN) {
        return DCAMERA_BAD_VALUE;
    }
    // ... 分片校验逻辑
}
```

**修复建议**：
1. 建议增加数据包源验证（会话绑定设备 ID）
2. 建议增加数据完整性 MAC 校验

---

### 4. 缓冲区溢出

| 风险等级 | 低 |
|----------|-----|
| **位置** | `decode_data_process.cpp` |
| **触发** | 视频帧数据过大 |
| **证据** | `MAX_YUV420_BUFFER_SIZE` 限制 |
| **防护** | 缓冲区大小检查 + `memcpy_s` |

**代码证据**：
```cpp
// 解码数据校验
if (bufferSize > MAX_YUV420_BUFFER_SIZE) {
    DHLOGE("Buffer size exceeds limit");
    return DCAMERA_BAD_VALUE;
}

// 使用安全的内存拷贝
memcpy_s(destBuffer, destCapacity, srcBuffer, srcSize);
```

**修复建议**：防护措施已完善，建议持续进行 Fuzz 测试。

---

### 5. 文件路径遍历

| 风险等级 | 低 |
|----------|-----|
| **位置** | `dcamera_utils_tools.cpp:DumpBufferToFile()` |
| **触发** | 恶意文件路径注入 |
| **证据** | Dump 文件写入逻辑 |
| **防护** | `realpath` 校验 + DUMP_PATH 白名单 |

**代码证据**：
```cpp
// 路径校验
char resolvedPath[PATH_MAX] = {0};
if (realpath(filePath.c_str(), resolvedPath) == nullptr) {
    DHLOGE("Invalid file path");
    return DCAMERA_BAD_VALUE;
}

if (strncmp(resolvedPath, DUMP_PATH, strlen(DUMP_PATH)) != 0) {
    DHLOGE("Path outside allowed directory");
    return DCAMERA_BAD_VALUE;
}
```

**修复建议**：防护措施已完善，建议仅在 debug 版本启用 dump 功能。

---

### 6. 动态库加载

| 风险等级 | 低 |
|----------|-----|
| **位置** | `allconnect_manager.cpp:dlopen()` |
| **触发** | 恶意 so 注入 |
| **证据** | AllConnect 动态库加载 |
| **防护** | 路径限制在 `/system/lib/` |

**代码证据**：
```cpp
// 动态库加载
void* handle = dlopen("libcfwk_allconnect_client.z.so", RTLD_NOW);
if (handle == nullptr) {
    DHLOGE("Failed to load allconnect library: %{public}s", dlerror());
    return DCAMERA_ERR_DLOPEN;
}
```

**修复建议**：防护措施已完善，建议使用 `dlopen` 前进行文件完整性校验。

---

### 7. 隐私信息泄露

| 风险等级 | 低 |
|----------|-----|
| **位置** | 日志输出点 |
| **触发** | 设备 ID / Token 泄露 |
| **证据** | 多处使用 `GetAnonyString()` |
| **防护** | 匿名化处理 |

**代码证据**：
```cpp
// 日志脱敏
std::string GetAnonyString(const std::string& str)
{
    if (str.length() <= ANONY_LEN) {
        return "****";
    }
    return str.substr(0, FRONT_LEN) + "****" + str.substr(str.length() - BACK_LEN);
}

// 使用
DHLOGI("DeviceId: %{public}s", GetAnonyString(devId).c_str());
```

**修复建议**：建议审计所有日志点，确保敏感信息已脱敏。

---

## 权限声明

| 权限 | 声明位置 | 用途 | 最小化评估 |
|------|----------|------|------------|
| `ohos.permission.ACCESS_SERVICE_DM` | dcamera.cfg | 设备管理访问 | 必要 |
| `ohos.permission.DISTRIBUTED_DATASYNC` | dcamera.cfg | 数据同步 | 必要 |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | dcamera.cfg | 软总线中心 | 必要 |
| `ohos.permission.CAMERA` | dcamera.cfg | 相机访问 | 必要 |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | dcamera.cfg | 分布式硬件访问 | 必要 |

**评估结论**：权限声明符合最小权限原则。

---

## 安全机制总结

### 已实现的安全机制

| 机制 | 实现位置 | 效果 |
|------|----------|------|
| **IPC 权限校验** | Stub 文件 | 防止越权调用 |
| **参数长度校验** | Proxy/Stub | 防止注入攻击 |
| **缓冲区边界检查** | DataProcess | 防止溢出 |
| **安全内存操作** | 全局 | 防止内存破坏 |
| **SELinux 隔离** | dcamera.cfg | 进程级隔离 |
| **日志脱敏** | dh_log.h | 防止信息泄露 |
| **Fuzz 测试** | test/ | 持续安全测试 |

### 建议改进

| 优先级 | 建议 | 理由 |
|--------|------|------|
| **低** | 增加数据包源验证 | 增强跨设备安全 |
| **低** | 动态库加载前校验 | 防御供应链攻击 |
| **低** | 持续 Fuzz 测试 | 发现潜在漏洞 |

---

## 结论

**总体评估**：该组件安全实现较好，已实现多层防护机制。

**主要优势**：
1. IPC 接口有完善的权限校验
2. 参数输入有严格的长度校验
3. 内存操作使用安全的 `memcpy_s`
4. 有完整的 SELinux 隔离配置
5. 日志输出注意脱敏处理

**改进空间**：
1. 可考虑增加跨设备数据包的源验证机制
2. 可考虑对动态加载的库进行完整性校验
3. 持续进行安全测试和代码审计
