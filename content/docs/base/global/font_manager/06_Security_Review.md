# 安全风险评审

## 概述

本章节对 font_manager 进行安全风险评审，识别潜在的攻击面、信任边界和安全风险点。

## 评审范围

| 组件 | 评审状态 |
|------|----------|
| N-API 层 (font_manager_addon) | ✅ 已评审 |
| 客户端 (FontManagerClient) | ✅ 已评审 |
| 服务端 (FontManagerServer) | ✅ 已评审 |
| 核心框架 (FontManager) | ✅ 已评审 |
| 文件操作 | ✅ 已评审 |
| IPC/Binder 通信 | ✅ 已评审 |
| 权限校验 | ✅ 已评审 |

**未评审**: 测试代码、ANI 接口

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  沙箱内应用                                          │   │
│  │  • 有限权限                                         │   │
│  │  • 需申请 UPDATE_FONT 权限                          │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  N-API 层 (fontmanager)                             │   │
│  │  • 参数校验                                         │   │
│  │  • 类型转换                                         │   │
│  │  • 异步执行                                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  客户端 (FontManagerClient)                         │   │
│  │  • 路径验证                                         │   │
│  │  • IPC 调用                                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓ IPC (Binder)                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  服务端 (FontManagerServer)                         │   │
│  │  • 权限校验                                         │   │
│  │  • 业务逻辑                                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  核心框架 (FontManager)                              │   │
│  │  • 文件操作                                         │   │
│  │  • 配置管理                                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  文件系统 (/data/service/el1/)                      │   │
│  │  • 字体文件存储                                     │   │
│  │  • 配置 JSON 文件                                   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 攻击面分析

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| N-API 参数 | 应用传入的字体路径、字体名称 | 中 |
| 文件描述符 | InstallFont 接收的文件描述符 | 高 |
| IPC 调用 | Binder 通信中的参数传递 | 中 |
| 配置文件 | install_fontconfig.json | 低 |
| 权限校验 | UPDATE_FONT 权限 | 中 |

## 安全风险清单

### ✅ 已缓解风险

#### 1. 权限校验

**风险**: 未授权应用调用字体安装/卸载 API

**缓解措施**: 服务端强制校验 `ohos.permission.UPDATE_FONT` 权限

**证据**: `service/server/src/font_manager_server.cpp:213-223`

```cpp
int32_t FontManagerServer::CheckPermission()
{
    uint32_t callerToken = IPCSkeleton::GetCallingTokenID();
    int result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, PERMISSION_UPDATE_FONT);
    if (result != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return ERR_NO_PERMISSION;
    }
    return ERR_OK;
}
```

**评估**: ✅ 已正确实现，调用方无绕过可能

---

#### 2. 路径遍历防护

**风险**: 应用通过路径遍历访问敏感文件

**缓解措施**: 客户端使用 `realpath()` 规范化路径

**证据**: `service/client/src/font_manager_client.cpp:94-118`

```cpp
bool FontManagerClient::PathToRealPath(const std::string& path, std::string& realPath)
{
    // 检查空路径
    if (path.empty()) {
        return false;
    }
    // 检查路径长度
    if (path.length() >= PATH_MAX) {
        return false;
    }
    // realpath 规范化并解析符号链接
    if (realpath(path.c_str(), tmpPath) == nullptr) {
        return false;
    }
    // 验证文件存在
    if (access(realPath.c_str(), F_OK) != 0) {
        return false;
    }
    return true;
}
```

**评估**: ✅ 正确使用 `realpath()` 防止路径遍历

---

#### 3. 文件描述符验证

**风险**: 传入无效的文件描述符

**缓解措施**: 客户端验证 fd 有效性

**证据**: `service/client/src/font_manager_client.cpp:38-50`

```cpp
FILE* fp = fopen(realPath.c_str(), "rb");
if (!fp) {
    outValue = ERR_FILE_NOT_EXISTS;
    return ERR_OK;
}
int fd = fileno(fp);
if (fd < 0) {
    outValue = ERR_FILE_NOT_EXISTS;
    (void)fclose(fp);
    return ERR_OK;
}
```

**评估**: ✅ 正确验证 fd 有效性

---

#### 4. 文件数量限制

**风险**: 无限安装字体导致存储耗尽

**缓解措施**: 限制最大安装数量 (200)

**证据**: `frameworks/fontmgr/src/font_manager.cpp:30,67-69`

```cpp
static constexpr int32_t MAX_INSTALL_NUM = 200;

if (fontConfig.GetInstalledFontsNum() >= MAX_INSTALL_NUM) {
    return ERR_MAX_FILE_COUNT;
}
```

**评估**: ✅ 已实施数量限制

---

#### 5. 字体文件格式验证

**风险**: 恶意字体文件导致安全漏洞

**缓解措施**: 服务端验证字体文件有效性

**证据**: `frameworks/fontmgr/src/font_manager.cpp:50-53`

```cpp
std::vector<std::string> fullNameVector = FontManagerUtils::GetFullNamesByFd(fd);
if (fullNameVector.size() == 0) {
    return ERR_FILE_VERIFY_FAIL;
}
```

**评估**: ⚠️ 需确认 `GetFullNamesByFd` 的验证强度

---

### ⚠️ 潜在风险

#### 6. TOCTOU 竞态条件

**风险**: `access()` 检查和实际打开之间存在时间窗口

**位置**: `service/client/src/font_manager_client.cpp:94-118`

**描述**: 

```cpp
// 检查点
if (access(realPath.c_str(), F_OK) != 0) {
    return false;
}
// 后续 fopen() 可能在检查后被攻击者替换文件
FILE* fp = fopen(realPath.c_str(), "rb");
```

**触发条件**: 攻击者在 `access()` 和 `fopen()` 之间替换文件

**影响**: 可能安装恶意文件

**修复建议**: 

```cpp
// 使用 open() + fstat() 替代 access() + fopen()
int fd = open(realPath.c_str(), O_RDONLY);
if (fd < 0) {
    return ERR_FILE_NOT_EXISTS;
}
struct stat st;
if (fstat(fd, &st) < 0 || !S_ISREG(st.st_mode)) {
    close(fd);
    return ERR_FILE_NOT_EXISTS;
}
// 使用 fd 而不是 FILE*
```

**优先级**: 中

---

#### 7. 字体配置文件注入

**风险**: JSON 配置文件被恶意修改

**位置**: `frameworks/fontmgr/src/font_config.cpp`

**描述**: FontConfig 直接读写 install_fontconfig.json

**触发条件**: 如果配置文件的目录权限配置不当

**影响**: 可能导致字体加载异常或安全问题

**缓解因素**: 配置文件位于 `/data/service/el1/{userId}/for-all-app/fonts/`，受系统权限保护

**修复建议**: 使用签名或校验和保护配置文件完整性

**优先级**: 低

---

#### 8. SA 空闲自动卸载

**风险**: SA 卸载可能导致竞态问题

**位置**: `service/server/src/font_manager_server.cpp:163-183`

**描述**:

```cpp
void FontManagerServer::AddUnloadFontServiceTask()
{
    // 空闲 10 秒后自动卸载 SA
    handler_->PostTask(task, UNLOAD_TASK, DELAY_MILLISECONDS_FOR_UNLOAD_SA);
}
```

**潜在问题**: 如果在卸载过程中有新的调用进入，可能导致问题

**缓解因素**: 使用 `callingCount_` 计数器防止卸载时新调用进入

**评估**: ✅ 已有防护措施，但建议添加额外日志

**优先级**: 低

---

#### 9. 临时文件处理

**风险**: 安装过程中的临时文件可能被利用

**位置**: `frameworks/fontmgr/src/font_manager.cpp:97-118`

**描述**:

```cpp
std::string tempPath = installPath + TEMP_FILE + fileName;
// 复制到临时文件
FontManagerUtils::CopyFile(fd, tempPath);
// 重命名为最终文件名
FontManagerUtils::RenameFile(tempPath, destPath);
```

**潜在问题**: 
- 临时文件目录 `TEMP_FILE` 权限
- 原子性重命名

**缓解因素**: 
- 使用 `RenameFile` (原子操作)
- 失败时清理临时文件

**评估**: ✅ 已有基本防护

**优先级**: 低

---

#### 10. 内存安全

**风险**: C++ 代码可能存在内存安全问题

**缓解措施**: 
- 构建时启用 sanitizer (cfi, ubsan, boundary_sanitize)
- 使用 `-fdata-sections`, `-ffunction-sections` 等编译选项

**证据**: `service/BUILD.gn:138-145`

```gn
sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    integer_overflow = true
    ubsan = true
}
```

**评估**: ✅ 已启用多种安全检测

---

## 安全加固建议

### 高优先级

| 建议 | 原因 | 复杂度 |
|------|------|--------|
| 修复 TOCTOU 竞态条件 | 存在潜在的文件替换攻击 | 中 |
| 增加字体文件格式验证强度 | 防止恶意字体文件 | 中 |

### 中优先级

| 建议 | 原因 | 复杂度 |
|------|------|--------|
| 添加配置文件完整性校验 | 防止配置注入 | 低 |
| 增加安全审计日志 | 记录敏感操作 | 低 |

### 低优先级

| 建议 | 原因 | 复杂度 |
|------|------|--------|
| 增加 SA 卸载日志 | 便于问题定位 | 低 |
| 使用内存安全语言重写 | 长期安全改进 | 高 |

## 相关文档

- [N-API 参考](02_NAPI_Reference.md)
- [内部 API](03_Inner_API.md)
- [故障排查](07_Troubleshooting.md)
