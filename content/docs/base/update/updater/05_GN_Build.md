# OpenHarmony Updater GN 构建系统

## 目的

本文档详细说明 Updater 子系统的 GN 构建配置，包括 targets、依赖关系、编译产物和配置开关。

## 适用范围

- 构建工程师
- 子系统开发者
- 系统集成工程师

## 构建文件总览

### 文件位置

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 根构建文件，定义测试分组 |
| `updater_default_cfg.gni` | 全局默认配置 |
| `services/BUILD.gn` | 主服务构建 |
| `services/*/BUILD.gn` | 各模块构建文件 |
| `interfaces/kits/*/BUILD.gn` | 接口库构建文件 |
| `utils/BUILD.gn` | 工具库构建文件 |

## 关键 Targets

### 可执行程序

| Target | 位置 | 说明 | 安装镜像 |
|--------|------|------|---------|
| `updater` | `services/BUILD.gn` | 主 Updater 程序 | updater |
| `updater_binary` | `services/updater_binary/BUILD.gn` | 升级二进制程序 | updater |
| `updater_reboot` | `utils/BUILD.gn` | 重启工具 | updater |
| `write_updater` | `utils/BUILD.gn` | 写入 Misc 工具 | - |
| `diff` | `services/diffpatch/BUILD.gn` | 差分工具 | - |

### 静态库

| Target | 位置 | 说明 | 依赖 |
|--------|------|------|------|
| `libupdater_static` | `services/BUILD.gn` | 主静态库 | 全模块 |
| `libupdater` | `services/BUILD.gn` | Updater 库 | - |
| `libupdater_sys_installer` | `services/BUILD.gn` | sys_installer 库 | - |
| `libupdater_binary` | `services/updater_binary/BUILD.gn` | 二进制库 | - |
| `libapplypatch` | `services/applypatch/BUILD.gn` | 补丁应用 | - |
| `libfsmanager` | `services/fs_manager/BUILD.gn` | 文件系统管理 | - |
| `libupdaterpackage` | `services/package/BUILD.gn` | 包管理 | libringbuffer |
| `libupdaterscript` | `services/script/BUILD.gn` | 脚本引擎 | libthreadpool |
| `libthreadpool` | `services/script/BUILD.gn` | 线程池 | - |
| `libupdaterlog` | `services/log/BUILD.gn` | 日志 | - |
| `libui` | `services/ui/BUILD.gn` | UI | - |
| `libflashd` | `services/flashd/BUILD.gn` | Flashd | - |
| `flashd_deamon` | `services/flashd/BUILD.gn` | Flashd 守护进程 | - |
| `libbinchunkupdate` | `services/stream_update/BUILD.gn` | 流式更新 | - |
| `libBinFlowUpdate` | `services/flow_update/update_bin/BUILD.gn` | 流式二进制更新 | - |
| `libwritestate` | `services/write_state/BUILD.gn` | 状态写入 | - |
| `libptableparse` | `services/ptable_parse/BUILD.gn` | 分区表解析 | - |
| `libpatch` | `services/diffpatch/patch/BUILD.gn` | 补丁库 | - |
| `libringbuffer` | `services/common/ring_buffer/BUILD.gn` | 环形缓冲区 | - |
| `libutils` | `utils/BUILD.gn` | 通用工具 | - |
| `libutils_fs` | `utils/BUILD.gn` | 文件系统工具 | - |
| `libutils_json` | `utils/BUILD.gn` | JSON 工具 | - |
| `libutils_common` | `utils/BUILD.gn` | 通用工具 | - |
| `libpackageExt` | `interfaces/kits/packages/BUILD.gn` | 包扩展 | - |
| `libupdaterkits` | `interfaces/kits/updaterkits/BUILD.gn` | Updater Kits | - |
| `libmiscinfo` | `interfaces/kits/misc_info/BUILD.gn` | Misc 信息 | - |
| `libslotinfo` | `interfaces/kits/slot_info/BUILD.gn` | 槽位信息 | - |
| `libdiff_patch` | `interfaces/kits/diff_patch/BUILD.gn` | 差分补丁 | - |
| `rust_hash_signed_data` | `services/rust/hash_signed_data/BUILD.gn` | Rust 哈希签名 | - |

### 共享库

| Target | 位置 | 说明 | 安装镜像 |
|--------|------|------|---------|
| `libupdaterlog_shared` | `services/log/BUILD.gn` | 日志共享库 | system, updater |
| `libupdaterpackage_shared` | `services/package/BUILD.gn` | 包管理共享库 | system, updater |
| `libverify_shared` | `services/package/BUILD.gn` | 验证共享库 | - |
| `libpatch_shared` | `services/diffpatch/patch/BUILD.gn` | 补丁共享库 | - |
| `libpackage_shared` | `interfaces/kits/packages/BUILD.gn` | 包接口共享库 | system, updater |
| `libupdater_shared` | `interfaces/kits/updaterkits/BUILD.gn` | Updater 共享库 | system, updater |
| `libdiff_patch_shared` | `interfaces/kits/diff_patch/BUILD.gn` | 差分补丁共享库 | system, updater |
| `libupdate_hdi_impl` | `services/hdi/server/BUILD.gn` | HDI 实现共享库 | updater |

## 依赖关系

### 主 Updater 依赖

```
updater (executable)
└── libupdater_static
    ├── libpackageExt
    ├── libapplypatch
    │   └── libpatch
    ├── libpatch
    ├── libfsmanager
    ├── libupdaterlog
    ├── libupdaterpackage
    │   └── libringbuffer
    ├── libwritestate
    ├── libmiscinfo
    ├── libsdupdate
    ├── libslotinfo
    ├── libui (if updater_ui_support)
    └── libflashd (if !ohos_indep_compiler_enable)
        ├── flashd_deamon
        ├── libupdate_hdi_impl
        └── libBinFlowUpdate
```

### Updater Binary 依赖

```
updater_binary (executable)
└── libupdater_binary
    ├── libmiscinfo
    ├── libapplypatch
    ├── libpatch
    ├── libBinFlowUpdate
    ├── libfsmanager
    ├── libupdaterlog
    ├── libupdaterpackage
    ├── libupdaterscript
    ├── libbinchunkupdate
    └── libutils
```

## Feature 标志

### 配置位置

所有 Feature 标志定义在 `updater_default_cfg.gni`：

```gn
# updater_default_cfg.gni

# AB 分区支持（来自 init 配置）
declare_args() {
  init_feature_ab_partition = true
}

# Updater 特性开关
declare_args() {
  updater_feature_use_ptable = true
  updater_feature_updater_gen_executable = false
  updater_feature_sign_on_server = true
  updater_hdc_depend = true
}

# UI 支持
updater_ui_support =
    defined(ohos_indep_compiler_enable) && ohos_indep_compiler_enable == false

# 服务器签名
updater_sign_on_server = updater_feature_sign_on_server
```

### Feature 说明

| Feature | 默认 | 说明 | 影响 |
|---------|------|------|------|
| `init_feature_ab_partition` | true | AB 分区支持 | 启用 AB 分区切换功能 |
| `updater_feature_use_ptable` | true | 分区表支持 | 使用分区表解析分区 |
| `updater_feature_updater_gen_executable` | false | 生成可执行文件 | 控制生成特定可执行文件 |
| `updater_feature_sign_on_server` | true | 服务器签名 | 启用服务器端签名验证 |
| `updater_ui_support` | true | UI 支持 | 编译 UI 模块和依赖 |
| `updater_hdc_depend` | true | HDC 调试支持 | 启用 HDC 调试通道 |
| `updater_zlib_enable` | true | Zlib 支持 | 启用 Zlib 压缩（非独立编译器） |
| `ohos_indep_compiler_enable` | - | 独立编译器模式 | 禁用 Flashd 和 UI 功能 |

## 编译定义 (Defines)

### 全局定义

```gn
# updater_default_cfg.gni
declare_args() {
  build_ohos = true
  updater_build_variant_user = true
}

# services/BUILD.gn
defines = [
  "BUILD_OHOS",
  "UPDATER_USE_PTABLE",
  "OPENSSL_SUPPRESS_DEPRECATED",
]

if (build_variant == "user") {
  defines += [ "UPDATER_BUILD_VARIANT_USER" ]
}

if (updater_ui_support) {
  defines += [ "UPDATER_UI_SUPPORT" ]
}

if (init_feature_ab_partition) {
  defines += [ "UPDATER_AB_SUPPORT" ]
}

if (updater_feature_sign_on_server) {
  defines += [ "SIGN_ON_SERVER" ]
}
```

### 模块定义

| 模块 | 定义 | 说明 |
|------|------|------|
| diffpatch | `DIFF_PATCH_SDK` | SDK 模式编译 |
| flashd | `WITH_SELINUX`, `SURPPORT_SELINUX` | SELinux 支持 |
| package | `OPENSSL_SUPPRESS_DEPRECATED` | 抑制 OpenSSL 弃用警告 |

## 编译配置 (Configs)

### 共享库配置

```gn
# config/shared_library/BUILD.gn
config("updater_shared_config") {
  cflags = [
    "-fdata-sections",
    "-ffunction-sections",
  ]
  ldflags = [
    "-Wl,--exclude-libs,ALL",
    "-Wl,--gc-sections",
  ]
}
```

### UI 配置

```gn
# services/ui/BUILD.gn
config("updater_ui_support_cfg") {
  include_dirs = [
    "${updater_path}/resources/font",
    "${updater_path}/resources/image",
  ]
}
```

## 编译产物

### 输出路径

| 产物类型 | 输出路径 | 说明 |
|---------|---------|------|
| 可执行程序 | `out/<product>/updater/` | updater 分区产物 |
| 共享库 | `out/<product>/system/lib64/` | system 分区产物 |
| 静态库 | `out/<product>/obj/` | 中间产物 |
| 配置文件 | `out/<product>/updater/etc/` | 配置文件 |
| 资源文件 | `out/<product>/updater/resources/` | UI 资源 |

### 产物映射

```
// base/update/updater/bundle.json:76-98
"sub_component": [
    "//base/update/updater/resources:updater_resources",              → resources/
    "//base/update/updater/services/etc:updater_files",                → etc/*.cfg
    "//base/update/updater/services/package:libupdaterpackage",        → libupdaterpackage.a
    "//base/update/updater/services/package:libverify_shared",         → libverify_shared.so
    "//base/update/updater/services/script:libupdaterscript",          → libupdaterscript.a
    "//base/update/updater/services/log:libupdaterlog",                → libupdaterlog.a
    "//base/update/updater/services/updater_binary:updater_binary",    → updater_binary
    "//base/update/updater/services:updater",                          → updater
    "//base/update/updater/services/applypatch:libapplypatch",         → libapplypatch.a
    "//base/update/updater/services/fs_manager:libfsmanager",          → libfsmanager.a
    "//base/update/updater/utils:libutils",                            → libutils.a
    "//base/update/updater/utils:updater_reboot",                      → updater_reboot
    "//base/update/updater/utils:write_updater",                       → write_updater
    ...
]
```

### 运行时加载关系

```
Updater 分区启动时:
    init → 读取 init.cfg → 启动 updater
    
updater 进程加载:
    - libupdater_static.a (静态链接)
    - libupdaterlog_shared.so (动态链接)
    - libupdaterpackage_shared.so (动态链接)
    
正常系统进程加载:
    - libupdater_shared.so (UpdaterKits)
    - libpackage_shared.so (Packages)
    - libdiff_patch_shared.so (DiffPatch)
```

## 测试 Targets

### 单元测试

```
test/unittest:updater_unittest
test/unittest/applypatch_test:applypatch_unittest
test/unittest/common/ring_buffer:ring_buffer_test
test/unittest/factory_reset_test:factory_reset_unittest
test/unittest/flow_update/update_bin:bin_flow_update_test
test/unittest/package:package_unittest
test/unittest/script:script_unittest
test/unittest/stream_update:bin_chunk_update_test
test/unittest/updater_binary:binary_unittest
test/unittest/updater:updater_test
test/unittest/utils:utils_test
test/unittest/flashd_test:flashd_unittest (条件)
test/unittest/flashd_test:flashd_utils_unittest (条件)
test/unittest/service_test:updater_service_unittest (条件)
test/unittest/updater_ui_test:ui_unittest (条件)
```

### Fuzz 测试

```
test/fuzztest/UpdaterFormatPartition_fuzzer:UpdaterFormatPartitionFuzzTest
test/fuzztest/UpdaterMountForPath_fuzzer:UpdaterMountForPathFuzzTest
test/fuzztest/UpdaterStartUpdaterProc_fuzzer:UpdaterStartUpdaterProcFuzzTest
test/fuzztest/applypatch_fuzzer:ApplyPatchFuzzTest
test/fuzztest/binflow_fuzzer:BinFlowFuzzTest
test/fuzztest/dopartitions_fuzzer:DoPartitionsFuzzTest
test/fuzztest/extractandexecutescript_fuzzer:ExtractAndExecuteScriptFuzzTest
test/fuzztest/getupdatepackageinfo_fuzzer:GetUpdatePackageInfoFuzzTest
test/fuzztest/package_fuzzer:PackageFuzzTest
test/fuzztest/readfstabfromfile_fuzzer:ReadFstabFromFileFuzzTest
test/fuzztest/rebootandinstallupgradepackage_fuzzer:RebootAndInstallUpgradePackageFuzzTest
test/fuzztest/scriptmanager_fuzzer:ScriptManagerFuzzTest
test/fuzztest/updaterfactoryreset_fuzzer:UpdaterFactoryResetFuzzTest
test/fuzztest/updatermain_fuzzer:UpdaterMainFuzzTest
test/fuzztest/updaterutils_fuzzer:UpdaterUtilsFuzzTest
test/fuzztest/writeupdatermsg_fuzzer:WriteUpdaterMsgFuzzTest
```

## 构建命令示例

### 完整构建

```bash
# 生成 ninja 文件
gn gen out --root=.

# 构建整个 Updater
ninja -C out //base/update/updater/services:updater

# 构建所有测试
ninja -C out //base/update/updater:unittest
ninja -C out //base/update/updater:fuzztest
```

### 单独构建模块

```bash
# 构建包管理库
ninja -C out //base/update/updater/services/package:libupdaterpackage

# 构建对外接口
ninja -C out //base/update/updater/interfaces/kits/updaterkits:libupdater_shared
n
# 构建工具
ninja -C out //base/update/updater/utils:updater_reboot
```

## 关键结论

1. **静态库为主**: Updater 核心以静态库形式链接，减少运行时依赖。

2. **双镜像安装**: 共享库同时安装到 system 和 updater 分区，供不同场景使用。

3. **Feature 可配置**: 通过 `updater_default_cfg.gni` 灵活开关功能。

4. **条件编译**: UI 和 Flashd 模块支持条件编译，适应不同设备需求。

5. **完整的测试覆盖**: 提供单元测试和 Fuzz 测试，共 28+ 个测试目标。

## 相关跳转

- [配置标志附录](./appendix/Config_Flags.md)
- [目录结构](./02_Directory_Structure.md)
- [对外 API](./03_Public_API.md)
