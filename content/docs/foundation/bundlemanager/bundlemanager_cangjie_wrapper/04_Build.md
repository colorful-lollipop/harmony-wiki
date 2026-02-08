# 构建配置

> bundlemanager_cangjie_wrapper GN Targets 与编译产物

## 构建文件清单

| 文件路径 | 作用 |
|----------|------|
| `BUILD.gn` | 根构建配置，SDK 产物拷贝 |
| `ohos/bundle/BUILD.gn` | bundle 包构建配置 |
| `ohos/bundle/bundle_manager/BUILD.gn` | bundle_manager 模块构建配置 |
| `ohos/element_name/BUILD.gn` | element_name 模块构建配置 |
| `ohos/metadata/BUILD.gn` | metadata 模块构建配置 |
| `ohos/skill/BUILD.gn` | skill 模块构建配置 |

**证据来源**: `BUILD.gn` 系列文件

---

## Targets 清单

### 根目录 BUILD.gn

**Target**: `copy_sdk_bundlemanager_cangjie_libs`
- **类型**: `copy_ohos_cangjie_sdk_api_lib`
- **sources**: 4 个包引用
- **功能**: 将构建产物拷贝到 SDK 目录

```gn
bundlemanager_cangjie_wrapper_packages_ohos = [
    "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/bundle/bundle_manager:ohos.bundle.bundle_manager",
    "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/skill:ohos.skill",
    "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/element_name:ohos.element_name",
    "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/metadata:ohos.metadata"
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_bundlemanager_cangjie_libs") {
  ohos_inputs = bundlemanager_cangjie_wrapper_packages_ohos
}
```

**证据来源**: `BUILD.gn:16-25`

---

### ohos/bundle/BUILD.gn

**Target**: `ohos.bundle`
- **类型**: `ohos_cangjie_shared_library`
- **sources**:
  - 条件编译 (`is_mingw || is_mac`): `../../mock/ohos.bundle.cj`
  - 其他平台: `bundle.cj`

```gn
ohos_cangjie_shared_library("ohos.bundle") {
  if (is_mingw || is_mac){
    sources = ["../../mock/ohos.bundle.cj"]
  } else {
    sources = ["bundle.cj"]
  }

  subsystem_name = "bundlemanager"
  part_name = "bundlemanager_cangjie_wrapper"
}
```

**证据来源**: `ohos/bundle/BUILD.gn:19-29`

---

### ohos/bundle/bundle_manager/BUILD.gn

**Target**: `ohos.bundle.bundle_manager`
- **类型**: `ohos_cangjie_shared_library`
- **sources** (11 个 .cj 文件):
  - `ability_info.cj`
  - `application_info.cj`
  - `bundle_flag.cj`
  - `bundle_info.cj`
  - `bundle_manager.cj`
  - `cj_bundle_enum.cj`
  - `cj_bundle_ffi.cj`
  - `cj_bundle_utils.cj`
  - `default_app_manager.cj`
  - `error_code.cj`
  - `error_message.cj`
  - `extension_ability_info.cj`
  - `hap_module_info.cj`

**内部依赖 (cj_deps)**:
```
├── //ohos/element_name:ohos.element_name
├── //ohos/metadata:ohos.metadata
└── //ohos/skill:ohos.skill
```

**Cangjie 外部依赖 (cj_external_deps)**:
```
├── cangjie_ark_interop:ohos.business_exception
├── cangjie_ark_interop:ohos.labels
├── cangjie_ark_interop:ohos.ffi
├── hiviewdfx_cangjie_wrapper:ohos.hilog
└── global_cangjie_wrapper:ohos.resource
```

**Native 外部依赖 (external_deps)**:
```
└── bundle_framework:cj_bundle_manager_ffi
```

**证据来源**: `ohos/bundle/bundle_manager/BUILD.gn:18-60`

---

### ohos/element_name/BUILD.gn

**Target**: `ohos.element_name`
- **类型**: `ohos_cangjie_shared_library`
- **sources**:
  - 条件编译: `../../mock/ohos.element_name.cj`
  - 其他平台: `element_name.cj`

**Cangjie 外部依赖 (cj_external_deps)**:
```
├── cangjie_ark_interop:ohos.ffi
└── cangjie_ark_interop:ohos.labels
```

**Native 外部依赖 (external_deps)**:
```
└── ability_runtime:cj_ability_ffi
```

**证据来源**: `ohos/element_name/BUILD.gn`

---

### ohos/metadata/BUILD.gn

**Target**: `ohos.metadata`
- **类型**: `ohos_cangjie_shared_library`
- **sources**:
  - 条件编译: `../../mock/ohos.metadata.cj`
  - 其他平台: `metadata.cj`

**Cangjie 外部依赖 (cj_external_deps)**:
```
├── cangjie_ark_interop:ohos.ffi
└── cangjie_ark_interop:ohos.labels
```

**证据来源**: `ohos/metadata/BUILD.gn`

---

### ohos/skill/BUILD.gn

**Target**: `ohos.skill`
- **类型**: `ohos_cangjie_shared_library`
- **sources**:
  - 条件编译: `../../mock/ohos.skill.cj`
  - 其他平台: `skill.cj`, `skill_ffi.cj`

**Cangjie 外部依赖 (cj_external_deps)**:
```
├── cangjie_ark_interop:ohos.business_exception
├── cangjie_ark_interop:ohos.ffi
└── cangjie_ark_interop:ohos.labels
```

**证据来源**: `ohos/skill/BUILD.gn`

---

## 依赖关系图

```
                           bundle.json
                              │
                              ▼
                   ┌──────────────────────┐
                   │ copy_sdk_bundlemanager│
                   │ _cangjie_libs        │
                   │ (SDK 拷贝)           │
                   └──────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐   ┌─────────────────┐   ┌─────────────────┐
│ ohos.element  │   │  ohos.metadata  │   │   ohos.skill    │
│    _name      │   │                 │   │                 │
└───────────────┘   └─────────────────┘   └─────────────────┘
        │                     │                     │
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
              ┌─────────────────────────────────────┐
              │    ohos.bundle.bundle_manager       │
              │    (核心业务模块)                   │
              │                                     │
              │  内部依赖:                          │
              │  - element_name                    │
              │  - metadata                        │
              │  - skill                           │
              │                                     │
              │  外部依赖:                          │
              │  - bundle_framework (native)        │
              │  - cangjie_ark_interop             │
              │  - hiviewdfx_cangjie_wrapper       │
              │  - global_cangjie_wrapper           │
              └─────────────────────────────────────┘
                              │
                              ▼
              ┌─────────────────────────────────────┐
              │          ohos.bundle                │
              │      (Bundle 命名空间)              │
              └─────────────────────────────────────┘
```

---

## 条件编译

### 平台条件

| 条件 | 影响模块 | 说明 |
|------|----------|------|
| `is_mingw \|\| is_mac` | 所有子模块 | Windows/Mac 平台使用 mock 文件 |

### 受影响模块

| 模块 | 正常 sources | Mock sources |
|------|--------------|--------------|
| `ohos.bundle` | `bundle.cj` | `../../mock/ohos.bundle.cj` |
| `ohos.bundle.bundle_manager` | 13 个 .cj 文件 | `../../../mock/ohos.bundle.bundle_manager.cj` |
| `ohos.element_name` | `element_name.cj` | `../../mock/ohos.element_name.cj` |
| `ohos.metadata` | `metadata.cj` | `../../mock/ohos.metadata.cj` |
| `ohos.skill` | `skill.cj`, `skill_ffi.cj` | `../../mock/ohos.skill.cj` |

---

## 编译产物

### 预期产物

| 模块 | 产物类型 | 产物名称 |
|------|----------|----------|
| `ohos.bundle` | .so | `libohos.bundle.z.so` |
| `ohos.bundle.bundle_manager` | .so | `libohos.bundle.bundle_manager.z.so` |
| `ohos.element_name` | .so | `libohos.element_name.z.so` |
| `ohos.metadata` | .so | `libohos.metadata.z.so` |
| `ohos.skill` | .so | `libohos.skill.z.so` |

### SDK 产物

产物通过 `copy_sdk_bundlemanager_cangjie_libs` 拷贝到 SDK 目录。

---

## bundle.json 配置

```json
{
    "name": "@ohos/bundlemanager_cangjie_wrapper",
    "description": "The bundlemanager_cangjie_wrapper is a Cangjie API encapsulated on OpenHarmony based on the bundle_framework subsystem.",
    "version": "6.1",
    "component": {
        "name": "bundlemanager_cangjie_wrapper",
        "subsystem": "bundlemanager",
        "adapted_system_type": ["standard"],
        "rom": "400KB",
        "ram": "468KB",
        "deps": {
            "components": [
                "cangjie_ark_interop",
                "global_cangjie_wrapper",
                "hiviewdfx_cangjie_wrapper",
                "bundle_framework",
                "ability_runtime"
            ]
        },
        "build": {
            "sub_component": [
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/bundle/bundle_manager:ohos.bundle.bundle_manager",
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/element_name:ohos.element_name",
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/metadata:ohos.metadata",
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/skill:ohos.skill"
            ],
            "inner_kits": [
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/bundle/bundle_manager:ohos.bundle.bundle_manager",
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper/ohos/element_name:ohos.element_name",
                "//foundation/bundlemanager/bundlemanager_cangjie_wrapper:copy_sdk_bundlemanager_cangjie_libs"
            ]
        }
    }
}
```

**证据来源**: `bundle.json`

---

## 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [02_API_Reference.md](02_API_Reference.md) - API 参考
- [SUMMARY.md](SUMMARY.md) - 文档导航
