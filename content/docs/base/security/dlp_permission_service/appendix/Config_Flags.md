# 配置宏与特性开关 (Config Flags)

## 目的

本文档列出 DLP 权限服务的关键编译宏和 feature flags，帮助开发者理解编译时配置选项。

## 适用范围

- 目标读者：构建工程师、平台配置者
- 覆盖内容：GN feature flags、编译宏、调试选项

---

## GN Feature Flags

### dlp_permission_service.gni

**文件路径**：根目录

**全局变量**：

| Flag | 默认值 | 条件 | 说明 |
|------|---------|------|------|
| `dlp_permission_service_gathering_policy` | false | - | 聚合沙箱策略（是否启用数据聚合功能） |
| `dlp_parse_inner` | true | `global_parts_info.account_os_account` 存在 | 启用内部 DLP 文件解析特性（账户集成） |
| `dlp_credential_enable` | true | `global_parts_info.security_dlp_credential_service` 存在 | 启用 DLP 凭证服务支持 |
| `dlp_permission_service_credential_connection_enable` | true | - | 启用凭证连接功能 |
| `dlp_file_version_inner` | true | - | 启用内部 DLP 文件版本 |

**代码证据**：
- dlp_permission_service.gni:18-36

### identify_sensitive_content.gni

**文件路径**：根目录

**全局变量**：

| Flag | 默认值 | 条件 | 说明 |
|------|---------|------|------|
| `data_identify_anonymize_service_enable` | false | - | 启用数据识别和匿名化服务 |

**代码证据**：
- identify_sensitive_content.gni 文件内容

---

## 编译宏 (Defines)

### 根据 Feature Flags 生成的宏

| 宏 | 条件 | 用途 |
|-----|------|------|
| `DLP_GATHERING_SANDBOX` | `dlp_permission_service_gathering_policy` = true | 聚合策略功能 |
| `DLP_PARSE_INNER` | `dlp_parse_inner` = true | 内部解析功能（账户集成） |
| `SUPPORT_DLP_CREDENTIAL` | `dlp_credential_enable` = true | 凭证支持 |
| `DLP_FILE_VERSION_INNER` | `dlp_file_version_inner` = true | 内部文件版本 |

**代码证据**：
- feature flags 使用：interfaces/inner_api/dlp_parse/BUILD.gn

### 平台相关宏

| 宏 | 条件 | 用途 |
|-----|------|------|
| `IS_EMULATOR` | `is_emulator` = true | 模拟器特定代码 |
| `FILE_IDENTIFY_ENABLE` | `target_platform == "pc" && data_identify_anonymize_service_enable` | 文件识别功能（PC 平台） |
| `_ARM64_` | `current_cpu == "arm64"` | ARM64 架构特定代码 |

**代码证据**：
- 模拟器宏：interfaces/kits/dlp_permission/BUILD.gn:36-37
- 文件识别宏：interfaces/kits/identify_sensitive_content/BUILD.gn

---

## 调试宏

### 日志宏

| 宏 | 条件 | 用途 |
|-----|------|------|
| `HILOG_ENABLE` | 所有 N-API targets | 启用 HiLog 日志输出 |

**代码证据**：
- 日志宏：interfaces/kits/dlp_permission/BUILD.gn:53, interfaces/kits/dlp_permission/BUILD.gn:123

### 调试级别宏

| 宏 | 条件 | 用途 |
|-----|------|------|
| `DLP_DEBUG_ENABLE=0` | `build_variant == "user"` | 用户构建（调试关闭） |
| `DLP_DEBUG_ENABLE=1` | `build_variant == "root"` | Root 构建（调试开启） |

**代码证据**：
- 调试宏：config/BUILD.gn 或项目配置

---

## 安全相关宏

### 防御性宏

| 宏 | 条件 | 用途 |
|-----|------|------|
| `_FORTIFY_SOURCE=2` | 所有 targets | 启用运行时缓冲区溢出检测 |

**代码证据**：
- Fortify 宏：config/BUILD.gn, `common_build_options_flags` 配置

### Sanitize 选项

| 选项 | 应用范围 | 用途 |
|------|---------|------|
| `sanitize: integer_overflow = true` | libdlppermission_napi.so, libdlpsetdlpfeature_napi.so | 整数溢出检测 |
| `sanitize: cfi = true` | 同上 | 控制流完整性检查 |
| `sanitize: cfi_cross_dso = true` | 同上 | 跨 DSO CFI 检查 |
| `branch_protector_ret = "pac_ret"` | 同上 | PAC 返回地址保护 |

**代码证据**：
- Sanitize 配置：interfaces/kits/dlp_permission/BUILD.gn:20-25, interfaces/kits/dlp_permission/BUILD.gn:99-104

---

## 运行时配置

### 服务参数 (dlp_permission.para)

**文件路径**：services/dlp_permission/sa/etc/dlp_permission.para

**配置项**：
- `gathering.policy = false` - 聚合策略开关

**代码证据**：
- 服务配置：services/dlp_permission/sa/etc/BUILD.gn

### 支持的文件类型 (dlp_config.json)

**文件路径**：services/dlp_permission/sa/etc/dlp_config.json

**配置项**：DLP 支持的文件类型列表

**代码证据**：
- 文件类型配置：services/dlp_permission/sa/etc/dlp_config.json

---

## 配置影响

### 功能影响

| Flag/宏 | 影响 |
|----------|------|
| `dlp_permission_service_gathering_policy` | 启用/禁用数据聚合功能，影响 `GetDlpGatheringPolicy()` 接口 |
| `dlp_parse_inner` | 启用账户集成，影响 DLP 文件解析和访问记录 |
| `dlp_credential_enable` | 启用凭证服务连接，影响证书生成/解析 |
| `SUPPORT_DLP_CREDENTIAL` | 条件编译凭证相关代码 |
| `FILE_IDENTIFY_ENABLE` | 启用敏感内容识别模块（PC 平台） |

### 构建产物影响

| Flag/宏 | 影响的产物 |
|----------|------------|
| `dlp_credential_enable` = false | libdlppermission_napi.so 使用 dlp_connection_static_mock.cpp |
| `support_jsapi` = false | libdlppermission_napi.so 不编译 |
| `is_standard_system` = false | libdlp_permission_service.z.so 不编译 |

---

## 相关跳转链接

- [GN Targets](05_GN_Targets.md) - 查看完整的构建配置
- [编译产物](06_Build_Artifacts.md) - 查看产物和安装路径

---

最后更新时间：2026-02-06
