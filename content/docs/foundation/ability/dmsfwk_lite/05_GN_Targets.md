# 05_GN_Targets - 构建系统与编译产物

## 1. 构建入口

### 1.1 主 BUILD.gn

**位置**: `BUILD.gn`

**功能**: 定义 dmsfwk_lite 组件的构建目标。

**关键配置**:
```gn
# 仅适用于 liteos_a 或 linux 内核
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  # 定义共享库目标
  lite_library("dmslite") { ... }
  
  # 定义组件目标
  lite_component("dtbschedmgr") { ... }
  
  # 定义通知文件生成
  generate_notice_file("dtbschedmgr_notice_file") { ... }
}
```

## 2. Targets 列表

### 2.1 dmslite（共享库）

| 属性 | 值 |
|------|-----|
| Target 名 | `dmslite` |
| 类型 | `shared_library` |
| 输出文件 | `libdmslite.so` |
| 条件 | `ohos_kernel_type == "liteos_a" \|\| "linux"` |

**源码列表** (`sources`):

| 文件 | 模块 | 说明 |
|------|------|------|
| `source/dmslite.c` | Service | 服务注册 |
| `source/dmslite_famgr.c` | FA Manager | FA 管理 |
| `source/dmslite_feature.c` | Feature | Feature 实现 |
| `source/dmslite_msg_handler.c` | Msg Handler | 消息处理 |
| `source/dmslite_packet.c` | Packet | 报文打包 |
| `source/dmslite_parser.c` | Parser | TLV 解析 |
| `source/dmslite_permission.c` | Permission | 权限检查 |
| `source/dmslite_session.c` | Session | 会话管理 |
| `source/dmslite_tlv_common.c` | TLV Utils | TLV 工具 |

**编译选项** (`cflags`):
```gn
cflags = [ "-Wall" ]  # 开启所有警告
```

**宏定义** (`defines`):
```gn
defines = [
  "_GNU_SOURCE",
  "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
]
```

**头文件搜索路径** (`include_dirs`):

| 路径 | 说明 |
|------|------|
| `"include"` | 内部头文件 |
| `"interfaces/innerkits"` | 对外接口 |
| `"${aafwk_lite_path}/interfaces/inner_api/abilitymgr_lite"` | AAFwk 内部接口 |
| `"${aafwk_lite_path}/interfaces/kits/ability_lite"` | AAFwk Kit |
| `"${aafwk_lite_path}/interfaces/kits/want_lite"` | Want Kit |
| `"${appexecfwk_lite_path}/interfaces/kits/bundle_lite"` | Bundle Kit |
| `"${appexecfwk_lite_path}/interfaces/inner_api/bundlemgr_lite"` | Bundle 内部接口 |
| `"//commonlibrary/utils_lite/include"` | Utils |
| `"//foundation/communication/dsoftbus/interfaces/kits/bus_center"` | SoftBus 总线中心 |
| `"//foundation/communication/dsoftbus/interfaces/kits/common"` | SoftBus 通用 |
| `"//foundation/communication/dsoftbus/interfaces/kits/transport"` | SoftBus 传输 |
| `"//foundation/systemabilitymgr/samgr_lite/interfaces/innerkits"` | SAMGR 内部 |
| `"//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr"` | SAMGR Kit |
| `"//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry"` | SAMGR 注册 |
| `"//third_party/bounds_checking_function/include"` | 安全函数 |
| `"//third_party/cJSON"` | cJSON |

**公共依赖** (`public_deps`):

| 依赖 | 说明 |
|------|------|
| `${aafwk_lite_path}/frameworks/abilitymgr_lite:aafwk_abilityManager_lite` | Ability 管理器 |
| `//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared` | 日志服务 |
| `//foundation/communication/dsoftbus/sdk:softbus_client` | SoftBus 客户端 |
| `//foundation/systemabilitymgr/samgr_lite/samgr:samgr` | SAMGR |
| `//third_party/bounds_checking_function:libsec_shared` | 安全函数库 |

### 2.2 dtbschedmgr（组件）

| 属性 | 值 |
|------|-----|
| Target 名 | `dtbschedmgr` |
| 类型 | `lite_component` |
| 功能 | 聚合 dmslite 库 |

```gn
lite_component("dtbschedmgr") {
  features = [ ":dmslite" ]
}
```

### 2.3 dtbschedmgr_notice_file（通知文件）

| 属性 | 值 |
|------|-----|
| Target 名 | `dtbschedmgr_notice_file` |
| 类型 | `generate_notice_file` |
| 功能 | 生成第三方库版权声明 |

```gn
generate_notice_file("dtbschedmgr_notice_file") {
  module_name = "dtbschedmgr"
  module_source_dir_list = [
    "//third_party/bounds_checking_function",
    "//third_party/cJSON",
  ]
}
```

## 3. 依赖分析

### 3.1 组件依赖图

```
dtbschedmgr (lite_component)
    │
    ▼
dmslite (shared_library)
    │
    ├──► aafwk_abilityManager_lite ──► ability_lite 框架
    │
    ├──► hilog_shared ───────────────► 日志服务
    │
    ├──► softbus_client ─────────────► DSoftBus 客户端
    │       ├──► 设备发现
    │       ├──► 会话管理
    │       └──► 数据传输
    │
    ├──► samgr ──────────────────────► 系统服务管理
    │
    └──► libsec_shared ──────────────► bounds_checking_function
```

### 3.2 运行时依赖

| 依赖组件 | 用途 | 加载方式 |
|----------|------|----------|
| `samgr_lite` | 服务注册/发现 | 动态链接 |
| `dsoftbus` | 设备组网/传输 | 动态链接 |
| `bundle_framework_lite` | 包信息查询 | IPC/SAMGR |
| `ability_lite` | Want/Element 数据结构 | 头文件包含 |
| `hilog_lite` | 日志输出 | 动态链接 |

## 4. 编译产物

### 4.1 产物清单

| 产物 | 类型 | 路径（推测） | 说明 |
|------|------|--------------|------|
| `libdmslite.so` | 共享库 | `out/{target}/libs/` | 主库文件 |
| `dtbschedmgr` | 组件标记 | - | 构建系统标记 |
| `dtbschedmgr_notice_file` | 文本 | `out/{target}/notice_files/` | 版权声明 |

### 4.2 产物安装路径

基于 OpenHarmony 轻量级系统惯例：

```
/system/lib/libdmslite.so          # 共享库
/system/bin/                       # 无可执行文件（纯库）
/system/native_appid/              # Native 服务 AppID 文件
```

### 4.3 运行时加载关系

```
进程启动
    │
    ▼
/system/lib/libdmslite.so
    │
    ├──► /system/lib/libsamgr.so        (SAMGR)
    ├──► /system/lib/libsoftbus_client.so (SoftBus)
    ├──► /system/lib/libhilog.so        (HiLog)
    └──► /system/lib/libsec_shared.so   (安全函数)
```

## 5. bundle.json 配置

### 5.1 组件元数据

**位置**: `bundle.json`

```json
{
  "name": "@ohos/dmsfwk_lite",
  "description": "distributed abiltiy manager service",
  "version": "3.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "foundation/ability/dmsfwk_lite"
  }
}
```

### 5.2 组件配置

```json
{
  "component": {
    "name": "dmsfwk_lite",
    "subsystem": "ability",
    "adapted_system_type": ["small"]
  }
}
```

**适用系统类型**: `small`（轻量级系统）

### 5.3 依赖声明

**系统组件依赖**:
```json
"deps": {
  "components": [
    "utils_lite",
    "hilog_lite",
    "samgr_lite",
    "bundle_framework_lite",
    "ability_lite",
    "huks"
  ]
}
```

**第三方库依赖**:
```json
"third_party": [
  "bounds_checking_function",
  "cJSON"
]
```

### 5.4 构建配置

**子组件**:
```json
"build": {
  "sub_component": [
    "//foundation/ability/dmsfwk_lite:dtbschedmgr",
    "//foundation/ability/dmsfwk_lite/moduletest/dtbschedmgr_lite:distributed_schedule_test_dms_door"
  ]
}
```

**Inner Kits（对外接口）**:
```json
"inner_kits": [
  {
    "header": {
      "header_base": "foundation/ability/dmsfwk_lite/interfaces/innerkits/",
      "header_files": ["dmsfwk_interface.h"]
    },
    "name": "//foundation/ability/dmsfwk_lite:dtbschedmgr"
  }
]
```

## 6. 构建命令

### 6.1 完整构建

```bash
# 进入 OpenHarmony 源码根目录
cd ~/openharmony

# 设置构建目标
hb set
# 选择对应产品（如：hi3516dv300、rk3568 等）

# 构建 dmsfwk_lite
hb build --target //foundation/ability/dmsfwk_lite:dtbschedmgr

# 或构建全部
hb build
```

### 6.2 构建参数

| 参数 | 说明 |
|------|------|
| `ohos_kernel_type` | 内核类型（liteos_a/linux） |
| `WEARABLE_PRODUCT` | 可穿戴产品宏（影响权限检查） |
| `XTS_SUITE_TEST` | XTS 测试模式（跳过部分发送逻辑） |

### 6.3 条件编译

```c
// 可穿戴产品特殊处理
#ifdef WEARABLE_PRODUCT
    // 简化权限检查
#else
    // 完整权限检查（foundation/shell UID 检查）
#endif

// XTS 测试模式
#ifndef XTS_SUITE_TEST
    // 实际发送消息
    SendDmsMessage(...)
#else
    // 测试模式：跳过发送
    return DMS_EC_SUCCESS;
#endif
```

## 7. 构建开关

### 7.1 当前支持的开关

| 开关 | 位置 | 功能 | 默认 |
|------|------|------|------|
| `ohos_kernel_type` | BUILD.gn | 内核类型过滤 | - |
| `WEARABLE_PRODUCT` | 代码宏 | 可穿戴产品模式 | 未定义 |
| `XTS_SUITE_TEST` | 代码宏 | XTS 测试模式 | 未定义 |
| `__LINUX__` | 代码宏 | Linux 平台适配 | 自动检测 |

### 7.2 添加自定义开关

如需添加新的功能开关，建议：

1. 在 `BUILD.gn` 中添加 `defines`:
```gn
defines = [
  "CUSTOM_FEATURE",
]
```

2. 在代码中使用:
```c
#ifdef CUSTOM_FEATURE
    // 自定义功能
#endif
```

## 8. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [01_Directory_Structure](01_Directory_Structure.md) - 目录结构
- [04_Internal_API](04_Internal_API.md) - 内部接口
