# 07_构建与产物

> dmsfwk_lite 项目的 GN 构建配置、编译产物说明及 Feature 开关详解。

## 一、构建配置概览

### 1.1 构建系统

| 属性 | 值 |
|------|-----|
| **构建系统** | GN (Generate Ninja) |
| **构建工具** | hb (OpenHarmony Build) |
| **目标平台** | LiteOS-A / Linux |
| **编译类型** | 共享库 (.so) |

**证据来源**：`BUILD.gn:17-19` 目标平台与类型。

### 1.2 构建命令

```bash
# 全量构建
hb build

# 单独构建 dmsfwk_lite
hb build --gn-targets //foundation/ability/dmsfwk_lite:dtbschedmgr
```

---

## 二、GN Targets 详解

### 2.1 目标清单

| Target 名称 | 类型 | 依赖 | 产物 |
|-------------|------|------|------|
| `dmslite` | shared_library | 内部依赖 | `libdmslite.so` |
| `dtbschedmgr` | lite_component | `:dmslite` | 组件包 |

**证据来源**：`BUILD.gn:18-73` 构建配置。

### 2.2 dmslite Target 详细配置

```gn
# BUILD.gn:18-69
lite_library("dmslite") {
  target_type = "shared_library"

  cflags = [ "-Wall" ]
  cflags_cc = cflags

  defines = [
    "_GNU_SOURCE",
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
  ]

  sources = [
    "source/dmslite.c",
    "source/dmslite_famgr.c",
    "source/dmslite_feature.c",
    "source/dmslite_msg_handler.c",
    "source/dmslite_packet.c",
    "source/dmslite_parser.c",
    "source/dmslite_permission.c",
    "source/dmslite_session.c",
    "source/dmslite_tlv_common.c",
  ]

  include_dirs = [
    "include",
    "interfaces/innerkits",
    "${aafwk_lite_path}/interfaces/inner_api/abilitymgr_lite",
    "${aafwk_lite_path}/interfaces/kits/ability_lite",
    "${aafwk_lite_path}/interfaces/kits/want_lite",
    "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
    "${appexecfwk_lite_path}/interfaces/inner_api/bundlemgr_lite",
    "//commonlibrary/utils_lite/include",
    "//foundation/communication/dsoftbus/interfaces/kits/bus_center",
    "//foundation/communication/dsoftbus/interfaces/kits/common",
    "//foundation/communication/dsoftbus/interfaces/kits/transport",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/innerkits",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
    "//third_party/bounds_checking_function/include",
    "//third_party/cJSON",
  ]

  deps = []

  public_deps = [
    "${aafwk_lite_path}/frameworks/abilitymgr_lite:aafwk_abilityManager_lite",
    "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared",
    "//foundation/communication/dsoftbus/sdk:softbus_client",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
    "//third_party/bounds_checking_function:libsec_shared",
  ]
}
```

---

## 三、依赖关系

### 3.1 系统内部依赖

| 依赖模块 | 用途 | 依赖类型 | 来源 |
|---------|------|---------|------|
| `abilitymgr_lite` | Ability 管理 | public_deps | `BUILD.gn:63` |
| `samgr_lite` | 服务注册 | public_deps | `BUILD.gn:66` |
| `hilog_lite` | 日志输出 | public_deps | `BUILD.gn:64` |
| `dsoftbus` | 通信 | public_deps | `BUILD.gn:65` |

### 3.2 第三方依赖

| 依赖库 | 用途 | 依赖类型 | 来源 |
|-------|------|---------|------|
| `bounds_checking_function` | 安全字符串函数 | public_deps | `BUILD.gn:67` |
| `cJSON` | JSON 解析 | include_dirs | `BUILD.gn:57` |

**证据来源**：`BUILD.gn:60-68` 依赖配置。

### 3.3 组件依赖

| 依赖组件 | 子系统 | 用途 |
|---------|-------|------|
| `ability_lite` | ability | Want、Ability 接口 |
| `bundle_framework_lite` | appexecfwk | Bundle 信息 |
| `samgr_lite` | systemabilitymgr | 系统服务管理 |
| `dsoftbus` | communication | 设备间通信 |

**证据来源**：`bundle.json:22-30` 组件依赖。

---

## 四、编译产物

### 4.1 产物清单

| 产物类型 | 文件名 | 路径 | 说明 |
|---------|-------|------|------|
| **共享库** | `libdmslite.so` | `out/<platform>/libs/` | 主共享库 |
| **符号表** | `libdmslite.so.symbol` | `out/<platform>/symbols/` | 调试符号 |
| **声明文件** | `dmsfwk_interface.h` | `out/<platform>/include/` | 头文件导出 |

### 4.2 安装路径

| 路径 | 说明 |
|------|------|
| `/system/lib/` | 系统库目录 |
| `/system/bin/` | 可执行文件目录（如果有） |
| `/system/etc/` | 配置文件目录（如果有） |

---

## 五、Feature 开关

### 5.1 编译宏

| 宏定义 | 值 | 作用 | 来源 |
|-------|-----|------|------|
| `_GNU_SOURCE` | - | 启用 GNU 扩展 | `BUILD.gn:25` |
| `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` | - | 启用 BMS BundleManager | `BUILD.gn:26` |

### 5.2 条件编译

| 条件 | 宏 | 说明 |
|------|-----|------|
| 可穿戴产品 | `WEARABLE_PRODUCT` | 使用 wearable 产品特定代码 |
| Linux 内核 | `__LINUX__` | Linux 平台特定代码 |
| XTS 测试 | `XTS_SUITE_TEST` | 测试套件特定代码 |

**证据来源**：`source/dmslite_permission.c:23-42` 条件编译。

### 5.3 产品类型区分

```cpp
// source/dmslite_permission.c:23-42
#ifdef WEARABLE_PRODUCT
#include "bundle_manager.h"
#else
#include "bundle_inner_interface.h"
#include "bundle_manager.h"
#endif
```

```cpp
// source/dmslite_utils.c:54-72
#ifdef WEARABLE_PRODUCT
#define DMS_ALLOC(size) OhosMalloc(MEM_TYPE_APPFMK_LSRAM, size)
#define DMS_FREE(a) ...
#else
#define DMS_ALLOC(size) malloc(size)
#define DMS_FREE(a) ...
#endif
```

---

## 六、构建变体

### 6.1 按内核类型

| 内核类型 | 宏定义 | 产物 |
|---------|-------|------|
| LiteOS-A | `ohos_kernel_type == "liteos_a"` | libdmslite.so |
| Linux | `ohos_kernel_type == "linux"` | libdmslite.so |

**证据来源**：`BUILD.gn:17`

```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  lite_library("dmslite") {
    ...
  }
}
```

### 6.2 按产品类型

| 产品类型 | 宏定义 | 差异 |
|---------|-------|------|
| 默认 | 无 | 标准代码路径 |
| 可穿戴 | `WEARABLE_PRODUCT` | 简化内存管理 |

---

## 七、构建产物验证

### 7.1 检查共享库

```bash
# 查看库信息
file out/linux/libs/libdmslite.so

# 查看依赖
ldd out/linux/libs/libdmslite.so

# 查看符号
nm -D out/linux/libs/libdmslite.so
```

### 7.2 预期输出

```
# file 输出
ELF 32-bit LSB shared object, ARM, EABI5

# ldd 预期依赖
libsamgr.so
libhilog.so
libdsoftbus.so
libutils.so
libc.so
```

---

## 八、常见构建问题

### Q1：找不到头文件

**问题**：`fatal error: xxx.h: No such file or directory`

**解决**：
1. 确认 `include_dirs` 包含正确路径
2. 确认依赖组件已构建

### Q2：符号未定义

**问题**：`undefined reference to xxx`

**解决**：
1. 确认 `public_deps` 包含正确依赖
2. 确认依赖库已构建

### Q3：条件编译代码未生效

**问题**：`WEARABLE_PRODUCT` 代码未编译

**解决**：
1. 确认产品类型配置正确
2. 检查 `hb set` 产品配置

---

## 九、相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |
| [03_CodeMap.md](./03_CodeMap.md) | 代码地图 |
| [04_Interface.md](./04_Interface.md) | 对外接口 |

---

*文档版本：v1.0*  
*最后更新：2026-02-07*
