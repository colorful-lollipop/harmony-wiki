# 构建系统与编译产物

## 6.1 GN 构建文件结构

```
/base/telephony/telephony_data/
├── BUILD.gn                    # 根构建入口
├── bundle.json                # 组件元数据
├── signature/
│   └── pm.gni                 # HAP 签名配置
├── etc/
│   └── BUILD.gn               # 配置文件构建
└── test/
    ├── BUILD.gn               # 测试构建入口
    └── unittest/
        ├── data_test/BUILD.gn # 单元测试
        └── data_gtest/BUILD.gn # GTest 测试
```

---

## 6.2 主要 Targets

### 6.2.1 根 BUILD.gn Targets

**文件**: `/Volumes/lexar/code/d/work/oh/base/telephony/telephony_data/BUILD.gn`

| Target 名称 | 类型 | 输出 | 描述 |
|-------------|------|------|------|
| `Telephony_Data_Storage` | ohos_hap | `.hap` | 主 HAP 包 |
| `tel_telephony_data` | ohos_shared_library | `.so` | 核心共享库 |
| `tel_telephony_data_headers` | ohos_shared_headers | `.h` | 公共头文件 |

### 6.2.2 HAP Target 详情

```gn
ohos_hap("Telephony_Data_Storage") {
    hap_name = "TelephonyDataStorage"
    subsystem_name = "telephony"
    part_name = "telephony_data"
    
    sources = [
        "src/**/*.cpp",
        "interfaces/**/*.cpp",
    ]
    
    include_dirs = [
        "interfaces/innerkits/include",
        "common/include",
        "sim/include",
        "sms_mms/include",
        "pdp_profile/include",
        "opkey/include",
        "global_params/include",
    ]
    
    deps = [
        "//foundation/ability/ability_runtime:ability_runtime",
        "//foundation/data_share/relational_store:rdbs",
        "//foundation/security/access_token:access_token",
        "//base/hiviewdfx/hilog_native:hilog",
        "//base/global/i18n_native: i18n",
        "//third_party/sqlite:sqlite",
    ]
    
    cflags = [
        "-Wall",
        "-Wextra",
        "-Werror",
    ]
    
    # 安全编译选项
    if (use_cfi) {
        cflags += [ "-fsanitize=cfi" ]
    }
    
    # 堆保护
    if (enable_heap_profiler) {
        cflags += [ "-fno-omit-frame-pointer" ]
    }
}
```

### 6.2.3 共享库 Target 详情

```gn
ohos_shared_library("tel_telephony_data") {
    sources = [
        "common/src/**/*.cpp",
        "sim/src/**/*.cpp",
        "sms_mms/src/**/*.cpp",
        "pdp_profile/src/**/*.cpp",
        "opkey/src/**/*.cpp",
        "global_params/src/**/*.cpp",
    ]
    
    include_dirs = [
        "interfaces/innerkits/include",
        "common/include",
        "sim/include",
        "sms_mms/include",
        "pdp_profile/include",
        "opkey/include",
        "global_params/include",
    ]
    
    deps = [
        "//foundation/ability/ability_runtime:ability_runtime",
        "//foundation/data_share/relational_store:rdbs",
        "//foundation/security/access_token:access_token",
        "//base/hiviewdfx/hilog_native:hilog",
    ]
    
    # 链接选项
    ldflags = [
        "-Wl,-z,relro",
        "-Wl,-z,now",
        "-fstack-protector-strong",
    ]
}
```

---

## 6.3 配置文件构建 (etc/BUILD.gn)

**文件**: `/Volumes/lexar/code/d/work/oh/base/telephony/telephony_data/etc/BUILD.gn`

| Target | 类型 | 源文件 | 安装路径 |
|--------|------|--------|----------|
| `pdp_profile_default` | ohos_prebuilt_etc | `pdp_profile.json` | `/system/etc/telephony/` |
| `ecc_data_default` | ohos_prebuilt_etc | `ecc_data.json` | `/system/etc/telephony/` |
| `num_match_default` | ohos_prebuilt_etc | `number_match.json` | `/system/etc/telephony/` |
| `opkey_info_default` | ohos_prebuilt_etc | `OpkeyInfo.json` | `/system/etc/telephony/` |

```gn
ohos_prebuilt_etc("pdp_profile_default") {
    source = "pdp_profile.json"
    relative_install_dir = "telephony"
}

ohos_prebuilt_etc("ecc_data_default") {
    source = "ecc_data.json"
    relative_install_dir = "telephony"
}

ohos_prebuilt_etc("num_match_default") {
    source = "number_match.json"
    relative_install_dir = "telephony"
}

ohos_prebuilt_etc("opkey_info_default") {
    source = "OpkeyInfo.json"
    relative_install_dir = "telephony"
}
```

---

## 6.4 组件元数据 (bundle.json)

**文件**: `/Volumes/lexar/code/d/work/oh/base/telephony/telephony_data/bundle.json`

```json
{
    "name": "telephony_data",
    "subsystem": "telephony",
    "version": "1.0.0",
    "description": "Telephony data storage service",
    "components": [
        {
            "component": {
                "name": "telephony_data",
                "description": "Telephony Data Storage HAP",
                "label": "Telephony Data Storage",
                "type": "service",
                "subsystem": "telephony",
                "hap_path": "./",
                "build_target": [
                    "Telephony_Data_Storage"
                ],
                "inner_kits": [
                    {
                        "name": "tel_telephony_data_headers",
                        "header_path": "interfaces/innerkits/include",
                        "description": "Telephony data inner API headers"
                    }
                ],
                "deps": {
                    "components": [
                        "ability_runtime",
                        "data_share",
                        "access_token",
                        "hilog"
                    ]
                }
            }
        }
    ]
}
```

---

## 6.5 编译产物清单

### 6.5.1 主要产物

| 产物类型 | 文件名 | 路径 | 说明 |
|----------|--------|------|------|
| **HAP 包** | `Telephony_Data_Storage.hap` | `out/` | 可安装的 Ability 包 |
| **共享库** | `libtel_telephony_data.z.so` | `out/` | 核心数据存储库 |
| **头文件** | `tel_telephony_data_headers` | `out/` | 对外 API 头文件目录 |
| **测试可执行文件** | `tel_telephony_data_test` | `out/` | 单元测试 |
| **GTest 库** | `libtel_telephony_data_gtest.z.so` | `out/` | GTest 测试库 |

### 6.5.2 安装产物

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| HAP | `/app/com.ohos.telephonydataability/` | telephony data ability 包 |
| 共享库 | `/system/lib/` | 系统共享库 |
| 配置文件 | `/system/etc/telephony/` | APN、紧急号码等配置 |
| 签名 | `/system/etc/telephony/signature/` | HAP 签名文件 |

### 6.5.3 运行时加载关系

```
应用启动
    │
    ▼
DataShareHelper::Creator()
    │
    ▼
加载 libtel_telephony_data.z.so
    │
    ├──► 初始化 RDB 数据库
    ├──► 注册 DataAbility
    └──► 等待 IPC 请求

系统启动
    │
    ▼
SystemAbilityManager
    │
    └──► 启动 Telephony Data Storage SA
            │
            └──► 加载配置文件
            └──► 初始化 RDB
            └──► 注册 URI 路由
```

---

## 6.6 构建配置选项

### 6.6.1 GN 变量

| 变量 | 默认值 | 描述 |
|------|--------|------|
| `use_cfi` | false | 启用 CFI (Control Flow Integrity) |
| `enable_heap_profiler` | false | 启用堆内存分析 |
| `enable_allocator` | system | 内存分配器选择 |
| `debuggable` | false | 调试模式 |

### 6.6.2 编译选项

| 选项 | 值 | 用途 |
|------|-----|------|
| `-Wall` | 启用 | 警告信息 |
| `-Wextra` | 启用 | 额外警告 |
| `-Werror` | 启用 | 警告转错误 |
| `-fstack-protector-strong` | 启用 | 堆栈保护 |
| `-fsanitize=cfi` | 可选 | CFI 防护 |

---

## 6.7 构建命令

### 6.7.1 使用 hb 构建

```bash
# 设置构建环境
source build/build.sh

# 构建 telephony_data
hb build telephony_data

# 清理构建
hb build -c clean telephony_data
```

### 6.7.2 使用 gn/ninja

```bash
# 生成构建文件
gn gen out/{device_name}

# 构建 telephony_data
ninja -C out/{device_name} Telephony_Data_Storage

# 只构建共享库
ninja -C out/{device_name} tel_telephony_data

# 只构建测试
ninja -C out/{device_name} tel_telephony_data_gtest
```

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure](02_Directory_Structure.md) |
| 架构说明 | [03_Architecture](03_Architecture.md) |
| 配置标志 | [appendix/Config_Flags](appendix/Config_Flags.md) |

---

*最后更新: 2024-02-06*
