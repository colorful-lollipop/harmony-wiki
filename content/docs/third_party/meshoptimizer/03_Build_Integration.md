# OH 构建系统集成

## 3.1 构建配置概述

### 3.1.1 构建目标类型

meshoptimizer 在 OpenHarmony 中编译为**共享库 (Shared Library)**：

```gn
ohos_shared_library("meshoptimizer") {
    # 库配置
}
```

### 3.1.2 配置文件清单

| 文件 | 位置 | 用途 |
|------|------|------|
| `BUILD.gn` | `third_party/meshoptimizer/` | OHOS 构建配置主文件 |
| `bundle.json` | `third_party/meshoptimizer/` | OHOS 组件元数据 |
| `CMakeLists.txt` | `third_party/meshoptimizer/` | 上游 CMake 配置（未使用） |

---

## 3.2 BUILD.gn 详细配置

### 3.2.1 完整配置

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.

# MIT LICENSE

import("//build/ohos.gni")

config("meshoptimizer_headers_config") {
    include_dirs = ["src"]
}

ohos_shared_library("meshoptimizer") {
    sources = [
        "src/meshoptimizer.h",
        "src/allocator.cpp",
        "src/clusterizer.cpp",
        "src/indexcodec.cpp",
        "src/indexgenerator.cpp",
        "src/overdrawanalyzer.cpp",
        "src/overdrawoptimizer.cpp",
        "src/quantization.cpp",
        "src/simplifier.cpp",
        "src/spatialorder.cpp",
        "src/stripifier.cpp",
        "src/vcacheanalyzer.cpp",
        "src/vcacheoptimizer.cpp",
        "src/vertexcodec.cpp",
        "src/vertexfilter.cpp",
        "src/vfetchanalyzer.cpp",
        "src/vfetchoptimizer.cpp",
    ]
    public_configs = [ ":meshoptimizer_headers_config" ]

    part_name = "meshoptimizer"
    subsystem_name = "thirdparty"
}
```

### 3.2.2 配置详解

#### 头文件配置

```gn
config("meshoptimizer_headers_config") {
    include_dirs = ["src"]
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `include_dirs` | `["src"]` | 导出 `src/` 目录为头文件搜索路径 |

**使用效果**: 消费者可以通过以下方式引用头文件：

```cpp
#include "meshoptimizer.h"
// 实际路径: third_party/meshoptimizer/src/meshoptimizer.h
```

#### 库配置

```gn
ohos_shared_library("meshoptimizer") {
    sources = [ ... ]
    public_configs = [ ":meshoptimizer_headers_config" ]
    part_name = "meshoptimizer"
    subsystem_name = "thirdparty"
}
```

| 属性 | 值 | 说明 |
|------|-----|------|
| `sources` | 17 个文件 | 包含所有核心算法的实现 |
| `public_configs` | 头文件配置 | 导出公共配置供消费者使用 |
| `part_name` | "meshoptimizer" | 部件名称 |
| `subsystem_name` | "thirdparty" | 子系统名称 |

---

## 3.3 源文件清单

### 3.3.1 编译源文件

| 序号 | 文件 | 功能模块 | 是否必需 |
|------|------|---------|---------|
| 1 | meshoptimizer.h | 头文件（包含内联实现） | ✅ 必需 |
| 2 | allocator.cpp | 内存分配器 | ✅ 必需 |
| 3 | clusterizer.cpp | 网格聚类 | ✅ 必需 |
| 4 | indexcodec.cpp | 索引编解码 | ✅ 必需 |
| 5 | indexgenerator.cpp | 索引生成 | ✅ 必需 |
| 6 | overdrawanalyzer.cpp | 过绘制分析 | ✅ 必需 |
| 7 | overdrawoptimizer.cpp | 过绘制优化 | ✅ 必需 |
| 8 | quantization.cpp | 量化算法 | ✅ 必需 |
| 9 | simplifier.cpp | 网格简化 | ✅ 必需 |
| 10 | spatialorder.cpp | 空间排序 | ✅ 必需 |
| 11 | stripifier.cpp | 三角形带生成 | ✅ 必需 |
| 12 | vcacheanalyzer.cpp | 顶点缓存分析 | ✅ 必需 |
| 13 | vcacheoptimizer.cpp | 顶点缓存优化 | ✅ 必需 |
| 14 | vertexcodec.cpp | 顶点编解码 | ✅ 必需 |
| 15 | vertexfilter.cpp | 顶点滤波 | ✅ 必需 |
| 16 | vfetchanalyzer.cpp | 顶点提取分析 | ✅ 必需 |
| 17 | vfetchoptimizer.cpp | 顶点提取优化 | ✅ 必需 |

### 3.3.2 未编译文件

| 文件/目录 | 说明 | 未编译原因 |
|----------|------|-----------|
| `gltf/` | gltfpack 工具 | 独立工具，按需单独编译 |
| `js/` | JavaScript 绑定 | Web 场景专用 |
| `demo/` | 示例程序 | 开发调试用 |
| `tools/` | 开发工具 | 构建工具 |
| `testcase/` | 测试代码 | 测试用 |
| `extern/` | 外部依赖 | 工具用依赖 |

---

## 3.4 模块化编译选项

### 3.4.1 最小配置

如果只需要部分功能，可以选择性编译：

#### 仅解码功能（glTF 场景）

```gn
ohos_shared_library("meshoptimizer_minimal") {
    sources = [
        "src/meshoptimizer.h",
        "src/allocator.cpp",
        "src/quantization.cpp",
        "src/vertexcodec.cpp",
        "src/vertexfilter.cpp",
        "src/indexcodec.cpp",
    ]
    # ...
}
```

#### 仅优化功能

```gn
ohos_shared_library("meshoptimizer_opt") {
    sources = [
        "src/meshoptimizer.h",
        "src/allocator.cpp",
        "src/clusterizer.cpp",
        "src/indexgenerator.cpp",
        "src/vcacheoptimizer.cpp",
        "src/overdrawoptimizer.cpp",
        "src/vfetchoptimizer.cpp",
        "src/simplifier.cpp",
    ]
    # ...
}
```

### 3.4.2 静态库变体

如需静态链接，可以使用 `ohos_static_library`：

```gn
ohos_static_library("meshoptimizer_static") {
    sources = [ ... ]  # 同上
    # ...
}
```

---

## 3.5 依赖关系

### 3.5.1 自身依赖

meshoptimizer **不依赖任何第三方库**：

| 依赖类型 | 状态 | 说明 |
|---------|------|------|
| 第三方库 | ❌ 无 | 零依赖设计 |
| OHOS 系统库 | ❌ 无 | 纯算法实现 |
| 标准库 | ✅ 有 | C++ 标准库 |

### 3.5.2 被依赖配置

#### bundle.json 中的 KIT 定义

```json
"inner_kits": [
    {
        "type": "so",
        "name": "//third_party/meshoptimizer:meshoptimizer",
        "header": {
            "header_files": ["src/meshoptimizer.h"],
            "header_base": "//third_party/meshoptimizer"
        }
    }
]
```

| KIT 属性 | 值 | 说明 |
|---------|-----|------|
| `type` | "so" | 共享库类型 |
| `name` | "//third_party/meshoptimizer:meshoptimizer" | 库目标路径 |
| `header_files` | ["src/meshoptimizer.h"] | 导出头文件 |
| `header_base` | "//third_party/meshoptimizer" | 头文件基准路径 |

---

## 3.6 编译选项配置

### 3.6.1 默认编译选项

meshoptimizer 使用 OHOS 默认编译选项，无需特殊配置：

| 编译选项 | 默认值 | 说明 |
|---------|-------|------|
| C++ 标准 | -std=gnu++17 | OHOS 标准 C++ 版本 |
| 优化级别 | -O2 | 默认优化级别 |
| RTTI | 取决于配置 | 运行时类型信息 |
| 异常处理 | 取决于配置 | C++ 异常支持 |

### 3.6.2 自定义编译选项（可选）

如需自定义编译选项，可以在配置中添加：

```gn
ohos_shared_library("meshoptimizer_custom") {
    sources = [ ... ]
    
    # 添加自定义定义
    defines = [
        "MESHOPT_USE_SIMD=1",     # 启用 SIMD 优化（如果工具链支持）
        "MESHOPT_ENABLE_TRACING=0", # 禁用追踪（减少代码大小）
    ]
    
    # 添加编译标志
    cflags = []
    cxxflags = [
        "-ffast-math",            # 快速数学运算
    ]
    
    # 排除特定警告
    suppressed_warnings = [
        "-Wunused-function",
    ]
}
```

---

## 3.7 构建验证

### 3.7.1 编译命令

```bash
# 编译 meshoptimizer
./build.sh --product-name <product> --ccache

# 仅编译 meshoptimizer
hb build -p <path_to_bundle.json>
```

### 3.7.2 验证步骤

#### 1. 检查编译输出

```bash
# 查找生成的库文件
find out/ -name "libmeshoptimizer.so"
```

#### 2. 检查头文件导出

```bash
# 验证头文件存在
ls -la third_party/meshoptimizer/src/
```

#### 3. 检查依赖关系（可选）

```bash
# 检查库依赖
readelf -d out/.../libmeshoptimizer.so
```

### 3.7.3 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 编译失败 | 源文件路径错误 | 检查 sources 列表 |
| 头文件找不到 | include_dirs 未配置 | 验证 public_configs |
| 链接失败 | 依赖缺失 | 检查 deps 配置 |

---

## 3.8 与上游构建系统的差异

### 3.8.1 CMake vs BUILD.gn

| 方面 | CMake (上游) | BUILD.gn (OHOS) |
|------|-------------|-----------------|
| 构建系统 | CMake | GN (Generate Ninja) |
| 目标类型 | STATIC/SHARED LIBRARY | ohos_shared_library |
| 头文件导出 | target_include_directories | public_configs |
| 模块化 | 可选组件 | 当前全量编译 |
| 安装目标 | install(TARGETS) | OHOS 自动安装 |

### 3.8.2 配置映射

| CMake 配置 | BUILD.gn 配置 |
|-----------|--------------|
| `add_library(meshoptimizer STATIC ...)` | `ohos_shared_library("meshoptimizer")` |
| `target_include_directories(... PUBLIC src)` | `config("meshoptimizer_headers_config") { include_dirs = ["src"] }` |
| `add_subdirectory(src)` | `sources = ["src/file.cpp", ...]` |

---

## 3.9 版本管理

### 3.9.1 版本映射

| OHOS 版本 | 上游版本 | 备注 |
|-----------|---------|------|
| 6.1 | v0.22 | 当前集成版本 |

### 3.9.2 版本更新流程

1. **检查上游 Release**
2. **下载并解压新版本**
3. **复制 src/ 目录**
4. **更新 BUILD.gn（如有新增文件）**
5. **更新 bundle.json 版本号**
6. **验证编译**
7. **更新文档**

---

## 3.10 总结

### 配置要点

| 配置项 | 状态 | 说明 |
|-------|------|------|
| 构建类型 | ✅ 共享库 | ohos_shared_library |
| 源文件 | ✅ 17 个 | 完整功能集 |
| 头文件导出 | ✅ 配置 | public_configs |
| 模块依赖 | ✅ 无依赖 | 零依赖设计 |
| 编译验证 | ✅ 通过 | OHOS 构建成功 |

### 最佳实践

1. **保持全量编译** - 当前配置已优化，无需特殊定制
2. **定期同步上游** - 源文件直接从上游复制
3. **验证编译** - 每次变更后执行编译验证
4. **更新文档** - 版本更新时同步更新本文档

---

*文档版本: v1.0*
*最后更新: 2026-02-07*
