# GN 目标梳理

## 文档目的

本文档详细说明 distributeddatamgr_cangjie_wrapper 项目的 GN 构建系统，包括所有 targets、依赖关系和输出产物。

## 适用范围

本文档覆盖：
- 所有 BUILD.gn 文件
- Targets 定义和依赖
- 编译产物和安装路径

## 构建系统概览

### GN 构建模板

项目使用 `ohos_cangjie_shared_library` 模板，该模板定义在：
```
//build/templates/cangjie/cjc.gni
```

**特点**:
- 生成仓颉共享库（.so 文件）
- 支持 cj_deps（Cangjie 内部依赖）
- 支持 cj_external_deps（外部 Cangjie 依赖）
- 支持 external_deps（外部 C++ 依赖）

**证据**: 根 BUILD.gn:14 - `import("//build/templates/cangjie/cjc.gni")`

## Target 列表

### 根 Target

#### copy_sdk_distributeddatamgr_cangjie_libs

**路径**: `BUILD.gn:25-28`

| 属性 | 值 |
|--------|-----|
| Target 名称 | copy_sdk_distributeddatamgr_cangjie_libs |
| Target 类型 | copy_ohos_cangjie_sdk_api_lib |
| 功能 | 将 ohos 和 kit 包复制到 SDK 目录 |
| 依赖 | 无 |

**复制的包**:
```gn
ohos_inputs = [
    "//.../ohos/data:ohos.data",
    "//.../ohos/data/preferences:ohos.data.preferences",
    "//.../ohos/data/data_share_predicates:ohos.data.data_share_predicates",
    "//.../ohos/data/relational_store:ohos.data.relational_store",
    "//.../ohos/data/values_bucket:ohos.data.values_bucket",
    "//.../ohos/data/distributed_kv_store:ohos.data.distributed_kv_store",
]

kit_inputs = [
    "//.../kit/ArkData:kit.ArkData"
]
```

**证据**: BUILD.gn:16-23 - distributeddatamgr_cangjie_wrapper_packages_ohos 和 kit 定义

### 模块 Targets

#### 1. kit.ArkData

**路径**: `kit/ArkData/BUILD.gn:19-33`

| 属性 | 值 |
|--------|-----|
| Target 名称 | kit.ArkData |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | index.cj |
| 输出产物 | libkit.ArkData.so |

**cj_deps**:
```gn
cj_deps = [
    "../../ohos/data:ohos.data",
    "../../ohos/data/data_share_predicates:ohos.data.data_share_predicates",
    "../../ohos/data/distributed_kv_store:ohos.data.distributed_kv_store",
    "../../ohos/data/preferences:ohos.data.preferences",
    "../../ohos/data/relational_store:ohos.data.relational_store",
    "../../ohos/data/values_bucket:ohos.data.values_bucket",
]
```

**元数据**:
- subsystem_name: distributeddatamgr
- part_name: distributeddatamgr_cangjie_wrapper

**证据**: kit/ArkData/BUILD.gn:19-33

#### 2. ohos.data

**路径**: `ohos/data/BUILD.gn:19-24`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | data.cj |
| 输出产物 | libohos.data.so |

**依赖**: 无

**证据**: ohos/data/BUILD.gn:19-24

#### 3. ohos.data.preferences

**路径**: `ohos/data/preferences/BUILD.gn`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data.preferences |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | preferences_options.cj, preferences.cj, preferences_ffi.cj |
| 输出产物 | libohos.data.preferences.so |

**external_deps**:
```gn
external_deps = [
    "preferences:cj_preferences_ffi",
]
```

**cj_external_deps**:
```gn
cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
```

**证据**: ohos/data/preferences/BUILD.gn:28-56

#### 4. ohos.data.values_bucket

**路径**: `ohos/data/values_bucket/BUILD.gn:18-35`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data.values_bucket |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | value_type.cj (Windows/Mac: mock/ohos.data.values_bucket.cj) |
| 输出产物 | libohos.data.values_bucket.so |

**条件编译**:
```gn
sources = [ "value_type.cj ]

if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.data.values_bucket.cj" ]
}
```

**cj_external_deps**:
```gn
cj_external_deps = [
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
]
```

**证据**: ohos/data/values_bucket/BUILD.gn:18-35

#### 5. ohos.data.data_share_predicates

**路径**: `ohos/data/data_share_predicates/BUILD.gn`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data.data_share_predicates |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | data_share_predicates.cj, data_share_predicates_common.cj, data_share_predicates_ffi.cj |
| 输出产物 | libohos.data.data_share_predicates.so |

**external_deps**:
```gn
external_deps = [
    "data_share:cj_data_share_predicates_ffi",
]
```

**cj_deps**:
```gn
cj_deps = [
    "../values_bucket:ohos.data.values_bucket",
]
```

**cj_external_deps**:
```gn
cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
```

**条件编译**: Windows/Mac 使用 mock

**证据**: ohos/data/data_share_predicates/BUILD.gn:19-42

#### 6. ohos.data.distributed_kv_store

**路径**: `ohos/data/distributed_kv_store/BUILD.gn`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data.distributed_kv_store |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | 7 个 .cj 文件 |
| 输出产物 | libohos.data.distributed_kv_store.so |

**external_deps**:
```gn
external_deps = [
    "kv_store:cj_distributed_kv_store_ffi",
]
```

**cj_deps**:
```gn
cj_deps = [
    "../data_share_predicates:ohos.data.data_share_predicates",
]
```

**cj_external_deps**:
```gn
cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability",
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
```

**条件编译**: Windows/Mac 使用 mock

**证据**: ohos/data/distributed_kv_store/BUILD.gn:19-57

#### 7. ohos.data.relational_store

**路径**: `ohos/data/relational_store/BUILD.gn`

| 属性 | 值 |
|--------|-----|
| Target 名称 | ohos.data.relational_store |
| Target 类型 | ohos_cangjie_shared_library |
| 源文件 | 6 个 .cj 文件 |
| 输出产物 | libohos.data.relational_store.so |

**external_deps**:
```gn
external_deps = [
    "relational_store:cj_relational_store_ffi",
]
```

**cj_external_deps**:
```gn
cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
]
```

**条件编译**: Windows/Mac 使用 mock

**证据**: ohos/data/relational_store/BUILD.gn:18-62

## 依赖关系图

### 依赖层级

```
bundle.json (组件定义)
    │
    ├── copy_sdk_distributeddatamgr_cangjie_libs (根 target)
    │       ├── ohos.data (基础包)
    │       ├── ohos.data.preferences
    │       │       └── external: preferences:cj_preferences_ffi
    │       ├── ohos.data.values_bucket
    │       ├── ohos.data.data_share_predicates
    │       │       ├── cj_deps: ohos.data.values_bucket
    │       │       └── external: data_share:cj_data_share_predicates_ffi
    │       ├── ohos.data.distributed_kv_store
    │       │       ├── cj_deps: ohos.data.data_share_predicates
    │       │       └── external: kv_store:cj_distributed_kv_store_ffi
    │       └── ohos.data.relational_store
    │               └── external: relational_store:cj_relational_store_ffi
    │
    └── kit.ArkData
            ├── cj_deps: 所有 ohos.data.* 模块
            └── (输出: libkit.ArkData.so)

外部组件依赖:
    ├── ability_cangjie_wrapper (应用上下文)
    ├── cangjie_ark_interop (仓颉互操作)
    ├── preferences (C++ 首选项服务)
    ├── data_share (C++ 数据共享服务)
    ├── kv_store (C++ KV 服务)
    ├── relational_store (C++ RDB 服务)
    └── hiviewdfx_cangjie_wrapper (HiLog)
```

## 编译产物

### 输出文件清单

| Target | 输出文件 | 平台 |
|--------|---------|------|
| kit.ArkData | libkit.ArkData.so | Linux/Android |
| ohos.data | libohos.data.so | Linux/Android |
| ohos.data.preferences | libohos.data.preferences.so | Linux/Android |
| ohos.data.values_bucket | libohos.data.values_bucket.so | Linux/Android |
| ohos.data.data_share_predicates | libohos.data.data_share_predicates.so | Linux/Android |
| ohos.data.distributed_kv_store | libohos.data.distributed_kv_store.so | Linux/Android |
| ohos.data.relational_store | libohos.data.relational_store.so | Linux/Android |

### 安装路径

#### SDK 安装路径

根据 `copy_sdk_distributeddatamgr_cangjie_libs` target，产物会被复制到：

```
${OUT_DIR}/libs/
    ├── ohos/
    │   ├── libohos.data.so
    │   ├── libohos.data.preferences.so
    │   ├── libohos.data.data_share_predicates.so
    │   ├── libohos.data.distributed_kv_store.so
    │   ├── libohos.data.relational_store.so
    │   └── libohos.data.values_bucket.so
    └── kit/
        └── libkit.ArkData.so
```

**证据**: BUILD.gn:25-28 - copy_ohos_cangjie_sdk_api_lib 模板使用

#### 运行时加载

应用运行时通过以下方式加载：
1. **仓颉运行时**：动态加载 `.so` 文件
2. **包导入**：应用通过 `import` 语句导入
   ```cangjie
   import kit.ArkData
   import ohos.data.distributed_kv_store
   ```

## 编译配置

### 条件编译

| 条件 | 值 | 影响的 Targets |
|------|-----|---------------|
| is_mingw | true | data_share_predicates, distributed_kv_store, relational_store, values_bucket 使用 mock |
| is_mac | true | data_share_predicates, distributed_kv_store, relational_store, values_bucket 使用 mock |
| 标准构建 | false | 使用真实实现 |

**证据**: 各模块 BUILD.gn 中的条件判断（如 values_bucket/BUILD.gn:21-24）

### 子系统信息

所有 targets 共享以下元数据：
```gn
subsystem_name = "distributeddatamgr"
part_name = "distributeddatamgr_cangjie_wrapper"
```

**证据**: kit/ArkData/BUILD.gn:31-33

## 构建命令

### 生成构建文件

```bash
# 设置构建工具链
hb set --cc-root

# 生成 Ninja 构建文件
hb build -f distributeddatamgr_cangjie_wrapper
```

### 编译特定 target

```bash
# 编译 kit.ArkData
hb build --build-target kit.ArkData

# 编译所有模块
hb build --build-target distributeddatamgr_cangjie_wrapper
```

## 产物大小估算

根据 bundle.json 中的资源占用：
- **ROM**: 900KB（所有共享库总大小）
- **RAM**: 864KB（运行时内存占用）

**证据**: bundle.json:20-21

## 相关跳转

- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
- [05_Internal_API.md](05_Internal_API.md) - FFI 层实现
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物详解
