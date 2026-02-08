# OpenHarmony Updater 安全风险评审

## 目的

本文档对 Updater 子系统进行安全风险评审，识别攻击面、信任边界和可被利用点。

## 适用范围

- 安全工程师
- 安全审计人员
- 架构师

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    可信区域 (Trusted)                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Bootloader  │  │   Kernel    │  │   Updater 分区      │ │
│  │  (签名验证)  │  │  (只读)     │  │  (受保护的分区镜像)  │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (Misc 分区传递命令)
┌─────────────────────────────────────────────────────────────┐
│                   不可信区域 (Untrusted)                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  正常系统    │  │  升级包文件  │  │   外部存储          │ │
│  │  (可能被入侵) │  │  (来源不明)  │  │  (SD卡/USB)        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|---------|------|
| **升级包** | 输入验证 | 高 | 恶意构造的升级包 |
| **Misc 分区** | 访问控制 | 高 | 未授权写入升级命令 |
| **分区操作** | 特权提升 | 高 | 分区擦写/格式化 |
| **脚本引擎** | 代码执行 | 中 | 脚本注入/越权操作 |
| **文件系统** | 路径遍历 | 中 | 非法路径访问 |
| **网络下载** | 中间人攻击 | 中 | 升级包篡改（若支持） |
| **HDC 调试** | 信息泄露 | 低 | 调试接口暴露 |
| **日志系统** | 信息泄露 | 低 | 敏感信息泄露 |

## 安全机制

### 1. 升级包签名验证

**机制**: 使用 PKCS#7 签名验证升级包完整性。

```cpp
// services/package/pkg_verify/pkg_verify_util.cpp:41-47
int32_t PkgVerifyUtil::VerifySourceDigest(std::vector<uint8_t> &signature, 
                                          std::vector<uint8_t> &sourceDigest,
                                          const std::string &keyPath) {
    Pkcs7SignedData pkcs7;
    std::vector<X509SignedInfo> sigs;
    int32_t ret = pkcs7.ReadSig(signature.data(), signature.size(), sigs);
    // ... 验证签名
}
```

**验证链**:
1. 解析 PKCS#7 签名块
2. 提取证书链
3. 验证证书链到根证书
4. 验证文件摘要

**证据**: `services/package/pkg_verify/pkcs7_signed_data.cpp:66-67`

### 2. 文件哈希验证

**机制**: 每个文件在包内有独立的哈希值，提取时验证。

```cpp
// services/package/pkg_verify/hash_data_verifier.cpp:125-178
bool HashDataVerifier::Verify() {
    // 1. 获取签名
    ret = verifyUtil.GetSignature(PkgStreamImpl::ConvertPkgStream(pkgStream),
                                  signatureSize, signature, commentTotalLenAll);
    // 2. 解析 PKCS#7
    // 3. 验证每个文件的哈希
    return pkcs7_ != nullptr && pkcs7_->ParsePkcs7Data(signature.data(), 
                                                       signature.size()) == 0;
}
```

**证据**: `services/package/pkg_verify/hash_data_verifier.cpp:174-178`

### 3. 分区访问控制

**机制**: 通过设备节点权限控制分区访问。

```cpp
// utils/utils.cpp:1328
void SetFileAttributes(const std::string& file, uid_t owner, gid_t group, mode_t mode);
```

**证据**: 分区设备节点通常权限为 `0600` (root 只读/写)

### 4. Misc 分区保护

**机制**: Misc 分区设备节点限制为 root 可写。

```cpp
// utils/write_updater.cpp:141
static void HandleMiscInfo(int argc, char **argv) {
    // 写入 Misc 分区需要 root 权限
}
```

**证据**: `utils/write_updater.cpp` 需以 root 运行

## 可被利用点分析

### 1. 升级包解析漏洞

**证据**: `services/package/pkg_verify/zip_pkg_parse.cpp:180-199`

```cpp
int32_t PkgVerifyUtil::ParsePackage(const PkgStreamPtr pkgStream, 
                                    size_t &signatureStart,
                                    size_t &signatureSize, 
                                    uint16_t &commentTotalLenAll) const {
    // 解析 ZIP 包尾部
    if (fileLen < (signatureSize + ZIP_EOCD_FIXED_PART_LEN)) {
        PKG_LOGE("Invalid fileLen[%zu] and signature size[%zu]", fileLen, signatureSize);
        UPDATER_LAST_WORD(PKG_INVALID_PARAM, fileLen, signatureSize);
        return PKG_INVALID_PARAM;
    }
    // ...
}
```

**可利用路径**:
1. 构造畸形 ZIP 文件
2. 触发解析函数
3. 可能导致缓冲区溢出或拒绝服务

**影响**: 代码执行（解析漏洞）或拒绝服务

**修复建议**:
- 增加更多边界检查
- 使用安全的解析库
- 限制文件大小

### 2. 脚本注入

**证据**: `services/script/script_interpreter/script_interpreter.cpp`

```cpp
// 脚本引擎解析并执行指令
// 如: write_raw_image("/data/image", "system")
```

**可利用路径**:
1. 通过漏洞修改升级包中的脚本
2. 注入恶意指令
3. 执行未授权的分区操作

**影响**: 代码执行、数据破坏

**修复建议**:
- 脚本签名验证（已有）
- 指令白名单校验
- 沙箱执行环境

### 3. 路径遍历

**证据**: `services/script/script_instruction/script_updateprocesser.cpp`

```cpp
// 从升级包提取文件到指定路径
// package_extract_file("file_in_pkg", "/target/path")
```

**可利用路径**:
1. 构造包含 `../` 的目标路径
2. 文件被提取到未授权位置
3. 覆盖系统文件

**影响**: 文件系统污染、提权

**修复建议**:
- 路径规范化
- 禁止 `..` 路径组件
- 目标路径白名单

### 4. 整数溢出

**证据**: `services/package/pkg_manager/pkg_stream.cpp`

```cpp
// 文件长度和偏移量使用 size_t
size_t fileLen = pkgStream->GetFileLength();
size_t signatureStart = pkgStream->GetFileLength() - pkgSignComment.signCommentAppendLen;
```

**可利用路径**:
1. 构造极大或负值长度字段
2. 触发整数溢出
3. 缓冲区溢出或信息泄露

**影响**: 代码执行或信息泄露

**修复建议**:
- 严格范围检查
- 使用安全整数类型
- 溢出检测

### 5. 竞态条件

**证据**: `services/fs_manager/mount.cpp`

```cpp
// 挂载和卸载操作
int MountPartition(const std::string &partition, const std::string &mountPoint, ...);
int UmountPartition(const std::string &mountPoint);
```

**可利用路径**:
1. 利用挂载/卸载之间的时间窗口
2. 符号链接竞争
3. 访问错误挂载点

**影响**: 未授权文件访问

**修复建议**:
- 原子操作
- 文件描述符验证
- 禁用符号链接跟随

## 安全建议

### 1. 输入验证强化

| 位置 | 建议 |
|------|------|
| 包解析 | 增加长度字段上限检查 |
| 路径处理 | 使用 `realpath()` 规范化路径 |
| 脚本解析 | 增加指令参数校验 |
| 分区操作 | 验证分区名白名单 |

### 2. 权限控制

```cpp
// 建议: 在关键操作前检查权限
bool CheckPrivileged() {
    return getuid() == 0;
}

// 建议: 限制文件权限
void SecureFile(const std::string& path) {
    chmod(path.c_str(), 0600);
    chown(path.c_str(), 0, 0);
}
```

### 3. 日志脱敏

```cpp
// 避免在日志中记录敏感信息
// 不推荐:
LOGI("Key: %s", privateKey.c_str());

// 推荐:
LOGI("Key loaded, length: %zu", privateKey.length());
```

### 4. 安全编译选项

```gn
# 建议增加以下编译选项
cflags += [
    "-fstack-protector-strong",    # 栈保护
    "-D_FORTIFY_SOURCE=2",         # 缓冲区检查
    "-fPIE",                       # 位置无关代码
]

ldflags += [
    "-pie",                        # PIE 链接
    "-z,relro",                    # 重定位只读
    "-z,now",                      # 立即绑定
]
```

## 检查范围与局限性

### 已检查范围

- ✅ 包解析和验证逻辑
- ✅ 脚本引擎执行流程
- ✅ 分区操作接口
- ✅ 文件系统操作
- ✅ Misc 分区访问
- ✅ 日志系统

### 局限性

- ❌ 未深入 Rust 代码 (`services/rust/`)
- ❌ 未检查 HDI 接口实现
- ❌ 未分析 UI 模块的安全问题
- ❌ 未检查驱动层安全
- ❌ 未分析 Bootloader 交互安全

## 关键结论

1. **签名验证是核心防线**: 升级包的 PKCS#7 签名验证是第一道也是最重要的防线，确保升级包来源可信。

2. **权限控制依赖系统**: Updater 依赖系统的 root 权限控制和设备节点权限，需确保系统配置正确。

3. **输入验证需加强**: 包解析、路径处理、脚本执行等位置的输入验证可进一步加强。

4. **无网络直接暴露**: Updater 本身不直接处理网络下载，由上层服务处理，降低了网络攻击面。

5. **代码审计建议**: 建议对关键路径（包解析、脚本引擎）进行更深入的代码审计和 Fuzz 测试。

## 相关跳转

- [架构说明](./01_Architecture.md#安全模型)
- [包管理内部接口](./04_Inner_API.md#pkg_manager)
- [对外 API](./03_Public_API.md)
