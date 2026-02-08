# GN Targets 分析

## 目的

本文档详细说明 startup_cangjie_wrapper 的 GN 构建目标（targets），包括目标类型、依赖关系和构建产物。

## 适用范围

- 需要理解构建系统的开发者
- 准备修改构建配置的工程师

---

## 构建文件清单

### BUILD.gn 文件

| 路径 | 行数 | 目标数量 |
|------|------|---------|
| `/BUILD.gn` | 22 | 1 |
| `/ohos/device_info/BUILD.gn` | 37 | 1 |

**总计**: 2 个 BUILD.gn 文件，2 个构建目标

**证据**: `BUILD.gn:1-22`, `ohos/device_info/BUILD.gn:1-37`

---

## 构建目标详解

### 根目录目标：copy_sdk_startup_cangjie_libs

**路径**: `/BUILD.gn`

**目标名称**: `copy_sdk_startup_cangjie_libs`

**目标类型**: `copy_ohos_cangjie_sdk_api_lib`

**子系统和部件**:
- subsystem_name: 未设置
- part_name: 未设置

**输入**: `startup_cangjie_wrapper_packages_ohos` 列表

**依赖**: 无

**代码**:
```gn
import("//build/templates/cangjie/cjc.gni")

startup_cangjie_wrapper_packages_ohos = [
    "//base/startup/startup_cangjie_wrapper/ohos/device_info:ohos.device_info"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_startup_cangjie_libs") {
  ohos_inputs = startup_cangjie_wrapper_packages_ohos
}
```

**证据**: `BUILD.gn:14-22`

**职责**: 复制仓颉 SDK 库到目标位置

---

### device_info 目标：ohos.device_info

**路径**: `/ohos/device_info/BUILD.gn`

**目标名称**: `ohos.device_info`

**目标类型**: `ohos_cangjie_shared_library`

**子系统和部件**:
- subsystem_name: `startup`
- part_name: `startup_cangjie_wrapper`

**sources**: 根据 platform 选择
- Windows/Mac: `mock/ohos.device_info.cj`
- 其他: `device_info.cj`

**external_deps**:
- `init:cj_device_info_ffi`

**cj_external_deps**:
- `cangjie_ark_interop:ohos.labels`

**代码**:
```gn
import("//build/ohos.gni")
import("//build/templates/cangjie/cjc.gni")

ohos_cangjie_shared_library("ohos.device_info") {
  if (is_mingw || is_mac){
    sources = [ "../../mock/ohos.device_info.cj" ]
  } else {
    sources = [ "device_info.cj" ]
  }

  external_deps = [ "init:cj_device_info_ffi" ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.labels",
  ]

  subsystem_name = "startup"
  part_name = "startup_cangjie_wrapper"
}
```

**证据**: `ohos/device_info/BUILD.gn:20-36`

**职责**: 编译 ohos.device_info 仓颉共享库

---

## 目标依赖图

```
copy_sdk_startup_cangjie_libs (copy_ohos_cangjie_sdk_api_lib)
  │
  ├─ ohos_inputs
  │   └─ //base/startup/startup_cangjie_wrapper/ohos/device_info:ohos.device_info
  │
  └─ ohos.device_info (ohos_cangjie_shared_library)
      │
      ├─ sources
      │   ├─ device_info.cj (Linux/OpenHarmony)
      │   └─ mock/ohos.device_info.cj (Windows/Mac)
      │
      ├─ external_deps
      │   └─ init:cj_device_info_ffi
      │
      ├─ cj_external_deps
      │   └─ cangjie_ark_interop:ohos.labels
      │
      ├─ subsystem_name: startup
      └─ part_name: startup_cangjie_wrapper
```

**证据**: `BUILD.gn:14-22`, `ohos/device_info/BUILD.gn:20-36`

---

## 目标属性详细说明

### copy_sdk_startup_cangjie_libs

| 属性 | 值 | 说明 |
|------|-----|------|
| target_type | copy_ohos_cangjie_sdk_api_lib | 复制仓颉 SDK 库 |
| ohos_inputs | startup_cangjie_wrapper_packages_ohos | 输入列表 |
| deps | 无 | 无依赖 |
| public_deps | 无 | 无公共依赖 |

### ohos.device_info

| 属性 | 值 | 说明 |
|------|-----|------|
| target_type | ohos_cangjie_shared_library | 仓颉共享库 |
| sources | 条件选择 | 根据 platform 选择源文件 |
| external_deps | ["init:cj_device_info_ffi"] | init 组件 FFI 依赖 |
| cj_external_deps | ["cangjie_ark_interop:ohos.labels"] | 仓颉互操作依赖 |
| subsystem_name | startup | 子系统名称 |
| part_name | startup_cangjie_wrapper | 部件名称 |
| include_dirs | 无 | 无额外包含目录 |
| defines | 无 | 无预定义宏 |
| configs | 无 | 无额外配置 |

---

## 构建配置

### Import 语句

| 文件 | Import 语句 | 用途 |
|------|-----------|------|
| `/BUILD.gn` | `import("//build/templates/cangjie/cjc.gni")` | 仓颉编译模板 |
| `/ohos/device_info/BUILD.gn` | `import("//build/ohos.gni")` | OpenHarmony 构建配置 |
| `/ohos/device_info/BUILD.gn` | `import("//build/templates/cangjie/cjc.gni")` | 仓颉编译模板 |

**证据**: `BUILD.gn:14`, `ohos/device_info/BUILD.gn:16,18`

### 平台配置切换

**条件**: `if (is_mingw || is_mac)`

**Windows/Mac 环境**:
```gn
sources = [ "../../mock/ohos.device_info.cj" ]
```

**Linux/OpenHarmony 环境**:
```gn
sources = [ "device_info.cj" ]
```

**证据**: `ohos/device_info/BUILD.gn:22-26`

---

## 依赖分析

### 外部依赖

| 依赖项 | 类型 | 用途 |
|--------|------|------|
| init:cj_device_info_ffi | external_deps | 设备信息 SA 服务 FFI 接口 |
| cangjie_ark_interop:ohos.labels | cj_external_deps | 仓颉注解类（APILevel、Hide） |

**证据**: `ohos/device_info/BUILD.gn:28-32`

### 依赖关系图

```
ohos.device_info
  │
  ├─ external_deps
  │   └─ init:cj_device_info_ffi
  │       └─ 提供设备信息 SA 服务 FFI 接口
  │
  └─ cj_external_deps
      └─ cangjie_ark_interop:ohos.labels
          ├─ APILevel 注解
          └─ Hide 注解
```

---

## Target ↔ 产物映射

### copy_sdk_startup_cangjie_libs

**输出**: 复制仓颉 SDK 库到目标位置

**产物位置**: TODO（需要确认）

### ohos.device_info

**目标类型**: `ohos_cangjie_shared_library`

**输出文件**: TODO（需要确认，可能是 `libohos.device_info.so`）

**安装路径**: TODO（需要确认，可能在 `/usr/lib/` 或 SDK 目录）

**运行时加载**: 由仓颉运行时自动加载

---

## 构建命令

### 编译 ohos.device_info

```bash
# 标准 OpenHarmony 环境
./build.sh --product-name=产品名 --build-target=ohos.device_info

# 或使用 hb 工具
hb build -f ohos.device_info
```

### 编译所有目标

```bash
# 编译整个 startup_cangjie_wrapper 组件
./build.sh --product-name=产品名 --build-target=startup_cangjie_wrapper

# 或使用 hb 工具
hb build -f startup_cangjie_wrapper
```

### Windows/Mac 环境

```bash
# 会自动选择 mock 实现
./build.sh --product-name=产品名 --build-target=ohos.device_info
```

---

## 关键结论

1. **目标简单**: 仅 2 个构建目标，结构清晰
2. **条件编译**: 通过 `is_mingw || is_mac` 切换 mock 实现
3. **依赖明确**: 仅依赖 init 组件和 cangjie_ark_interop
4. **无复杂配置**: 无 include_dirs、defines、configs
5. **无子目标**: 每个目录仅 1 个目标
6. **产物映射**: TODO（需要确认）

---

## TODO（待确认）

- [ ] 确认 ohos.device_info 的实际输出文件名和扩展名
- [ ] 确认产物的实际安装路径
- [ ] 确认 copy_sdk_startup_cangjie_libs 的实际复制目标路径
- [ ] 确认仓颉共享库的运行时加载机制

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物

---

## 参考资料

- [OpenHarmony GN 构建系统](https://docs.openharmony.cn/application-dev/quick-start/naming-rules)
- [仓颉语言编译规范](https://developer.openharmony.cn/cn/doc/cangjie-compilation)
- [OpenHarmony 构建指南](https://docs.openharmony.cn/application-dev/quick-start/start-overview)

---

*最后更新: 2026-02-06*
