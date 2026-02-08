# 构建与产物

> GN 构建配置、编译产物和运行时加载关系

## 目的

本文档描述分布式数据对象组件的 GN 构建配置、编译产物和运行时加载关系，帮助开发者理解和修改构建流程。

## 适用范围

- OpenHarmony 标准系统
- 组件版本 3.1.0
- GN 构建系统

---

## GN 目标清单

### interfaces/innerkits/BUILD.gn

| Target | 类型 | 输出 | 关键依赖 |
|--------|------|------|----------|
| `distributeddataobject_impl` | ohos_shared_library | `libnative_dataobject.so` | `kv_store:distributeddata_inner`, `distributeddb`, `dsoftbus` |
| `distributeddataobject_static` | ohos_static_library | `.a` 静态库 | 同上 |
| `data_object_inner` | ohos_static_library | Platform SDK | `hisysevent:libhisysevent` |

**证据**: `interfaces/innerkits/BUILD.gn`

### interfaces/jskits/BUILD.gn

| Target | 类型 | 输出 | 关键依赖 |
|--------|------|------|----------|
| `build_module` | group | - | `distributeddataobject`, `gen_distributed_data_object_abc` |
| `distributeddataobject` | ohos_shared_library | `module/data/libdistributeddataobject.z.so` | `innerkits:distributeddataobject_impl`, `ability_runtime`, `napi` |
| `gen_distributed_data_object_abc` | es2abc_gen_abc | `.abc` 字节码 | - |

**证据**: `interfaces/jskits/BUILD.gn`

### frameworks/ets/taihe/ohos.data.distributedDataObject/BUILD.gn

| Target | 类型 | 输出 |
|--------|------|------|
| `distributed_dataobject_ani` | taihe_shared_library | ANI 共享库 |
| `ohos_distributeddataobject_etc` | ohos_prebuilt_etc | `framework/` 下的 ABC |

**证据**: `frameworks/ets/taihe/ohos.data.distributedDataObject/BUILD.gn`

### frameworks/jskitsimpl/collaboration_edit/BUILD.gn

| Target | 类型 | 输出 |
|--------|------|------|
| `collaborationeditobject` | ohos_shared_library | `module/data/libcollaborationeditobject.z.so` |

**证据**: `frameworks/jskitsimpl/collaboration_edit/BUILD.gn`

---

## 编译产物

### 库文件

| 产物类型 | 路径 | 用途 |
|----------|------|------|
| `libnative_dataobject.so` | `/usr/lib/` 或系统库目录 | 平台 SDK C++ 接口 |
| `libdistributeddataobject.z.so` | `/system/module/data/` | JS N-API 模块 |
| `libcollaborationeditobject.z.so` | `/system/module/data/` | 协作编辑 N-API 模块 |
| `.abc` 字节码 | `/system/module/framework/` | 编译后的 JS 代码 |

**运行时加载**:
- JS 引擎加载 `.z.so` 模块
- 模块注册 N-API 接口
- 模块加载 `.abc` 字节码

### Feature 开关

TODO: 识别关键宏和配置选项

---

## 证据

- GN 配置: `interfaces/innerkits/BUILD.gn`, `interfaces/jskits/BUILD.gn`
- 输出路径: BUILD.gn 中的 `output_dir` 配置

## 相关链接

- [代码地图](./02_CodeMap.md)
- [项目概览](./00_Overview.md)
- [内部实现](./07_Internals.md)
