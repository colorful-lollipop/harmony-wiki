# 04 - OpenHarmony 中的依赖关系与使用

## 4.1 使用方式概述

GN 在 OpenHarmony 中作为**构建基础设施**使用，所有 OH 组件都依赖 GN 进行构建。

### 使用特点

| 特点 | 说明 |
|-----|------|
| **工具链依赖** | 不通过 BUILD.gn 显式依赖，而是构建系统必需工具 |
| **全局使用** | 所有使用 GN 构建系统的组件都间接依赖 |
| **Prebuilt 形式** | 通过 prebuilt 工具链提供，非源码依赖 |
| **版本统一** | OH 使用统一版本的 GN |

## 4.2 依赖关系图

```mermaid
graph TD
    subgraph "OpenHarmony 构建系统"
        A[OH 构建脚本<br/>build.sh/hb] --> B[GN 工具]
        B --> C[Ninja]
        C --> D[编译器<br/>clang/gcc]
        D --> E[构建输出<br/>.so/.bin]
    end
    
    subgraph "典型组件依赖"
        F[组件 A<br/>BUILD.gn] -.->|使用| B
        G[组件 B<br/>BUILD.gn] -.->|使用| B
        H[组件 C<br/>BUILD.gn] -.->|使用| B
    end
    
    subgraph "依赖类型"
        I[显式依赖<br/>deps = []]
        J[隐式依赖<br/>构建工具]
    end
    
    style B fill:#f9f,stroke:#333,stroke-width:4px
```

## 4.3 谁在构建配置中引用 GN

### bundle.json 依赖声明

当前 `bundle.json` 中 `component.deps` 为空：

```json
"deps": {
    "components": [],
    "third_party": []
}
```

**原因**: GN 是构建工具而非运行时库，不通过组件依赖机制管理。

### 实际依赖者

所有包含 BUILD.gn 的 OH 组件都是 GN 的**隐式依赖者**。按子系统统计：

| 子系统 | 典型组件示例 | BUILD.gn 数量 |
|-------|-------------|--------------|
| foundation | arkui/ace_engine, multimedia/camera_framework | 1000+ |
| commonlibrary | c_utils, ets_utils | 200+ |
| drivers | hdf_core | 150+ |
| base | startup, security | 100+ |
| device | vendor 特定代码 |  varies |

## 4.4 典型使用场景

### 场景 1: 标准系统组件构建

```gn
# foundation/example/BUILD.gn
import("//build/ohos.gni")

ohos_shared_library("example_lib") {
  sources = [ "src/example.cpp" ]
  external_deps = [ "hilog:libhilog" ]
}

ohos_executable("example_tool") {
  deps = [ ":example_lib" ]
}
```

**GN 作用**: 解析 BUILD.gn，生成 Ninja 构建文件

### 场景 2: 测试组件构建

```gn
# test/unittest/BUILD.gn
import("//build/test.gni")

module_output_path = "example/unittest"

ohos_unittest("example_test") {
  sources = [ "test/example_test.cpp" ]
  external_deps = [ "gtest:gmock_main" ]
}
```

**GN 作用**: 处理测试模板，生成测试构建规则

### 场景 3: Lite 系统组件

```gn
# lite_component/BUILD.gn
import("//build/lite/config/component/lite_component.gni")

lite_component("lite_example") {
  features = [ ":lite_lib" ]
}
```

**GN 作用**: 支持轻量级系统构建配置

## 4.5 构建系统调用链

```
开发者执行
    │
    ▼
┌──────────────┐
│  hb build    │  ← 鸿蒙构建命令
│  或 build.sh │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   GN 生成    │  ← 调用 gn 生成 build.ninja
│ Ninja 文件   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Ninja 执行   │  ← ninja -C out
│  实际构建    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   构建输出   │  ← .so, .bin, .hap 等
└──────────────┘
```

## 4.6 GN 相关配置

### 全局 GN 参数

OH 构建系统传递的典型 GN 参数：

```gn
# OH 构建参数示例
target_os = "ohos"
target_cpu = "arm64"
ohos_version = "4.0"
use_musl = true
is_debug = false
```

### 组件级 GN 配置

组件可在 BUILD.gn 中定义的变量：

```gn
# 组件配置示例
declare_args() {
  # 特性开关
  enable_feature_x = true
  
  # 平台适配
  platform_specific_setting = "default"
}
```

## 4.7 依赖关系总结

### 直接依赖者

由于 GN 是构建工具，**没有直接的 BUILD.gn 依赖**。

### 间接依赖者（全部 OH 组件）

| 组件类型 | 数量估算 | 示例 |
|---------|---------|------|
| 系统服务 | 100+ | camera_service, media_service |
| 框架库 | 200+ | ace_engine, arkui |
| 驱动程序 | 150+ | hdf_core 各驱动 |
| 工具组件 | 50+ | bundletool, bm |
| 测试组件 | 500+ | 各组件的 unittest |

### 依赖类型说明

| 依赖类型 | 是否适用 | 说明 |
|---------|---------|------|
| 源码依赖 | 否 | GN 是工具，非库 |
| 编译依赖 | 是 | 所有组件编译都依赖 GN |
| 运行时依赖 | 否 | GN 不参与运行时 |
| 构建时依赖 | 是 | 构建必需工具 |

## 4.8 与其他构建工具的关系

```
┌────────────────────────────────────────────┐
│           OpenHarmony 构建工具栈            │
├────────────────────────────────────────────┤
│  高层: build.sh / hb (鸿蒙构建命令)         │
├────────────────────────────────────────────┤
│  配置: GN (生成 Ninja 文件)                 │
├────────────────────────────────────────────┤
│  执行: Ninja (并行构建执行)                 │
├────────────────────────────────────────────┤
│  编译: Clang/GCC (实际编译链接)             │
└────────────────────────────────────────────┘
```

## 4.9 维护与升级影响

### 升级 GN 版本的影响范围

| 影响项 | 程度 | 说明 |
|-------|-----|------|
| BUILD.gn 语法 | 可能变化 | 需检查语法兼容性 |
| 生成 Ninja 文件 | 必然变化 | 需验证构建正确性 |
| 组件构建结果 | 应无变化 | 仅构建过程变化 |
| 构建脚本 | 可能需要适配 | 检查 gn 调用参数 |

### 版本兼容性建议

- **向后兼容**: GN 通常保持向后兼容
- **测试策略**: 升级后在关键组件上验证构建
- **回滚准备**: 保留旧版本 prebuilt 作为备份
