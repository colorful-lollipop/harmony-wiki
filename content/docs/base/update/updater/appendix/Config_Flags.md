# 配置标志附录

## 目的

本文档汇总 Updater 子系统所有编译配置标志和宏定义。

## 适用范围

- 构建工程师
- 子系统开发者

## GN Feature 标志

### 位置: `updater_default_cfg.gni`

| 标志 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `init_feature_ab_partition` | declare_args | `true` | AB 分区支持（来自 init 配置） |
| `updater_feature_use_ptable` | declare_args | `true` | 使用分区表 |
| `updater_feature_updater_gen_executable` | declare_args | `false` | 生成可执行文件 |
| `updater_feature_sign_on_server` | declare_args | `true` | 服务器端签名验证 |
| `updater_hdc_depend` | declare_args | `true` | HDC 调试依赖 |
| `updater_ui_support` | 派生 | `!ohos_indep_compiler_enable` | UI 支持 |
| `updater_sign_on_server` | 派生 | `updater_feature_sign_on_server` | 服务器签名标志 |

### 位置: 全局配置

| 标志 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `build_ohos` | declare_args | `true` | 构建 OHOS |
| `updater_build_variant_user` | declare_args | `true` | User 版本构建 |
| `ohos_indep_compiler_enable` | 全局 | `false` | 独立编译器模式 |

## 编译宏定义

### 全局定义

| 宏 | 定义位置 | 说明 | 条件 |
|----|---------|------|------|
| `BUILD_OHOS` | `updater_default_cfg.gni` | 构建 OHOS | 始终定义 |
| `UPDATER_USE_PTABLE` | `services/BUILD.gn` | 使用分区表 | `updater_feature_use_ptable` |
| `UPDATER_BUILD_VARIANT_USER` | `services/BUILD.gn` | User 版本 | `build_variant == "user"` |
| `UPDATER_UI_SUPPORT` | `services/BUILD.gn` | UI 支持 | `updater_ui_support` |
| `UPDATER_AB_SUPPORT` | `services/BUILD.gn` | AB 分区支持 | `init_feature_ab_partition` |
| `SIGN_ON_SERVER` | `services/BUILD.gn` | 服务器签名 | `updater_feature_sign_on_server` |
| `OPENSSL_SUPPRESS_DEPRECATED` | 多处 | 抑制 OpenSSL 弃用警告 | 始终定义 |

### 模块特定定义

| 宏 | 模块 | 说明 |
|----|------|------|
| `DIFF_PATCH_SDK` | `services/diffpatch/BUILD.gn` | 差分补丁 SDK 模式 |
| `WITH_SELINUX` | `services/flashd/BUILD.gn` | SELinux 支持 |
| `SURPPORT_SELINUX` | `services/flashd/BUILD.gn` | SELinux 支持（拼写保持） |

### 头文件中的条件编译

```cpp
// interfaces/kits/include/slot_info/slot_info.h
#ifdef UPDATER_AB_SUPPORT
    // AB 分区支持代码
#else
    // 空实现
#endif

// services/include/updater/updater_const.h
#ifdef UPDATER_UI_SUPPORT
    // UI 相关常量
#endif

// services/package/pkg_verify/*.cpp
#ifdef SIGN_ON_SERVER
    // 服务器签名验证代码
#endif
```

## 常量定义

### Misc 分区相关

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `MAX_COMMAND_SIZE` | 32 | `misc_info.h:24` | 命令字段最大长度 |
| `MAX_STATUS_SIZE` | 32 | `misc_info.h:25` | 状态字段最大长度 |
| `MAX_UPDATE_SIZE` | 768 | `misc_info.h:26` | 升级路径字段最大长度 |
| `MAX_STAGE_SIZE` | 32 | `misc_info.h:27` | 阶段字段最大长度 |
| `MAX_FAULTINFO_SIZE` | 32 | `misc_info.h:28` | 故障信息字段最大长度 |
| `MAX_RESERVED_SIZE` | 224 | `misc_info.h:29` | 保留字段最大长度 |
| `MISC_BASE_OFFSET` | 0 | `misc_info.h:35` | Misc 分区基地址偏移 |
| `MISC_UPDATER_PARA_OFFSET` | 1MB | `misc_info.h:47` | Updater 参数偏移 |

### 升级类型常量

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `UPGRADE_TYPE_OTA` | "ota" | `updaterkits.h:21` | 标准 OTA 升级 |
| `UPGRADE_TYPE_SD` | "sdcard" | `updaterkits.h:22` | SD 卡升级 |
| `UPGRADE_TYPE_OTA_INTRAL` | "ota_intral" | `updaterkits.h:23` | 内部 OTA 升级 |
| `UPGRADE_TYPE_SD_INTRAL` | "sdcard_intral" | `updaterkits.h:24` | 内部 SD 卡升级 |
| `UPGRADE_TYPE_SUBPKG_UPDATE` | "subpkg_update" | `updaterkits.h:25` | 子包升级 |

### 包类型常量

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `PKG_PACK_TYPE_NONE` | 0 | `package.h` | 无类型 |
| `PKG_PACK_TYPE_UPGRADE` | 0 | `package.h` | 升级包 |
| `PKG_PACK_TYPE_ZIP` | 1 | `package.h` | ZIP 包 |
| `PKG_PACK_TYPE_LZ4` | 2 | `package.h` | LZ4 包 |
| `PKG_PACK_TYPE_GZIP` | 3 | `package.h` | GZIP 包 |

### 压缩方法常量

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `PKG_COMPRESS_NONE` | -1 | `package.h` | 无压缩 |
| `PKG_COMPRESS_ZSTD` | 0 | `package.h` | Zstd 压缩 |
| `PKG_COMPRESS_LZ4` | 1 | `package.h` | LZ4 压缩 |
| `PKG_COMPRESS_ZIP` | 2 | `package.h` | ZIP 压缩 |
| `PKG_COMPRESS_GZIP` | 3 | `package.h` | GZIP 压缩 |

### 摘要算法常量

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `PKG_DIGEST_TYPE_CRC` | 0 | `package.h` | CRC 校验 |
| `PKG_DIGEST_TYPE_SHA256` | 1 | `package.h` | SHA-256 |
| `PKG_DIGEST_TYPE_SHA384` | 2 | `package.h` | SHA-384 |
| `PKG_DIGEST_TYPE_SHA512` | 3 | `package.h` | SHA-512 |

### 签名方法常量

| 常量 | 值 | 位置 | 说明 |
|------|-----|------|------|
| `PKG_SIGN_METHOD_RSA` | 0 | `package.h` | RSA 签名 |
| `PKG_SIGN_METHOD_ECDSA` | 1 | `package.h` | ECDSA 签名 |

## 错误码

### Updater 状态码

| 错误码 | 值 | 位置 | 说明 |
|--------|-----|------|------|
| `UPDATE_ERROR` | -1 | `updater.h:27` | 通用错误 |
| `UPDATE_SUCCESS` | 0 | `updater.h:28` | 成功 |
| `UPDATE_CORRUPT` | 1 | `updater.h:29` | 包损坏或校验失败 |
| `UPDATE_SKIP` | 2 | `updater.h:30` | 跳过升级 |
| `UPDATE_RETRY` | 3 | `updater.h:31` | 可重试 |
| `UPDATE_RETRY_FAIL` | 4 | `updater.h:32` | 重试失败 |
| `UPDATE_SPACE_NOTENOUGH` | 5 | `updater.h:33` | 空间不足 |
| `UPDATE_UNKNOWN` | 6 | `updater.h:34` | 未知错误 |

### 包错误码

| 错误码 | 说明 |
|--------|------|
| `PKG_SUCCESS` | 成功 |
| `PKG_INVALID_PARAM` | 无效参数 |
| `PKG_INVALID_SIGNATURE` | 无效签名 |
| `PKG_INVALID_DIGEST` | 摘要验证失败 |

## 路径常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `MISC_PATH` | `/dev/block/misc` | Misc 分区路径 |
| `UPDATER_PATH` | `/updater` | Updater 工作目录 |
| `STREAM_ZIP_PATH` | `/updater/ota_package/update.zip` | 流式升级包路径 |

## 配置组合示例

### 标准配置

```gn
# 标准设备配置
init_feature_ab_partition = true
updater_feature_use_ptable = true
updater_feature_sign_on_server = true
updater_hdc_depend = true
```

### 轻量配置（无 UI）

```gn
# 轻量设备配置
ohos_indep_compiler_enable = true  # 禁用 UI
updater_feature_use_ptable = false  # 不使用分区表
updater_feature_sign_on_server = false  # 本地签名
```

### 调试配置

```gn
# 调试配置
updater_build_variant_user = false  # 非 user 版本，保留调试功能
updater_hdc_depend = true  # HDC 支持
```

## 相关跳转

- [GN 构建系统](../05_GN_Build.md)
- [项目概览](../00_Overview.md)
- [安全分析](../06_Security_Analysis.md)
