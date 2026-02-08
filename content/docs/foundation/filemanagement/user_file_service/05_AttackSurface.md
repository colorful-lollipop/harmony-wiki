# User File Service - 攻击面分析

## 概述

本文档系统分析 user_file_service 的攻击面，识别所有外部输入入口、敏感操作点和信任边界跨越点，为安全研究提供完整的攻击面地图。

**分析范围**：
- N-API 接口层 (`frameworks/js/napi/`, `interfaces/kits/`)
- IPC 接口层 (`interfaces/inner_api/`)
- 服务层 (`services/native/`)
- 工具类 (`utils/`)

**排除范围**：测试代码 (`test/`, `unittest/`, `fuzztest/`)

---

## 1. 外部输入入口清单

### 1.1 N-API 接口入口（高风险）

#### file.fileAccess 模块

| 函数名 | 文件路径 | 行号 | 输入参数 | 风险等级 |
|--------|----------|------|----------|---------|
| `NAPI_OpenFile` | `napi_fileaccess_helper.cpp` | 312 | uri, flags | 🔴 高 |
| `NAPI_CreateFile` | `napi_fileaccess_helper.cpp` | 361 | uri, name | 🔴 高 |
| `NAPI_Mkdir` | `napi_fileaccess_helper.cpp` | 418 | uri, name | 🔴 高 |
| `NAPI_Delete` | `napi_fileaccess_helper.cpp` | 475 | uri | 🔴 高 |
| `NAPI_Move` | `napi_fileaccess_helper.cpp` | 529 | srcUri, dstUri | 🔴 高 |
| `NAPI_Copy` | `napi_fileaccess_helper.cpp` | 754 | srcUri, dstUri | 🔴 高 |
| `NAPI_Rename` | `napi_fileaccess_helper.cpp` | 850 | uri, name | 🔴 高 |
| `NAPI_Query` | `napi_fileaccess_helper.cpp` | 579 | uri | 🟡 中 |
| `NAPI_Access` | `napi_fileaccess_helper.cpp` | 987 | uri | 🟡 中 |
| `NAPI_RegisterObserver` | `napi_fileaccess_helper.cpp` | 1217 | uri, notifyType | 🟡 中 |
| `NAPI_UnregisterObserver` | `napi_fileaccess_helper.cpp` | 1265 | uri | 🟡 中 |

**证据**：`frameworks/js/napi/file_access_module/napi_fileaccess_helper.cpp`

```cpp
// 示例：NAPI_OpenFile 入口点
static napi_value NAPI_OpenFile(napi_env env, napi_callback_info info)
{
    // 解析 JS 传入的 uri 和 flags 参数
    NAPI_GET_STRING_PARAM(env, args[0], uri, value);
    NAPI_GET_INT32_PARAM(env, args[1], flags, value);
    // ... 后续处理
}
```

#### file.picker 模块

| 函数名 | 文件路径 | 行号 | 输入参数 | 风险等级 |
|--------|----------|------|----------|---------|
| `StartModalPicker` | `picker_n_exporter.cpp` | 477 | pickerType, options | 🟡 中 |
| `PickerNExporter::Select` | `picker_n_exporter.cpp` | 280 | context, option | 🟡 中 |
| `PickerNExporter::Save` | `picker_n_exporter.cpp` | 350 | context, option | 🟡 中 |

#### file.recent 模块

| 函数名 | 文件路径 | 行号 | 输入参数 | 风险等级 |
|--------|----------|------|----------|---------|
| `AddRecentFile` | `recent_n_exporter.cpp` | 83 | uri, options | 🔴 高 |
| `RemoveRecentFile` | `recent_n_exporter.cpp` | 136 | uri | 🔴 高 |

**证据**：`interfaces/kits/native/recent/recent_n_exporter.cpp:41`

```cpp
static bool CheckPermission(const std::string &permission)
{
    // 权限检查点
}
```

### 1.2 IPC 接口入口（高风险）

#### FileAccessExtStubImpl IPC 处理函数

| 函数名 | 文件路径 | 行号 | IPC Code | 风险等级 |
|--------|----------|------|----------|---------|
| `OpenFile` | `file_access_ext_stub_impl.cpp` | 52 | OPEN_FILE | 🔴 高 |
| `CreateFile` | `file_access_ext_stub_impl.cpp` | 79 | CREATE_FILE | 🔴 高 |
| `Delete` | `file_access_ext_stub_impl.cpp` | 127 | DELETE | 🔴 高 |
| `Mkdir` | `file_access_ext_stub_impl.cpp` | 103 | MKDIR | 🔴 高 |
| `Move` | `file_access_ext_stub_impl.cpp` | 149 | MOVE | 🔴 高 |
| `Copy` | `file_access_ext_stub_impl.cpp` | 178 | COPY | 🔴 高 |
| `Rename` | `file_access_ext_stub_impl.cpp` | 237 | RENAME | 🔴 高 |
| `ListFile` | `file_access_ext_stub_impl.cpp` | 261 | LIST_FILE | 🟡 中 |
| `Query` | `file_access_ext_stub_impl.cpp` | 340 | QUERY | 🟡 中 |
| `Access` | `file_access_ext_stub_impl.cpp` | 424 | ACCESS | 🟡 中 |
| `StartWatcher` | `file_access_ext_stub_impl.cpp` | 446 | START_WATCHER | 🟡 中 |
| `StopWatcher` | `file_access_ext_stub_impl.cpp` | 468 | STOP_WATCHER | 🟡 中 |

**证据**：`interfaces/inner_api/file_access/src/file_access_ext_stub_impl.cpp:38`

```cpp
bool FileAccessExtStubImpl::CheckCallingPermission(const std::string &permission)
{
    HITRACE_METER_NAME(HITRACE_TAG_FILEMANAGEMENT, __PRETTY_FUNCTION__);
    // 权限检查实现
    return AccessTokenKit::VerifyAccessToken(tokenId, permission) == PERMISSION_GRANTED;
}
```

### 1.3 配置/数据输入入口

| 输入源 | 位置 | 用途 | 风险等级 |
|--------|------|------|----------|
| `5010.json` | `services/` | SA 配置 | 🟢 低 |
| `file_access_service.cfg` | `services/` | 启动配置 | 🟢 低 |
| RDB 数据库 | `services/rdb_adapter/` | 同步文件夹数据 | 🟡 中 |
| 共享内存 | `file_info_shared_memory.h` | 文件信息传递 | 🟡 中 |

---

## 2. 敏感操作清单

### 2.1 文件系统操作（高风险）

| 操作 | 函数/方法 | 位置 | 风险描述 |
|------|-----------|------|----------|
| 文件创建 | `CreateFile()` | `file_access_ext_stub_impl.cpp:79` | 任意路径文件创建 |
| 文件删除 | `Delete()` | `file_access_ext_stub_impl.cpp:127` | 任意文件删除 |
| 文件移动 | `Move()` | `file_access_ext_stub_impl.cpp:149` | 路径变更、覆盖 |
| 文件复制 | `Copy()` | `file_access_ext_stub_impl.cpp:178` | 数据泄露、DoS |
| 目录创建 | `Mkdir()` | `file_access_ext_stub_impl.cpp:103` | 目录结构操作 |
| 文件打开 | `OpenFile()` | `file_access_ext_stub_impl.cpp:52` | 文件描述符获取 |
| 重命名 | `Rename()` | `file_access_ext_stub_impl.cpp:237` | 文件名变更 |

### 2.2 IPC 连接操作（高风险）

| 操作 | 函数 | 位置 | 风险描述 |
|------|------|------|----------|
| 连接扩展 | `ConnectFileExtAbility()` | `file_access_service.cpp:523` | 跨应用连接 |
| 断开扩展 | `DisConnectFileExtAbility()` | `file_access_service.cpp:550` | 连接管理 |
| 获取代理 | `GetExtensionProxy()` | `file_access_service.cpp:400` | 代理对象获取 |

### 2.3 观察者操作（中风险）

| 操作 | 函数 | 位置 | 风险描述 |
|------|------|------|----------|
| 注册观察者 | `RegisterNotify()` | `file_access_service.cpp:320` | 回调注册、资源消耗 |
| 注销观察者 | `UnregisterNotify()` | `file_access_service.cpp:380` | 清理操作 |
| 发送通知 | `OnChange()` | `file_access_service.cpp:600` | 跨进程通知 |

### 2.4 云盘同步操作（中风险）

| 操作 | 函数 | 位置 | 风险描述 |
|------|------|------|----------|
| 注册同步文件夹 | `Register()` | `file_access_service.cpp:700` | 路径注册 |
| 激活同步 | `Active()` | `file_access_service.cpp:750` | 同步状态变更 |
| 更新显示名 | `UpdateDisplayName()` | `file_access_service.cpp:800` | 元数据修改 |

---

## 3. 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              非信任区域 (Untrusted)                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  第三方应用 (JS/ArkTS)                                                │  │
│  │  • 恶意构造的 URI: "file://../../../etc/passwd"                       │  │
│  │  • 超长文件名                                                         │  │
│  │  • 特殊字符注入                                                       │  │
│  │  • 非法 flags 组合                                                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ N-API 调用                             │
│                                    ▼                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                            信任边界 #1: N-API 层                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  N-API 胶水层 (frameworks/js/napi/)                                    │  │
│  │  • ToUTF8String() - 字符串参数解析                                     │  │
│  │  • napi_get_value_string_utf8() - JS → C++ 转换                       │  │
│  │  ⚠️ 风险: 参数类型/长度/编码验证                                        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ IPC 调用                               │
│                                    ▼                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                         信任边界 #2: IPC 传输层                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  IPC Client (interfaces/inner_api/)                                    │  │
│  │  • MessageParcel 序列化/反序列化                                       │  │
│  │  • SendRequest() / ReadFromParcel()                                   │  │
│  │  ⚠️ 风险: IPC 数据篡改、中间人                                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ 跨进程通信                             │
│                                    ▼                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                         信任边界 #3: 系统服务层                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  FileAccessService (SA 5010)                                          │  │
│  │  • CheckCallingPermission() - 权限校验                                 │  │
│  │  • IsFilePathValid() - 路径遍历检查                                    │  │
│  │  • URI 规范化处理                                                      │  │
│  │  ✅ 关键安全控制点                                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ 扩展连接                               │
│                                    ▼                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                         信任边界 #4: 扩展能力层                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  File Extension Ability                                               │  │
│  │  • MediaLibraryExtAbility (媒体库)                                     │  │
│  │  • ExternalFileExtAbility (外置存储)                                   │  │
│  │  • 其他自定义扩展                                                       │  │
│  │  ⚠️ 风险: 扩展实现的安全性差异                                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    │ 系统调用                               │
│                                    ▼                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                             受信任区域 (Trusted)                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  底层文件系统                                                          │  │
│  │  • /storage/emulated/0/ (公共存储)                                     │  │
│  │  • 应用沙箱目录                                                        │  │
│  │  • 外置存储设备                                                        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 数据流与检查点

### 4.1 文件访问请求数据流

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  应用    │────▶│  N-API   │────▶│  IPC     │────▶│  SA 5010 │────▶│ 扩展    │
│  (JS)    │     │  层      │     │  Client  │     │  服务    │     │ Ability  │
└──────────┘     └──────────┘     └──────────┘     └────┬─────┘     └──────────┘
                                                        │
                            ┌───────────────────────────┼───────────────────────────┐
                            │                           │                           │
                            ▼                           ▼                           ▼
                    ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
                    │ 权限检查点   │           │ 路径检查点   │           │ 资源限制点   │
                    │ CheckCalling │           │ IsFilePath   │           │ 连接池限制   │
                    │ Permission() │           │ Valid()      │           │ 观察者上限   │
                    └──────────────┘           └──────────────┘           └──────────────┘
```

### 4.2 安全检查点详表

| 检查点 | 位置 | 检查内容 | 失败处理 |
|--------|------|----------|----------|
| **权限检查** | `file_access_ext_stub_impl.cpp:64` | `ohos.permission.FILE_ACCESS_MANAGER` | 返回 `E_PERMISSION` |
| **系统应用检查** | `recent_n_exporter.cpp:54` | `IsSystemAppByFullTokenID()` | 拒绝访问 |
| **路径遍历检查** | `file_uri_check.h:29` | 检测 `../` 和 `/..` | 返回 `E_INVALID_URI` |
| **URI 格式检查** | `file_access_helper.cpp:107` | Scheme 和格式验证 | 返回错误码 |
| **Token 有效性** | `ufs_access_token_helper.cpp:63` | AccessTokenKit 验证 | 拒绝请求 |

---

## 5. 攻击向量分析

### 5.1 路径遍历攻击

**攻击面**：所有接受 URI/路径参数的 N-API 和 IPC 接口

**潜在载荷**：
```javascript
// 标准路径遍历
faHelper.listFile("file:///storage/../../../../etc/passwd");

// URL 编码绕过 (需验证)
faHelper.listFile("file:///storage/%2e%2e%2f%2e%2e%2fetc/passwd");

// 空字节注入 (历史漏洞模式)
faHelper.openFile("file:///storage/valid.txt%00/../../../etc/passwd");
```

**缓解措施**：
- `IsFilePathValid()` 函数检查 `../` 和 `/..` 模式
- URI 规范化处理

**证据**：`utils/file_uri_check.h:29-45`

```cpp
static bool IsFilePathValid(const std::string &filePath)
{
    // 检查 "../"
    size_t pos = filePath.find(PATH_INVALID_FLAG1);
    while (pos != std::string::npos) {
        if (pos == 0 || filePath[pos - 1] == FILE_SEPARATOR_CHAR) {
            return false;  // 发现路径遍历尝试
        }
        pos = filePath.find(PATH_INVALID_FLAG1, pos + PATH_INVALID_FLAG_LEN);
    }
    // ... 检查 "/.."
}
```

### 5.2 权限绕过攻击

**攻击面**：IPC 接口调用

**攻击场景**：
```cpp
// 尝试在未授权情况下调用敏感接口
sptr<FileAccessServiceBaseStub> proxy = GetProxy();
// 绕过权限检查直接调用
proxy->CreateFile(maliciousUri);  // 应在服务端被拦截
```

**缓解措施**：
- 所有 IPC 接口入口调用 `CheckCallingPermission()`
- 服务端权限验证，客户端无法绕过

**证据**：`file_access_ext_stub_impl.cpp:64-68`

```cpp
if (!CheckCallingPermission(FILE_ACCESS_MANAGER_PERMISSION)) {
    return E_PERMISSION;  // 权限不足
}
```

### 5.3 DoS 攻击

**攻击面**：
- 大量观察者注册 → 内存耗尽
- 大文件列表请求 → CPU/内存资源耗尽
- 频繁 IPC 调用 → 服务过载

**潜在载荷**：
```javascript
// 注册大量观察者
for (let i = 0; i < 100000; i++) {
    faHelper.registerObserver("uri" + i, callback);
}
```

**缓解措施**：
- TODO(证据不足): 需确认是否有观察者数量限制
- 连接池管理

### 5.4 符号链接攻击

**攻击面**：`recent_n_exporter.cpp` 使用 `symlink()`

**证据**：`interfaces/kits/native/recent/recent_n_exporter.cpp:128`

```cpp
// 创建符号链接
symlink(targetPath, linkPath);
```

**风险**：如果 `targetPath` 未经验证，可能导致未授权文件访问

---

## 6. 攻击面统计

| 类别 | 数量 | 高风险 | 中风险 | 低风险 |
|------|------|--------|--------|--------|
| N-API 接口 | 12+ | 8 | 4 | 0 |
| IPC 接口 | 12+ | 8 | 4 | 0 |
| 敏感操作 | 15+ | 10 | 3 | 2 |
| 检查点 | 5 | - | - | - |

---

## 7. 审计检查清单

### 7.1 输入验证审计

- [ ] 所有 N-API 参数是否都有类型检查？
- [ ] 字符串参数是否都有长度限制？
- [ ] URI 参数是否都经过 `IsFilePathValid()` 检查？
- [ ] 数值参数是否都有范围验证？

### 7.2 权限控制审计

- [ ] 所有 IPC 接口是否都有权限检查？
- [ ] `CheckCallingPermission()` 是否在所有入口调用？
- [ ] 权限检查失败是否都返回正确错误码？

### 7.3 资源管理审计

- [ ] 观察者注册是否有数量上限？
- [ ] IPC 连接是否有超时和清理机制？
- [ ] 大文件操作是否有大小限制？

---

## 8. 相关文档

- [01_Architecture.md](01_Architecture.md) - 架构设计和数据流
- [06_Security_Review.md](06_Security_Review.md) - 详细风险评估和修复建议
- [02_NAPI_Reference.md](02_NAPI_Reference.md) - API 完整参考

---

## 9. 证据汇总

| 证据类型 | 文件路径 | 关键行号 |
|---------|----------|----------|
| N-API 注册 | `native_fileaccess_module.cpp` | 81, 91 |
| 权限检查 | `file_access_ext_stub_impl.cpp` | 38, 64 |
| 路径检查 | `file_uri_check.h` | 29 |
| IPC 接口 | `file_access_service.h` | 227 |
| Token 检查 | `ufs_access_token_helper.cpp` | 63 |

---

**最后更新**：2026-02-07
**分析人**：OpenHarmony Wiki Agent
