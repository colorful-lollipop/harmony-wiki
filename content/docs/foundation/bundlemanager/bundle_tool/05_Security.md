# 安全风险评审 (Security)

> bundle_tool 威胁模型、攻击面与安全风险分析

## 威胁模型概述

### 系统边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                         信任边界                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     bundle_tool (不可信)                      │   │
│  │   - 用户输入参数                                              │   │
│  │   - 文件路径                                                  │   │
│  │   - 网络数据（通过 IPC 传递）                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↑ IPC                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │               BundleManagerService (可信)                   │   │
│  │   - 签名验证                                                  │   │
│  │   - 权限检查                                                  │   │
│  │   - 文件操作                                                  │   │
│  │   - 数据存储                                                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 攻击面清单

### 1. 命令行参数输入

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `-p, --bundle-path` | HAP/HSP 文件路径 | **高** |
| `-n, --bundle-name` | 包名字符串 | 中 |
| `-u, --user-id` | 用户 ID 数值 | 低 |
| `-s, --shared-bundle-dir-path` | HSP 目录路径 | **高** |
| `-f, --file-path` | HQF 文件路径 | **高** |

**处理文件**: `bundle_command.cpp`

### 2. 文件路径遍历

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| 路径遍历 | 恶意构造的路径可能导致任意文件访问 | **高** |
| 符号链接 | 符号链接可能导致文件覆盖 | 中 |

**证据来源**: `bundle_command.cpp` - `GetBundlePath()` 方法

### 3. IPC 通信

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| IBundleMgr 调用 | Bundle 信息查询 | 低 |
| IBundleInstaller 调用 | 安装/卸载操作 | **高** |
| 回调数据 | 状态接收回调 | 低 |

**证据来源**: `bundle_command.h:343-344` - IPC 代理

### 4. 设备标识

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| UDID 获取 | 设备唯一标识暴露 | 低 |
| 用户 ID | 账号标识 | 低 |

---

## 信任边界

### 边界 1: 用户输入 → bundle_tool

| 数据流 | 说明 | 信任级别 |
|--------|------|----------|
| 命令行参数 | 解析后传递给 IPC | 不可信 |
| 文件路径 | 需要规范化验证 | 不可信 |

**边界控制**: 所有用户输入必须经过验证

### 边界 2: bundle_tool → BundleManagerService

| 数据流 | 说明 | 信任级别 |
|--------|------|----------|
| Install() | 安装请求 | 需验证 |
| Uninstall() | 卸载请求 | 需验证 |
| Query() | 查询请求 | 需验证 |

**边界控制**: BundleManagerService 负责最终验证

### 边界 3: BundleManagerService → 文件系统

| 数据流 | 说明 | 信任级别 |
|--------|------|----------|
| HAP 读取 | 读取安装包 | 需沙箱限制 |
| 数据目录 | 应用数据存储 | 需权限控制 |
| 缓存目录 | 临时文件 | 需清理策略 |

---

## 安全风险点

### 🔴 高风险

#### 1. 路径遍历漏洞

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:807-819
ErrCode BundleManagerShellCommand::GetBundlePath(const std::string& param,
    std::vector<std::string>& bundlePaths) const
{
    if (param.empty()) {
        return OHOS::ERR_INVALID_VALUE;
    }
    if (param == "-r" || param == "--replace" || param == "-p" ||
        param == "--bundle-path" || param == "-u" || param == "--user-id" ||
        param == "-w" || param == "--waitting-time") {
        return OHOS::ERR_INVALID_VALUE;
    }
    bundlePaths.emplace_back(param);
    return OHOS::ERR_OK;
}
```

**问题**: 
- 路径参数仅检查了是否为选项关键字，未进行路径规范化
- 缺少对 `../` 等路径遍历模式的检查

**触发条件**:
```bash
bm install -p /data/../../system/bin/malicious
```

**影响**:
- 任意文件读取
- 配置文件篡改
- 权限提升

**修复建议**:
```cpp
// 规范化路径
std::string NormalizePath(const std::string& path) {
    char resolvedPath[PATH_MAX];
    if (realpath(path.c_str(), resolvedPath) == nullptr) {
        return "";
    }
    // 检查路径前缀是否在允许范围内
    std::string allowedPrefix = "/data/local/tmp/";
    if (strncmp(resolvedPath, allowedPrefix.c_str(), allowedPrefix.length()) != 0) {
        return ""; // 路径不在允许范围内
    }
    return std::string(resolvedPath);
}
```

---

#### 2. 特权操作风险 (bundle_test_tool)

**证据**:
```cpp
// frameworks/src/bundle_test_tool.cpp 多处调用
// 26 处 setuid() 调用，17 处 seteuid() 调用

// 示例 (line 2978-2980):
int32_t uid = GetUidByBundleName(bundleName);
setuid(uid);  // 切换到目标应用 UID
// 执行操作...
setuid(Constants::ROOT_UID);  // 恢复 root

// 代码签名设备访问 (line 7046-7053):
int32_t fd = open(CODE_SIGN_PATH, O_RDONLY);
int ret = ioctl(fd, CODESIGN_SET_ENP_DEVICE_FLAG, 1);
close(fd);

// Native Token 创建 (line 5462-5481):
perms[0] = "ohos.permission.MANAGE_EDM_POLICY";
perms[1] = "ohos.permission.MANAGE_DISPOSED_APP_STATUS";
perms[2] = "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED";
perms[3] = "ohos.permission.GET_INSTALLED_BUNDLE_LIST";
tokenId = GetAccessTokenId(&infoInstance);
SetSelfTokenID(tokenId);
```

**问题**:
1. **特权切换**: 频繁在 root 与普通 UID 之间切换，存在 TOCTOU 竞态条件风险
2. **代码签名设备**: 直接操作内核代码签名设备，可修改系统级签名策略
3. **权限获取**: 测试工具获取了系统级权限，包括 EDM 策略管理和应用处置状态管理

**触发条件**:
```bash
# 使用 bundle_test_tool 执行特权操作
bundle_test_tool --install --bundle-name com.example.app
```

**影响**:
- 权限提升
- 系统安全策略绕过
- 恶意应用安装

**修复建议**:
1. 限制 bundle_test_tool 的发布范围，仅限内部测试使用
2. 增加额外的权限检查层
3. 对特权操作增加审计日志
4. 考虑使用能力 (capabilities) 替代 setuid

---

#### 3. 企业证书安装风险

**证据**:
```cpp
// frameworks/src/bundle_test_tool.cpp:7145-7147
int32_t fd = open(certPath.c_str(), O_RDONLY);
auto res = bundleInstallerProxy_->InstallEnterpriseReSignatureCert(certAlias, fd, userId);
close(fd);
```

**问题**:
- `certPath` 来自用户输入，未验证路径合法性
- 直接打开并传递文件描述符给系统服务

**触发条件**:
```bash
bundle_test_tool --install-cert --path /path/to/cert
```

**影响**:
- 安装恶意企业证书
- 绕过应用签名验证
- 企业设备管理策略被篡改

**修复建议**:
- 验证证书路径在允许目录内
- 检查证书文件权限和所有权
- 增加证书内容预验证

---

#### 2. 安装包签名绕过

**证据**:
```
依赖: appverify (bundle.json:38)
```

**问题**: 签名验证完全依赖 BundleManagerService

**触发条件**:
```bash
# 恶意构造的 HAP
bm install -p /path/to/unsigned.hap
```

**影响**:
- 安装未签名应用
- 恶意代码执行

**修复建议**:
- 在 bundle_tool 侧增加签名预检查
- 显示签名信息供用户确认

---

#### 3. 权限滥用

**证据**:
```
bundle_command.h: Enable/Disable 命令
```

**问题**: enable/disable 命令在 root 版本可用

**触发条件**:
```bash
# 禁用系统应用
bm disable -n com.system.app
```

**影响**:
- 系统功能破坏
- 设备功能丧失

**修复建议**:
- 增加权限检查
- 限制可操作的包名范围

---

### 🟠 中风险

#### 4. 权限模式检查绕过

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:51-56
const char* IS_ROOT_MODE_PARAM = "const.debuggable";
const std::string IS_DEVELOPER_MODE_PARAM = "const.security.developermode.state";
const int32_t ROOT_MODE = 1;

// frameworks/src/bundle_command.cpp:284-288
int32_t mode = GetIntParameter(IS_ROOT_MODE_PARAM, USER_MODE);
if (mode == ROOT_MODE) {
    commandMap_.emplace("enable", [this] { return this->RunAsEnableCommand(); });
    commandMap_.emplace("disable", [this] { return this->RunAsDisableCommand(); });
}

// frameworks/src/bundle_command.cpp:355-362
ErrCode BundleManagerShellCommand::RunAsCopyApCommand()
{
    int32_t mode = GetIntParameter(IS_ROOT_MODE_PARAM, USER_MODE);
    bool isDeveloperMode = system::GetBoolParameter(IS_DEVELOPER_MODE_PARAM, false);
    if (mode != ROOT_MODE && !isDeveloperMode) {
        APP_LOGI("in user mode but not in developer mode");
        return ERR_OK;  // 静默返回，不报错
    }
    // ...
}
```

**问题**:
1. **参数可篡改**: `const.debuggable` 和 `const.security.developermode.state` 是系统参数，可能被篡改
2. **静默失败**: copy-ap 在权限不足时静默返回，可能误导用户认为操作成功
3. **权限检查不一致**: 不同命令使用不同的权限检查模式

**触发条件**:
```bash
# 如果系统参数被篡改
bm enable -n com.system.app  # 可能绕过权限检查
```

**影响**:
- 权限绕过
- 未授权操作执行

**修复建议**:
- 使用 AccessToken 框架进行权限校验
- 统一权限检查接口
- 权限不足时返回明确错误

---

#### 5. 输入验证不完整

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:688-696 - 用户 ID 验证
if (!OHOS::StrToInt(optarg, userId) || userId < 0) {
    APP_LOGE("bm install with error userId %{private}s", optarg);
    resultReceiver_.append(STRING_REQUIRE_CORRECT_VALUE);
    return OHOS::ERR_INVALID_VALUE;
}

// 问题: 虽然有验证，但缺少上限检查

// frameworks/src/bundle_command.cpp:701-706 - 等待时间验证
if (!OHOS::StrToInt(optarg, waittingTime) || waittingTime < MINIMUM_WAITTING_TIME ||
    waittingTime > MAXIMUM_WAITTING_TIME) {
    APP_LOGE("bm install with error waittingTime %{private}s", optarg);
    resultReceiver_.append(STRING_REQUIRE_CORRECT_VALUE);
    return OHOS::ERR_INVALID_VALUE;
}

// 问题: 字符串参数缺少长度限制
```

**问题**:
- 数值参数有范围检查，但字符串参数缺少长度限制
- 缺少对特殊字符的过滤
- 路径参数未验证是否存在路径遍历

**触发条件**:
```bash
# 超长包名
bm install -n $(python3 -c "print('A'*10000)")
```

**影响**:
- 缓冲区溢出风险
- 日志注入
- 拒绝服务

**修复建议**:
- 限制字符串参数长度
- 使用正则表达式验证包名格式
- 增加特殊字符过滤

---

#### 6. IPC 通信安全风险

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:303-317
ErrCode BundleManagerShellCommand::Init()
{
    if (bundleMgrProxy_ == nullptr) {
        bundleMgrProxy_ = BundleCommandCommon::GetBundleMgrProxy();
        if (bundleMgrProxy_) {
            if (bundleInstallerProxy_ == nullptr) {
                bundleInstallerProxy_ = bundleMgrProxy_->GetBundleInstaller();
            }
        }
    }

    if ((bundleMgrProxy_ == nullptr) || (bundleInstallerProxy_ == nullptr) ||
        (bundleInstallerProxy_->AsObject() == nullptr)) {
        result = OHOS::ERR_INVALID_VALUE;
    }
    return result;
}

// frameworks/src/bundle_command.cpp:2367-2373
sptr<BundleDeathRecipient> recipient(
    new (std::nothrow) BundleDeathRecipient(statusReceiver));
if (recipient == nullptr) {
    return;
}
bundleInstallerProxy_->AsObject()->AddDeathRecipient(recipient);
```

**问题**:
- IPC 代理在 Init() 中创建，但缺少对服务身份的验证
- DeathRecipient 用于监控服务死亡，但可能被利用进行 DoS

**触发条件**:
```bash
# 如果 BundleManagerService 被恶意停止
bm install -p app.hap  # 可能异常终止
```

**影响**:
- 服务冒充攻击
- 拒绝服务

**修复建议**:
- 验证 IPC 服务端身份
- 增加 IPC 调用超时机制
- 完善异常处理

---

#### 7. 资源耗尽风险

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:60-62
const int32_t MINIMUM_WAITTING_TIME = 180; // 3 mins
const int32_t MAXIMUM_WAITTING_TIME = 600; // 10 mins

// frameworks/src/bundle_command.cpp:786-802
int32_t installResult = InstallOperation(bundlePath, installParam, waittingTime, resultMsg);
if (installResult == OHOS::ERR_OK) {
    resultReceiver_ = STRING_INSTALL_BUNDLE_OK + "\n";
    // ...
}
```

**问题**:
- 等待时间最长可达 10 分钟，期间资源被占用
- 缺少对 HAP 文件大小的预检查
- 并发安装可能导致资源耗尽

**触发条件**:
```bash
# 同时发起多个长时间安装
bm install -p large1.hap -w 600 &
bm install -p large2.hap -w 600 &
bm install -p large3.hap -w 600 &
```

**影响**:
- 内存耗尽
- 存储空间耗尽
- 拒绝服务

**修复建议**:
- 限制并发安装数量
- 增加文件大小预检查
- 优化等待机制，支持取消操作

---

#### 8. UDID 信息泄露

**证据**:
```cpp
// frameworks/include/bundle_command.h:304
std::string GetUdid() const;

// 命令行暴露
bm get -u
```

**问题**: 设备 UDID 可被任意 shell 用户获取

**触发条件**:
```bash
bm get -u
# 输出: udid of current device is : 23CADE0C...
```

**影响**:
- 设备追踪
- 隐私泄露
- 设备指纹识别

**修复建议**:
- 增加权限检查 (如 ohos.permission.GET_UDID)
- 限制 UDID 获取范围
- 考虑使用匿名化标识

---

### 🟢 低风险

#### 9. 日志信息泄露

**证据**:
```gn
// frameworks/BUILD.gn:24-26
defines = [
  "APP_LOG_TAG = \"BMSTool\"",
  "LOG_DOMAIN = 0xD001123",
]

// 代码中使用 (bundle_command.cpp:21, 601, 689)
APP_LOGI("bm exec start");
APP_LOGD("option: %{public}d, optopt: %{public}d, optind: %{public}d", ...);
APP_LOGE("bm install with error userId %{private}s", optarg);
```

**问题**:
- 日志标签固定，便于过滤攻击
- 部分日志包含敏感信息 (如 userId)
- DEBUG 日志可能暴露内部状态

**触发条件**:
```bash
# 查看日志
hilog | grep BMSTool
```

**影响**:
- 运行信息泄露
- 调试信息暴露
- 攻击者了解系统内部状态

**修复建议**:
- 控制日志级别，生产环境关闭 DEBUG
- 敏感信息使用 %{private}s 脱敏
- 考虑动态日志标签

---

#### 10. 错误信息泄露

**证据**:
```cpp
// frameworks/src/bundle_command.cpp:230-259
std::map<int32_t, int32_t> BundleManagerShellCommand::errCodeMap_ = {
    {ERR_APPEXECFWK_INSTALL_VERSION_DOWNGRADE, IStatusReceiver::ERR_INSTALL_VERSION_DOWNGRADE},
    {ERR_APPEXECFWK_INSTALL_FAILED_INCONSISTENT_SIGNATURE, IStatusReceiver::ERR_INSTALL_FAILED_INCONSISTENT_SIGNATURE},
    // ... 详细错误映射
};

// frameworks/src/bundle_command_common.cpp:87-795
std::map<int32_t, std::string> BundleCommandCommon::bundleMessageMap_ = {
    {IStatusReceiver::ERR_INSTALL_PARSE_FAILED, "error: install parse failed."},
    {IStatusReceiver::ERR_INSTALL_VERSION_DOWNGRADE, "error: install version downgrade."},
    // ... 详细错误信息
};
```

**问题**:
- 错误信息过于详细，可能泄露系统内部状态
- 错误码映射暴露内部处理逻辑

**触发条件**:
```bash
bm install -p invalid.hap
# 返回: "error: install parse failed."
# 或: "error: signature verification failed due to bad public key."
```

**影响**:
- 系统信息泄露
- 攻击面暴露
- 帮助攻击者构造针对性攻击

**修复建议**:
- 对用户返回泛化错误信息
- 详细错误记录到安全日志
- 区分内部错误和用户错误

---

## 安全机制

### 已有安全机制

| 机制 | 实现位置 | 证据 | 说明 |
|------|----------|------|------|
| **编译器安全选项** | BUILD.gn | `frameworks/BUILD.gn:29-39` | sanitize (boundary, cfi, integer_overflow, ubsan), stack-protector-strong |
| **IPC 身份验证** | IPC 框架 | `bundle_command_common.cpp:34-51` | 通过 SystemAbilityManager 获取服务代理 |
| **DeathRecipient** | bundle_command.cpp | `bundle_command.cpp:2367-2373` | 监控远程服务死亡，防止悬挂调用 |
| **权限参数检查** | bundle_command.cpp | `bundle_command.cpp:688-706` | 用户 ID 和等待时间的范围检查 |
| **签名验证** | BundleManagerService | `bundle.json:38` | 依赖 appverify 组件 |
| **权限框架** | access_token | `bundle_test_tool.cpp:5462-5481` | 使用 AccessTokenKit 进行权限管理 |
| **SELinux** | selinux_adapter | `bundle.json:34` | 依赖 selinux_adapter 组件 |

### 安全机制详情

#### 编译器安全选项
```gn
// frameworks/BUILD.gn:29-39
sanitize = {
  boundary_sanitize = true      # 边界检查
  cfi = true                    # 控制流完整性
  cfi_cross_dso = true          # 跨 DSO CFI
  integer_overflow = true       # 整数溢出检查
  ubsan = true                  # 未定义行为检查
}
cflags = [ "-fstack-protector-strong" ]  # 栈保护
```

#### 输入验证实现
```cpp
// frameworks/src/bundle_command.cpp:807-819
ErrCode BundleManagerShellCommand::GetBundlePath(const std::string& param,
    std::vector<std::string>& bundlePaths) const
{
    if (param.empty()) {
        return OHOS::ERR_INVALID_VALUE;
    }
    // 检查参数是否为选项关键字
    if (param == "-r" || param == "--replace" || param == "-p" || ...) {
        return OHOS::ERR_INVALID_VALUE;
    }
    bundlePaths.emplace_back(param);
    return OHOS::ERR_OK;
}
```

### 缺失的安全机制

| 机制 | 建议优先级 | 缺失证据 | 说明 |
|------|----------|----------|------|
| 路径规范化 | 高 | `bundle_command.cpp:807-819` 仅检查选项关键字 | 缺少 `../` 遍历检查 |
| 字符串长度限制 | 高 | 未发现长度检查 | 包名、路径等参数无长度限制 |
| 统一权限检查 | 高 | 使用 `GetIntParameter(IS_ROOT_MODE_PARAM)` | 应使用 AccessToken 框架 |
| 审计日志 | 中 | 仅使用 hilog 日志 | 缺少安全审计日志 |
| IPC 身份校验 | 中 | 未发现服务证书校验 | 应验证 IPC 服务端身份 |

---

## 安全建议

### 短期（必须）

1. **路径验证强化** (对应风险 #1)
   - 实现路径规范化，使用 `realpath()` 或等价函数
   - 检查路径遍历模式 (`../`)
   - 验证路径在允许的白名单范围内
   ```cpp
   bool IsPathAllowed(const std::string& path) {
       std::vector<std::string> allowedPrefixes = {
           "/data/local/tmp/",
           "/data/app/el1/bundle/public/"
       };
       // 检查路径前缀
   }
   ```

2. **bundle_test_tool 权限限制** (对应风险 #2)
   - 限制 bundle_test_tool 的发布范围，仅限内部测试
   - 增加额外的权限检查层
   - 对特权操作增加审计日志
   - 考虑使用 Linux capabilities 替代 setuid

3. **权限检查统一化** (对应风险 #4)
   - 使用 AccessToken 框架替代系统参数检查
   - 统一权限检查接口
   - 权限不足时返回明确错误，不静默处理

### 中期（建议）

4. **输入验证完整性** (对应风险 #5)
   - 限制字符串参数长度 (如包名 ≤ 128 字符)
   - 使用正则表达式验证包名格式 (`^[a-zA-Z][a-zA-Z0-9_]*(\.[a-zA-Z][a-zA-Z0-9_]*)+$`)
   - 增加特殊字符过滤

5. **IPC 安全强化** (对应风险 #6)
   - 验证 IPC 服务端身份证书
   - 增加 IPC 调用超时和重试机制
   - 完善异常处理和资源清理

6. **资源管理优化** (对应风险 #7)
   - 限制并发安装数量 (如最多 3 个)
   - 增加 HAP 文件大小预检查 (如 ≤ 500MB)
   - 支持安装操作取消

7. **敏感信息保护** (对应风险 #8, #9, #10)
   - UDID 获取增加权限检查
   - 生产环境关闭 DEBUG 级别日志
   - 对用户返回泛化错误信息

### 长期（规划）

8. **架构安全改进**
   - 考虑将 bundle_tool 运行在受限沙箱
   - 使用 SELinux 策略限制文件访问范围
   - 实现操作审计日志系统
   - 定期进行安全渗透测试

9. **代码安全加固**
   - 启用更多编译器安全选项 (-D_FORTIFY_SOURCE=2, -fstack-protector-strong 已启用)
   - 引入静态代码分析工具 (如 CodeQL, SonarQube)
   - 建立安全编码规范

---

## 验证范围说明

### 检查范围

| 组件 | 是否检查 | 说明 |
|------|----------|------|
| bundle_tool 源码 | ✅ | 主要分析对象 |
| frameworks/src | ✅ | 核心逻辑 |
| GN 构建配置 | ✅ | 构建安全 |
| bundle_framework | ❌ | 独立仓库，仅分析接口 |

### 局限性

1. **依赖分析**: 基于代码静态分析，未进行动态测试
2. **边界测试**: 未进行实际安全测试
3. **完整评估**: 需结合渗透测试

---

## 相关文档

- [01_Architecture.md](./01_Architecture.md) - 架构设计
- [04_Build.md](./04_Build.md) - 构建系统
- [06_Troubleshooting.md](./06_Troubleshooting.md) - 问题定位
