# 07_Build - 构建与产物

## 构建系统概述

**构建工具**: GN (Generate Ninja) + Ninja  
**配置入口**: `dmsfwk.gni`  
**构建文件**: 49 个 BUILD.gn 文件

---

## GN 目标清单

### 1. 服务层产物

| Target | 类型 | 输出文件 | 安装路径 |
|-------|------|---------|---------|
| `distributedschedsvr` | shared_library | `libdistributedschedsvr.z.so` | `system/lib/platformsdk/` |
| `distributed_ability_manager_svr` | shared_library | `libdistributed_ability_manager_svr.z.so` | `system/lib/` |
| `distributed_sched_utils` | shared_library | `libdistributed_sched_utils.z.so` | `system/lib/platformsdk/` |
| `dtbcollab_channel_manager` | shared_library | `libdtbcollab_channel_manager.z.so` | `system/lib/platformsdk/` |
| `distributed_ability_connection_manager` | shared_library | `libdistributed_ability_connection_manager.z.so` | `system/lib/platformsdk/` |
| `dtbcollab_av_stream_trans_provider` | shared_library | `libdtbcollab_av_stream_trans_provider.z.so` | `system/lib/platformsdk/` |

**核心 BUILD.gn**: `services/dtbschedmgr/BUILD.gn`

```gn
ohos_shared_library("distributedschedsvr") {
  sources = [ ... ]
  include_dirs = [ ... ]
  deps = [
    "//foundation/ability/dmsfwk/common:distributed_sched_utils",
    "//foundation/communication/dsoftbus:softbus_client",
    ...
  ]
  external_deps = [
    "ability_base:want",
    "ability_runtime:ability_manager",
    "hilog:libhilog",
    ...
  ]
}
```

### 2. N-API 产物

| Target | 类型 | 安装路径 |
|-------|------|---------|
| `continuationmanager_napi` | shared_library | `system/lib/module/continuation/` |
| `continuemanager_napi` | shared_library | `system/lib/module/app/ability/` |
| `abilityconnectionmanager_napi` | shared_library | `system/lib/module/distributedsched/` |

**BUILD.gn**: `interfaces/kits/napi/continuation_manager/BUILD.gn`

```gn
ohos_shared_library("continuationmanager_napi") {
  include_dirs = [ ... ]
  sources = [ ... ]
  deps = [
    "//foundation/ability/dmsfwk/services/dtbabilitymgr:distributed_ability_manager_svr",
  ]
  relative_install_dir = "module/continuation"
}
```

### 3. SDK 产物

| Target | 类型 | 输出 |
|-------|------|------|
| `common_sdk` | shared_library | `libcommon_sdk.z.so` |
| `continuation_manager` | shared_library | `libcontinuation_manager.z.so` |
| `distributed_sdk` | shared_library | `libdistributed_sdk.z.so` |

### 4. 框架产物

| Target | 类型 | 说明 |
|-------|------|------|
| `distributedextensionability` | shared_library | JS 扩展能力 |
| `distributedextensioncontext` | shared_library | 扩展上下文 |
| `distributed_extension_ability_native` | shared_library | Native 扩展 |

---

## Feature Flags

### dmsfwk.gni 配置

```gni
# 功能开关
dmsfwk_standard_form_share = true
dmsfwk_mission_manager = false
dmsfwk_check_wifi = true
dmsfwk_all_connect = false
dmsfwk_feature_dams_enable = true
dmsfwk_softbus_adapter_common = true

# 安全检查开关
dmsfwk_check_bt = false
dmsfwk_check_screenlock = false
```

### 条件编译宏

| 宏 | 条件 | 说明 |
|-----|------|------|
| `SUPPORT_DISTRIBUTED_MISSION_MANAGER` | `dmsfwk_mission_manager` | 分布式任务管理 |
| `DMS_CHECK_WIFI` | `dmsfwk_check_wifi` | WiFi 检查 |
| `DMS_CHECK_BLUETOOTH` | `dmsfwk_check_bt` | 蓝牙检查 |
| `COLLAB_ALL_CONNECT_DECISIONS` | `dmsfwk_all_connect` | 全连接决策 |

---

## 安全编译选项

### Sanitizers

所有生产库启用：

```gn
sanitize = {
  boundary_sanitize = true    # 边界检查
  cfi = true                  # 控制流完整性
  cfi_cross_dso = true        # 跨 DSO CFI
  integer_overflow = true     # 整数溢出检查
  ubsan = true                # 未定义行为检查
}
```

### 编译标志

```gn
cflags = [ "-fpie" ]           # 位置无关代码
cflags_cc = [ "-Os" ]          # 大小优化
ldflags = [
  "-Wl,-z,relro",             # 重定位只读
  "-Wl,-z,now"                # 立即绑定
]
```

### ARM 保护

```gn
branch_protector_ret = "pac_ret"  # 指针认证
```

---

## 构建命令

### 完整构建

```bash
gn gen out --args='target_os="ohos" target_cpu="arm64"'
ninja -C out foundation/ability/dmsfwk:fwk_group
ninja -C out foundation/ability/dmsfwk:service_group
```

### 单独构建服务

```bash
ninja -C out foundation/ability/dmsfwk/services/dtbschedmgr:distributedschedsvr
ninja -C out foundation/ability/dmsfwk/services/dtbabilitymgr:distributed_ability_manager_svr
```

### 构建测试

```bash
ninja -C out foundation/ability/dmsfwk:test
ninja -C out foundation/ability/dmsfwk/services/dtbschedmgr:unittest
```

---

## 产物安装路径

### 系统库

```
system/lib/platformsdk/
├── libdistributedschedsvr.z.so
├── libdistributed_sched_utils.z.so
├── libdtbcollab_channel_manager.z.so
└── ...

system/lib/
├── libdistributed_ability_manager_svr.z.so
└── ...

system/lib/module/continuation/
└── libcontinuationmanager_napi.z.so

system/lib/module/distributedsched/
└── libabilityconnectionmanager_napi.z.so

system/lib/module/app/ability/
└── libcontinuemanager_napi.z.so
```

### 配置文件

```
system/etc/init/
└── distributedsched.cfg

system/profile/
├── distributedsched_trust.json
├── 1401.json
└── 1404.json
```

---

## 依赖关系

### 核心依赖

```
dtbschedsvr
├── dsoftbus:softbus_client
├── ability_runtime:ability_manager
├── access_token:libaccesstoken_sdk
├── device_auth:deviceauth_sdk
├── ipc:ipc_core
└── samgr:samgr_proxy

distributed_ability_manager_svr
├── dtbschedsvr (部分接口)
├── ability_runtime:ability_manager
└── samgr:samgr_proxy
```

### 完整依赖树

根据 `bundle.json`，共依赖 53+ 组件：

- **核心**: ability_runtime, dsoftbus, ipc, samgr
- **安全**: access_token, device_auth, device_security_level
- **数据**: kv_store, distributed_bundle_framework
- **网络**: wifi, bluetooth
- **多媒体**: av_codec, media_foundation, video_processing_engine
- **其他**: hilog, hisysevent, hitrace, cJSON, openssl

---

## 相关链接

- 上一章: [06_SecurityReview.md](06_SecurityReview.md) - 安全风险评估
- 下一章: [08_Internals.md](08_Internals.md) - 内部实现
