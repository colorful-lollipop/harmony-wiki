# 安全风险分析

## 目的

本文档基于代码证据对 bundle_framework_lite 进行安全风险评审，识别攻击面、信任边界和潜在的安全漏洞。

## 检查范围

- **代码范围**: `frameworks/`, `interfaces/`, `services/`, `utils/` 目录下的生产代码
- **排除范围**: 测试代码（test/, *_test.cpp, fuzz/ 等）
- **分析维度**: 输入验证、路径遍历、权限控制、内存安全、IPC 安全、签名验证

## 攻击面清单

### 1. 外部输入攻击面

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| HAP 包安装 | `Install()` API | **高** |
| HAP 文件解析 | `BundleParser::ParseHapProfile()` | **高** |
| JSON 配置解析 | `BundleUtil::GetJsonStream()` | **中** |
| IPC 消息处理 | `BundleMsFeature::OnFeatureMessage()` | **中** |
| 路径参数 | `ExtractHap()`, `CreateDataDirectory()` | **高** |

### 2. 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           信任边界图                                     │
└─────────────────────────────────────────────────────────────────────────┘

  不信任区域                    信任边界                      信任区域
┌──────────────┐           ┌──────────────┐           ┌──────────────────┐
│              │           │              │           │                  │
│  第三方应用   │ ────────► │  BundleKit   │ ────────► │  BMS 服务        │
│  (低权限)    │   API     │  (API 校验)  │   IPC     │  (系统权限)      │
│              │           │              │           │                  │
└──────────────┘           └──────────────┘           └────────┬─────────┘
                                                               │
                                                               │ IPC
                                                               ▼
                                                      ┌──────────────────┐
                                                      │  Bundle Daemon   │
                                                      │  (root 权限)     │
                                                      └──────────────────┘
```

## 安全风险详情

### 风险 1: 路径遍历漏洞

**风险等级**: 🔴 **高**

**证据**: `services/bundlemgr_lite/src/gt_bundle_extractor.cpp:297-303`

```cpp
if (strstr(*bundleName, "../") != nullptr) {
    return ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_BUNDLENAME;
}
```

**问题分析**:
- 仅检查 `../` 字符串，可能绕过
- 缺少对 `..\` (Windows)、`..//`、`%2e%2e/` 等变体的检查

**证据**: `services/bundlemgr_lite/src/bundle_util.cpp:76-104`

```cpp
bool BundleUtil::CheckRealPath(const char *path)
{
    // ...
    for (; *next != '\0'; next++) {
        if (*next != '.') {
            continue;
        }
        next++;
        if (*next == '\0' || *next == '.' || *next == '/') {
            return false;
        }
    }
    return true;
}
```

**问题分析**:
- 检查逻辑复杂，可能遗漏某些路径遍历变体
- 未使用 `realpath()` 进行规范化（在部分代码路径中）

**触发路径**:
```
Install(hapPath="/data/app/../../../etc/passwd")
  └── CheckInstallFileIsValid()
       └── 可能通过检查
            └── ExtractHap()
                 └── 访问系统文件
```

**影响**: 可能读取/写入系统关键文件

**修复建议**:
```cpp
// 使用 realpath 规范化路径
char resolvedPath[PATH_MAX];
if (realpath(path, resolvedPath) == nullptr) {
    return ERR_APPEXECFWK_INSTALL_FAILED_FILE_PATH_INVALID;
}
// 检查路径是否在允许的目录下
if (!IsPathInAllowedDirectory(resolvedPath, allowedDirs)) {
    return ERR_APPEXECFWK_INSTALL_FAILED_FILE_PATH_INVALID;
}
```

---

### 风险 2: 签名验证绕过

**风险等级**: 🔴 **高**

**证据**: `services/bundlemgr_lite/src/gt_bundle_installer.cpp:108-152`

```cpp
uint8_t GtBundleInstaller::VerifySignature(const char *path, SignatureInfo &signatureInfo, 
    uint32_t &fileSize, uint8_t bundleStyle)
{
#ifdef _MINI_BMS_
    VerifyResult verifyResult;
    (void) APPVERI_SetDebugMode(true);  // ⚠️ 总是启用调试模式
    int32_t ret = APPVERI_AppVerify(path, &verifyResult);
```

**问题分析**:
- `_MINI_BMS_` 模式下无条件调用 `APPVERI_SetDebugMode(true)`
- 调试模式可能跳过签名验证

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:179-213`

```cpp
#ifdef OHOS_DEBUG
if (ManagerService::GetInstance().IsSignMode()) {
    errorCode = HapSignVerify::VerifySignature(path, signatureInfo);
}
#else
errorCode = HapSignVerify::VerifySignature(path, signatureInfo);
#endif
```

**问题分析**:
- `OHOS_DEBUG` 模式下可通过 `IsSignMode()` 关闭签名验证
- 生产环境不应启用 `OHOS_DEBUG`

**触发路径**:
```
编译时定义 _MINI_BMS_ 或 OHOS_DEBUG
  └── 安装 HAP
       └── 签名验证被绕过
            └── 安装恶意应用
```

**影响**: 可安装未签名或篡改的 HAP 包

**修复建议**:
1. 生产构建强制禁用调试模式
2. 添加编译时断言确保签名验证启用
3. 运行时检查签名验证状态

---

### 风险 3: 整数溢出

**风险等级**: 🟡 **中**

**证据**: `services/bundlemgr_lite/src/bundle_util.cpp:345-373`

```cpp
char *BundleUtil::Strscat(char *str[], uint32_t len)
{
    int32_t strSize = 0;
    for (uint32_t i = 0; i < len; i++) {
        if (str[i] == nullptr) {
            return nullptr;
        }
        strSize += strlen(str[i]);  // ⚠️ 可能溢出
    }
    char *outStr = reinterpret_cast<char *>(AdapterMalloc((strSize + 1) * sizeof(char)));
```

**问题分析**:
- `strSize` 是 `int32_t`，`strlen` 返回 `size_t`
- 多个长字符串相加可能溢出

**证据**: `services/bundlemgr_lite/src/gt_extractor_util.cpp:196-232`

```cpp
int32_t len = strlen(installPath) + strlen(path) + 1;  // ⚠️ 可能溢出
char *destPath = reinterpret_cast<char *>(UI_Malloc(len));
```

**问题分析**:
- 路径长度相加可能溢出，导致分配不足

**触发路径**:
```
构造超长路径字符串
  └── Strscat() / 路径拼接
       └── 整数溢出
            └── 缓冲区溢出
```

**影响**: 堆缓冲区溢出，可能导致代码执行

**修复建议**:
```cpp
// 检查溢出
if (strlen(str1) > INT32_MAX - strlen(str2) - 1) {
    return nullptr;  // 溢出
}
int32_t len = strlen(str1) + strlen(str2) + 1;
```

---

### 风险 4: IPC 消息 ID 越界

**风险等级**: 🟡 **中**

**证据**: `services/bundlemgr_lite/src/bundle_ms_feature.cpp:116-145`

```cpp
uint8_t funcId = request->msgId;
// ...
uint8_t ret = func(funcId, req, reply);  // ⚠️ funcId 未验证范围
```

**问题分析**:
- `funcId` 直接用作数组索引
- 缺少对 `funcId` 范围的验证

**证据**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon.cpp:93-110`

```cpp
int32_t funcId = request->msgId;
if (funcId < 0 || funcId >= BDS_BUTT) {  // ✅ 有边界检查
    return false;
}
```

**对比**: Daemon 有边界检查，但 BMS 缺少

**触发路径**:
```
伪造 IPC 消息，设置 msgId = 255
  └── BundleMsFeature::OnFeatureMessage()
       └── 越界访问函数指针表
            └── 崩溃或代码执行
```

**影响**: 可能执行任意代码

**修复建议**:
```cpp
if (funcId >= BMS_FUNC_COUNT) {
    HILOG_ERROR(HILOG_MODULE_APP, "Invalid funcId: %d", funcId);
    return ERR_APPEXECFWK_INVALID_PARAM;
}
```

---

### 风险 5: JSON 解析拒绝服务

**风险等级**: 🟡 **中**

**证据**: `services/bundlemgr_lite/src/bundle_parser.cpp` (多处使用 cJSON)

```cpp
cJSON *root = cJSON_Parse(buffer);
// 缺少对 JSON 深度和大小限制
```

**问题分析**:
- 使用 cJSON 解析外部输入
- 缺少对 JSON 深度、数组大小、字符串长度的限制
- 深层嵌套 JSON 可能导致栈溢出

**触发路径**:
```
构造深层嵌套 JSON (config.json)
  └── ParseHapProfile()
       └── cJSON_Parse()
            └── 栈溢出或内存耗尽
```

**影响**: 拒绝服务（DoS）

**修复建议**:
1. 限制 JSON 解析深度
2. 限制数组和对象大小
3. 使用流式解析器处理大文件

---

### 风险 6: 权限提升

**风险等级**: 🟡 **中**

**证据**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp:27-40`

```cpp
namespace {
const char *PERMISSIONS_PATH = "/storage/app/etc/permissions/";
constexpr pid_t PMS_UID = 7;
constexpr pid_t PMS_GID = 7;
```

**问题分析**:
- UID/GID 硬编码
- 缺少调用者身份验证

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:425-438`

```cpp
uint8_t BundleInstaller::Uninstall(const char *bundleName, const InstallParam &installParam)
{
    // ...
    if (bundleInfo->isSystemApp) {
        return ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE;
    }
```

**问题分析**:
- 仅检查 `isSystemApp` 标志
- 缺少调用者 UID 验证

**触发路径**:
```
普通应用调用 Uninstall()
  └── 可能卸载其他应用的数据
```

**影响**: 可能删除其他应用数据

**修复建议**:
1. IPC 调用时验证调用者 UID
2. 检查调用者是否有权限操作目标应用
3. 使用能力（Capability）机制替代 UID 检查

---

### 风险 7: 内存泄漏

**风险等级**: 🟢 **低**

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:168-245`

```cpp
uint8_t BundleInstaller::ProcessBundleInstall(...)
{
    BundleInfo *bundleInfo = nullptr;
    Permissions permissions = {.permNum = 0, .permissionTrans = nullptr};
    // ...
    CHECK_PRO_RESULT(errorCode, bundleInfo, permissions, bundleRes.abilityRes);
    // 部分路径可能泄漏内存
}
```

**问题分析**:
- 复杂错误处理路径
- 部分资源可能在错误时未释放

**修复建议**:
- 使用 RAII 模式管理资源
- 使用智能指针（如 `UPtr`）

---

### 风险 8: ZIP 解压漏洞

**风险等级**: 🟡 **中**

**证据**: `services/bundlemgr_lite/src/zip_file.cpp`

```cpp
// ZIP 文件解析
```

**问题分析**:
- 使用 zlib 解压 HAP 包
- 可能存在 ZIP Slip 漏洞（虽然部分代码有检查）

**证据**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp:41-78`

```cpp
for (const auto &fileName : fileNames) {
    if (fileName.find("..") != std::string::npos) {  // ✅ 有检查
        PRINTE("BundleDaemonHandler", "zip file is invalid!");
        return EC_NODIR;
    }
}
```

**修复建议**:
- 使用 `realpath()` 验证解压后的路径
- 确保解压路径在目标目录下

---

## 安全建议汇总

### 立即修复（高优先级）

1. **修复路径遍历漏洞**
   - 统一使用 `realpath()` 规范化路径
   - 验证路径在允许的目录下

2. **移除调试模式后门**
   - 生产构建禁用 `APPVERI_SetDebugMode(true)`
   - 添加编译时检查

3. **修复 IPC 越界访问**
   - 验证所有 IPC 消息 ID 范围

### 建议修复（中优先级）

4. **修复整数溢出**
   - 添加溢出检查
   - 使用安全整数类型

5. **加强 JSON 解析安全**
   - 限制解析深度和大小

6. **加强权限验证**
   - 验证 IPC 调用者身份

### 代码审计建议

7. 定期进行模糊测试（Fuzzing）
8. 使用静态分析工具扫描漏洞
9. 建立安全编码规范

## 检查局限性

1. **未覆盖范围**:
   - 第三方库（zlib, cJSON, appverify）的内部漏洞
   - 内核层面的安全问题
   - 硬件层面的安全问题

2. **分析限制**:
   - 基于静态代码分析，未进行动态测试
   - 部分漏洞需要特定环境触发
   - 缺少完整的威胁模型

---

**相关链接**:
- [项目概览](00_Overview.md)
- [架构说明](02_Architecture.md)
- [对外 API](03_Public_API.md)
