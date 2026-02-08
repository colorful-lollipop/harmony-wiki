# 配置 Flags

---

## 目的

本文档列出 DBMS 服务的关键配置宏和 feature flags，帮助开发者定制编译选项。

---

## 适用范围

- ✅ dbms.gni 中的所有 Feature Flags
- ✅ 条件编译宏
- ✅ 路径配置变量
- ❌ 动态配置（如运行时配置）

---

## Feature Flags

### dbms.gni Feature Flags

**文件**: `/Volumes/lexar/code/d/work/oh/foundation/bundlemanager/distributed_bundle_framework/dbms.gni`

| Flag | 默认值 | 类型 | 说明 | 影响范围 |
|------|--------|------|------|----------|
| `distributed_bundle_framework_graphics` | true | bool | 图形功能支持 | JS N-API 模块编译 |
| `ability_runtime_enable_dbms` | true | bool | Ability 运行时集成 | 账号助手编译 |
| `account_enable_dbms` | true | bool | 账号系统集成 | 外部依赖链接 |
| `distributed_bundle_framework_enable` | true | bool | 分布式 Bundle 框架总开关 | JS N-API 和 ANI 编译 |
| `hisysevent_enable_dbms` | true | bool | HiSysEvent 事件上报 | event_report.cpp 编译 |
| `distributed_bundle_image_framework_enable` | true | bool | 图片框架支持 | image_compress.cpp 编译 |

**证据**: `dbms.gni:28-59`

---

## 条件编译宏

### HISYSEVENT_ENABLE

**定义**: 当 `hisysevent_enable_dbms = true` 时定义

**影响范围**:
- `services/dbms/src/event_report.cpp` 编译
- `services/dbms/src/distributed_bms.cpp` 中的事件报告代码
- `services/dbms/BUILD.gn:84-88` 添加依赖

**代码示例**:
```cpp
// services/dbms/src/distributed_bms.cpp
#ifdef HISYSEVENT_ENABLE
    DBMSEventInfo GetEventInfo(...) { ... }
#endif

#ifdef HISYSEVENT_ENABLE
    int timerId = HiviewDFX::XCollie::GetInstance().SetTimer(...);
#endif
```

**证据**: `services/dbms/BUILD.gn:84-88`, `services/dbms/src/distributed_bms.cpp:70-98`

### ACCOUNT_ENABLE

**定义**: 当 `account_enable_dbms = true` 时定义

**影响范围**:
- 账号助手编译
- 外部依赖链接：`os_account:libaccountkits`, `os_account:os_account_innerkits`
- ACL 检查代码启用

**代码示例**:
```cpp
// services/dbms/src/dbms_device_manager.cpp
#ifdef ACCOUNT_ENABLE
    bool DbmsDeviceManager::CheckAclData(DistributedBmsAclInfo info) { ... }
#else
    APP_LOGI("ACCOUNT_ENABLE is false");
    return false;
#endif
```

**证据**: `services/dbms/BUILD.gn:90-94`, `services/dbms/src/dbms_device_manager.cpp:104-132`

### DISTRIBUTED_BUNDLE_IMAGE_ENABLE

**定义**: 当 `distributed_bundle_image_framework_enable = true` 时定义

**影响范围**:
- 图片压缩模块编译
- 外部依赖链接：`image_framework:image_native`
- 图标处理功能启用

**代码示例**:
```cpp
// services/dbms/src/distributed_bms.cpp
#ifdef DISTRIBUTED_BUNDLE_IMAGE_ENABLE
#include "image_packer.h"
#include "image_source.h"
#endif

bool DistributedBms::GetAbilityIconByContent(...) {
#ifdef DISTRIBUTED_BUNDLE_IMAGE_ENABLE
    // 图片处理逻辑
#endif
}
```

**证据**: `services/dbms/BUILD.gn:96-100`, `services/dbms/src/distributed_bms.cpp:35-38`

---

## 路径配置变量

### dbms.gni 路径定义

```gn
bundlemanager_path = "//foundation/bundlemanager"
bundle_framework_path = "${bundlemanager_path}/bundle_framework"
fuzz_test_path = "distributed_bundle_framework/distributed_bundle_framework"
common_path = "${bundle_framework_path}/common"
dbms_inner_api_path = "${bundlemanager_path}/distributed_bundle_framework/interfaces/inner_api"
dbms_services_path = "${bundlemanager_path}/distributed_bundle_framework/services/dbms"
distributeddatamgr_path = "//foundation/distributeddatamgr"
kits_path = "${bundle_framework_path}/interfaces/kits"
dbms_kits_path = "${bundlemanager_path}/distributed_bundle_framework/interfaces/kits"
```

**用途**: 在多个 BUILD.gn 文件中引用，避免硬编码路径

**证据**: `dbms.gni:14-25`

### 使用示例

```gn
# services/dbms/BUILD.gn
import("../../dbms.gni")

ohos_shared_library("libdbms") {
  deps = [
    "${dbms_inner_api_path}:dbms_fwk"  # 使用变量
  ]

  external_deps = [
    "ability_base:want",
    "access_token:libaccesstoken_sdk",
    # ... 其他依赖
  ]
}
```

**证据**: `services/dbms/BUILD.gn:14-15`, `interfaces/kits/js/distributedBundle/BUILD.gn:15`

---

## 日志配置

### 日志域和标签

**定义位置**: `services/dbms/BUILD.gn:54-57`

```cpp
defines = [
    "APP_LOG_TAG = \"DistributedBundleMgrService\"",
    "LOG_DOMAIN = 0xD0011E0",
]
```

**日志配置**:
- **日志标签**: DistributedBundleMgrService
- **日志域**: 0xD0011E0
- **日志级别**: 通过 APP_LOGI/APP_LOGD/APP_LOGE/APP_LOGW 控制

**证据**: `services/dbms/BUILD.gn:54-57`

### 日志查看命令

```bash
# 查看 DBMS 服务日志
hilog -T DistributedBundleMgrService -v

# 只看错误日志
hilog -T DistributedBundleMgrService -E

# 保存到文件
hilog -T DistributedBundleMgrService > dbms_log.txt

# 实时监控
hilog -T DistributedBundleMgrService -t
```

**证据**: 全源文件中的 `APP_LOGI` 等宏调用

---

## SA 配置常量

### SA ID

```cpp
DISTRIBUTED_BUNDLE_MGR_SERVICE_SYS_ABILITY_ID = 402
```

**用途**: 系统 Ability ID，用于 SA 注册和查询

**使用位置**:
- SA 注册: `services/dbms/src/distributed_bms.cpp:115`
- SA 查询: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:79`

**证据**: `services/dbms/src/distributed_bms.cpp:115`, `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:79`

### 服务名称

```cpp
DISTRIBUTED_BUNDLE_NAME = "distributed_bundle_framework"
SERVICES_NAME = "d-bms"
```

**用途**: 设备管理和 SA 注册使用

**证据**: `services/dbms/src/dbms_device_manager.cpp:29-30`

---

## 编译选项

### 安全编译选项

**统一安全配置**（所有共享库）:

**证据**: `services/dbms/BUILD.gn:28-37`

```gn
branch_protector_ret = "pac_ret"

sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
    integer_overflow = true
    ubsan = true
}

cflags = [ "-fstack-protector-strong" ]

cflags_cc = [
    "-Os",                    # 优化代码大小
    "-fno-unwind-tables",    # 移除调试表
    "-fstack-protector-strong"   # 栈保护
]
```

**作用**:
- **PAC-RET**: 返回地址保护
- **CFI**: 控制流完整性
- **边界检查**: 数组边界验证
- **整型溢出**: 整数运算检查
- **UBSan**: 未定义行为检查
- **栈保护**: 栈溢出保护

---

## GN 构建命令

### 常用构建命令

```bash
# 生成构建文件
gn gen --root=/path/to/ohos --args="distributed_bundle_framework_graphics=true"

# 构建目标
ninja -C out/default libdbms.z.so

# 构建所有
ninja -C out/default dbms_target

# 清理
ninja -C out/default -t clean

# 分析依赖
gn analyze out/default/libdbms.z.so --graph
```

### 变量控制

```bash
# 启用特定功能
gn gen --args="hisysevent_enable_dbms=true account_enable_dbms=true"

# 禁用特定功能
gn gen --args="distributed_bundle_image_framework_enable=false"

# 调试构建
gn gen --args="is_debug_system=true hilog_enable=true"
```

**证据**: `dbms.gni:28-59`

---

## 证据索引

| 配置项 | 证据来源 |
|---------|----------|
| Feature Flags | dbms.gni:28-59 |
| 路径变量 | dbms.gni:14-25 |
| 日志配置 | services/dbms/BUILD.gn:54-57 |
| 安全选项 | services/dbms/BUILD.gn:28-37 |
| SA ID | services/dbms/src/distributed_bms.cpp:115 |
