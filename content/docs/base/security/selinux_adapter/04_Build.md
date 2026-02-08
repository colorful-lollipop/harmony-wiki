# 构建系统

## 1. 构建配置

### 1.1 构建入口

| 文件 | 说明 |
|------|------|
| `BUILD.gn` (1613 行) | 主构建文件，包含所有 targets |
| `selinux.gni` | 构建参数定义 |
| `bundle.json` | 组件配置 |

### 1.2 构建参数 (selinux.gni)

**证据来源**: `selinux.gni:18-36`

```gni
declare_args() {
  selinux_adapter_build_path = "default"                      # 策略路径
  selinux_adapter_components = "default"                      # 组件类型 (system/vendor/default)
  selinux_adapter_vendor_policy_version = "40"               # 厂商策略版本
  selinux_adapter_special_build_policy_script = "default"     # 自定义策略编译脚本
  selinux_adapter_special_build_contexts_script = "default"  # 自定义上下文编译脚本
  special_build_ignore_cfg = "default"                       # 忽略配置
  special_selinux_check_config = "default"                    # SELinux 检查配置
  selinux_adapter_extra_args = "default"                      # 策略编译额外参数
  selinux_adapter_contexts_extra_args = "default"             # 上下文编译额外参数
  selinux_adapter_support_developer_mode = false              # 开发者模式
  selinux_adapter_special_build_selinux_gni_path = ""        # 自定义 selinux.gni 路径
  selinux_adapter_check_extend_list = "default"               # 检查扩展列表
  selinux_adapter_seharmony_build_path = "default"           # SEHarmony 策略路径
  selinux_adapter_mcs_enable = true                          # MCS 多级安全
}

declare_args() {
  selinux_adapter_enforce = true                             # 默认强制模式
  selinux_adapter_mcs_enable = true
}
```

### 1.3 组件类型

| 组件类型 | selinux_adapter_components | 用途 |
|----------|----------------------------|------|
| default | "default" | 系统默认（包含 system 和 vendor 策略） |
| system | "system" | 仅系统策略 |
| vendor | "vendor" | 仅厂商策略 |

## 2. Targets 列表

### 2.1 运行时库 (Shared Libraries)

| Target | 类型 | 源码 | 输出 | 安装位置 |
|--------|------|------|------|----------|
| `libload_policy` | ohos_shared_library | `load_policy.cpp` | libload_policy.so | system/ramdisk/updater |
| `librestorecon` | ohos_shared_library | `selinux_restorecon.c` | librestorecon.so | system/ramdisk/updater |
| `libhap_restorecon` | ohos_shared_library | `hap_restorecon.cpp`, `restore_task.cpp`... | libhap_restorecon.so | system |
| `libparaperm_checker` | ohos_shared_library | `param_checker.c` | libparaperm_checker.so | system/updater |
| `libservice_checker` | ohos_shared_library | `service_checker.cpp` | libservice_checker.so | system |

### 2.2 静态库

| Target | 类型 | 源码 | 输出 |
|--------|------|------|------|
| `libselinux_klog_real_static` | ohos_static_library | `selinux_klog.c` | libselinux_klog_real_static.a |
| `libselinux_klog_static` | ohos_static_library | `selinux_klog.c` | libselinux_klog_static.a |
| `librestorecon_static` | ohos_static_library | `selinux_restorecon.c` | librestorecon_static.a |
| `libselinux_error_static` | ohos_static_library | `selinux_error.cpp` | libselinux_error_static.a |
| `libselinux_hilog_static` | ohos_static_library | `selinux_log.c` | libselinux_hilog_static.a |
| `libselinux_parameter_static` | ohos_static_library | `contexts_trie.c`, `selinux_map.c`... | libselinux_parameter_static.a |
| `libselinux_parameter_static_noflto` | ohos_static_library | 同上 | libselinux_parameter_static_noflto.a |

### 2.3 可执行工具

| Target | 类型 | 源码 | 用途 |
|--------|------|------|------|
| `load_policy` | ohos_executable | `framework/tools/load_policy/load_policy.c` | 策略加载工具 |
| `restorecon` | ohos_executable | `framework/tools/restorecon/restorecon.c` | 文件标签恢复工具 |
| `hap_restorecon` | ohos_executable | `framework/tools/hap_restorecon/test.cpp` | HAP 恢复测试工具 |
| `param_check` | ohos_executable | `framework/tools/param_check/test.cpp` | 参数检查测试工具 |
| `service_check` | ohos_executable | `framework/tools/service_check/test.cpp` | 服务检查测试工具 |

### 2.4 策略编译 Action

| Target | 类型 | 脚本 | 输出 |
|--------|------|------|------|
| `build_policy` | action | `scripts/build_policy.py` | policy.31, system.cil, vendor.cil... |
| `build_update_policy` | action | `scripts/build_policy.py` | updater/policy.31 |
| `build_contexts` | action | `scripts/build_contexts.py` | file_contexts, service_contexts... |
| `build_updater_contexts` | action | `scripts/build_contexts.py` | updater/file_contexts... |
| `build_ignore_cfg` | action | `scripts/build_ignore_cfg.py` | ignore_cfg, app_allow_cfg |
| `selinux_check` | action | `scripts/selinux_check/selinux_check_main.py` | 策略合规性检查 |

### 2.5 产物预置 (Prebuilt)

| Target | 类型 | 来源 | 安装路径 |
|--------|------|------|----------|
| `build_sepolicy` | ohos_prebuilt_etc | `out/policy.31` | `/etc/selinux/targeted/policy/` |
| `build_updater_sepolicy` | ohos_prebuilt_etc | `out/updater/policy.31` | `/etc/selinux/targeted/policy/` |
| `config` | ohos_prebuilt_etc | `config/config` | `/etc/selinux/` |
| `file_contexts` | ohos_prebuilt_etc | `out/file_contexts` | `/etc/selinux/targeted/contexts/` |
| `sehap_contexts` | ohos_prebuilt_etc | `out/sehap_contexts` | `/etc/selinux/targeted/contexts/` |
| `parameter_contexts` | ohos_prebuilt_etc | `out/parameter_contexts` | `/etc/selinux/targeted/contexts/` |
| `service_contexts` | ohos_prebuilt_etc | `out/service_contexts` | `/etc/selinux/targeted/contexts/` |
| `hdf_service_contexts` | ohos_prebuilt_etc | `out/hdf_service_contexts` | `/etc/selinux/targeted/contexts/` |
| `ignore_cfg` | ohos_prebuilt_etc | `out/ignore_cfg` | `/etc/selinux/` |
| `app_allow_cfg` | ohos_prebuilt_etc | `out/app_allow_cfg` | `/etc/selinux/` |

### 2.6 CIL 策略文件

| Target | 来源 | 安装路径 |
|--------|------|----------|
| `system_cil` | out/system.cil | /etc/selinux/ |
| `vendor_cil` | out/vendor.cil | /etc/selinux/ |
| `public_cil` | out/public.cil | /etc/selinux/ |
| `version_cil` | out/compatible/*.cil | /etc/selinux/compatible/ |
| `system_common_cil` | out/system_common.cil | /etc/selinux/ |

### 2.7 主 Group

```gn
group("selinux_group") {
  deps = [
    ":app_allow_cfg",
    ":build_updater_sepolicy",
    ":config",
    ":file_contexts",
    ":file_contexts_updater",
    ":hap_restorecon",
    ":hdf_service_contexts",
    ":ignore_cfg",
    ":libload_policy",
    ":librestorecon",
    ":libparaperm_checker",
    ":libservice_checker",
    ":load_policy",
    ":param_check",
    ":parameter_contexts",
    ":restorecon",
    ":sehap_contexts",
    ":service_check",
    ":service_contexts",
    ":updater_config",
    ":selinux_check",  # if !asan_detector
    # ... CIL 文件 targets
  ]
}
```

## 3. 依赖关系

### 3.1 内部依赖

| Target | deps | public_deps | external_deps |
|--------|------|-------------|---------------|
| libload_policy | libselinux_klog_static | - | selinux:libselinux |
| librestorecon | libselinux_klog_static | selinux_core_config | FreeBSD, hilog, selinux |
| libhap_restorecon | libselinux_error_static, libselinux_hilog_static | selinux_core_config | FreeBSD, cJSON, hilog, hisysevent, selinux |
| libparaperm_checker | libselinux_klog_static, libselinux_parameter_static | selinux_core_config | bounds_checking_function, selinux |
| libservice_checker | libselinux_error_static, libselinux_hilog_static | selinux_core_config | bounds_checking_function, hilog, selinux |

### 3.2 外部依赖 (bundle.json)

**证据来源**: `bundle.json:36-46`

```json
"deps": {
  "components": [
    "cJSON",
    "c_utils",
    "FreeBSD",
    "hilog",
    "bounds_checking_function",
    "hisysevent",
    "selinux",
    "pcre2"
  ],
  "third_party": []
}
```

### 3.3 编译时工具依赖

| 工具 | 来源 | 用途 |
|------|------|------|
| checkpolicy | selinux:checkpolicy | 策略语法检查 |
| secilc | selinux:secilc | CIL 策略编译 |
| sefcontext_compile | selinux:sefcontext_compile | 上下文编译 |
| libselinux | selinux:libselinux | SELinux 用户空间库 |

## 4. 编译产物

### 4.1 系统镜像产物

| 产物类型 | 路径 | 说明 |
|----------|------|------|
| 运行时库 | `/system/lib64/libload_policy.z.so` | 策略加载 |
| 运行时库 | `/system/lib64/librestorecon.z.so` | 文件恢复 |
| 运行时库 | `/system/lib64/libhap_restorecon.z.so` | HAP 上下文 |
| 运行时库 | `/system/lib64/libparaperm_checker.z.so` | 参数检查 |
| 运行时库 | `/system/lib64/libservice_checker.z.so` | 服务检查 |
| 二进制策略 | `/etc/selinux/targeted/policy/policy.31` | SELinux 策略 |
| 文件上下文 | `/etc/selinux/targeted/contexts/file_contexts` | 文件标签映射 |
| 参数上下文 | `/etc/selinux/targeted/contexts/parameter_contexts` | 参数标签映射 |
| 服务上下文 | `/etc/selinux/targeted/contexts/service_contexts` | SA 标签映射 |
| HDF 服务上下文 | `/etc/selinux/targeted/contexts/hdf_service_contexts` | HDF 标签映射 |
| HAP 上下文 | `/etc/selinux/targeted/contexts/sehap_contexts` | 应用标签映射 |
| 模式配置 | `/etc/selinux/config` | enforcing/permissive |

### 4.2 Updater 产物

| 产物 | 路径 |
|------|------|
| 策略 | `/updater/etc/selinux/targeted/policy/policy.31` |
| 上下文 | `/updater/etc/selinux/targeted/contexts/file_contexts` |
| 配置 | `/updater/etc/selinux/config` |

### 4.3 工具产物

| 工具 | 路径 | 说明 |
|------|------|------|
| load_policy | `/system/bin/load_policy` | 策略加载命令 |
| restorecon | `/system/bin/restorecon` | 文件恢复命令 |

## 5. 编译命令

### 5.1 完整编译

```bash
./build.sh --product-name=rk3568 -T selinux_adapter --ccache
```

### 5.2 仅编译策略

```bash
# 策略编译
./build.sh --product-name=rk3568 -T selinux_adapter --build-target build_policy --ccache

# 上下文编译
./build.sh --product-name=rk3568 -T selinux_adapter --build-target build_contexts --ccache
```

### 5.3 仅编译库

```bash
# 运行时库
./build.sh --product-name=rk3568 -T selinux_adapter --build-target libload_policy --ccache
./build.sh --product-name=rk3568 -T selinux_adapter --build-target librestorecon --ccache
```

### 5.4 运行测试

```bash
# 运行单元测试
./build.sh --product-name=rk3568 --build-target selinux_adapter_test --ccache

# 测试二进制位置
./out/rk3568/tests/selinux_adapter/selinux_adapter/selinux_adapter_unittest
```

## 6. 编译脚本

### 6.1 build_policy.py

**位置**: `scripts/build_policy.py`

**功能**: 收集并编译 SELinux 策略文件

**流程**:
```
1. 收集 sepolicy/*/*.te 文件
2. 调用 checkpolicy 语法检查
3. 调用 secilc 编译为 CIL
4. 输出 policy.31, *.cil
```

### 6.2 build_contexts.py

**位置**: `scripts/build_contexts.py`

**功能**: 编译 Security Context 文件

**输出**:
- file_contexts (文件路径映射)
- parameter_contexts (参数映射)
- service_contexts (服务映射)
- hdf_service_contexts (HDF 服务映射)
- sehap_contexts (HAP 应用映射)

### 6.3 selinux_check/

**位置**: `scripts/selinux_check/`

**功能**: 策略合规性检查

| 检查项 | 脚本 |
|--------|------|
| 分区标签使用 | check_partition_label_use.py |
| 域检查 | check_domain.py |
| ioctl 权限 | check_ioctl_xperm.py |
| 数据正则 | check_data_regex.py |
| 上下文一致性 | check_consistency_of_tcontext.py |
| Socket 权限 | check_socket.py |
| 基线检查 | check_baseline.py |
| 文件上下文类型 | check_file_contexts_typeattr.py |
| 权限组 | check_perm_group.py |
| 宽容模式 | check_permissive.py |
| 虚拟fs访问 | check_virtfs_access.py |
| 参数检查 | check_parameter.py |

## 7. 构建开关

### 7.1 条件编译

| 条件 | 作用 | 代码位置 |
|------|------|----------|
| `selinux_adapter_support_developer_mode` | 开发者模式支持 | BUILD.gn:62-64 |
| `selinux_adapter_mcs_enable` | MCS 多级安全 | BUILD.gn:170-172 |
| `is_emulator` | 模拟器模式 | BUILD.gn:59-61 |
| `startup_init_with_param_base` | 初始化参数基座 | BUILD.gn:18-21 |

### 7.2 编译选项

```gn
cflags = [
  "-D_GNU_SOURCE",
  "-Wall",
  "-Werror",
]
```

## 8. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概述 | [01_Overview.md](01_Overview.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| API 接口 | [03_API.md](03_API.md) |
| 安全评审 | [05_Security.md](05_Security.md) |
| 故障排查 | [06_Troubleshooting.md](06_Troubleshooting.md) |
