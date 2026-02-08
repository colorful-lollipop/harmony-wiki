# 05 - API/接口差异

## 5.1 概述

GN 是一个**元构建系统工具**，并非应用程序库，因此不存在传统意义上的 "API 差异"。

本节主要说明：
1. OH Patch 引入的行为变更
2. GN 配置在 OH 中的使用方式
3. 与上游 GN 的功能等价性

## 5.2 Patch 引入的行为变更

### 路径表示变更

| 项目 | 上游 GN | OH GN (Patch 后) |
|-----|--------|-----------------|
| 生成文件路径表示 | `obj/BUILD_DIR/gen/foo.o` | `obj/out/Debug/gen/foo.o` |
| 占位符支持 | 支持 BUILD_DIR 占位符 | 不支持，使用实际路径 |

**说明**: 
- 这不是 API 变更，而是内部路径表示方式的变化
- 对开发者编写的 BUILD.gn 没有影响
- 仅影响生成的 build.ninja 文件中的路径字符串

### 功能等价性

| 功能 | 上游 | OH | 差异 |
|-----|-----|---|-----|
| GN 语法解析 | ✓ | ✓ | 无差异 |
| Ninja 文件生成 | ✓ | ✓ | 无差异 |
| 构建目标定义 | ✓ | ✓ | 无差异 |
| 依赖管理 | ✓ | ✓ | 无差异 |
| 工具链配置 | ✓ | ✓ | 无差异 |
| 路径占位符 | ✓ | ✗ | OH 移除 BUILD_DIR 支持 |

## 5.3 OH 特定的 GN 配置

### 标准模板导入

OH 组件通常导入以下 .gni 文件：

```gn
# 标准系统组件
import("//build/ohos.gni")

# 测试组件
import("//build/test.gni")

# Lite 系统组件
import("//build/lite/config/component/lite_component.gni")
```

### OH 特有 GN 模板

| 模板 | 位置 | 用途 |
|-----|------|-----|
| `ohos_shared_library` | //build/ohos.gni | 定义共享库 |
| `ohos_executable` | //build/ohos.gni | 定义可执行文件 |
| `ohos_unittest` | //build/test.gni | 定义单元测试 |
| `lite_component` | //build/lite/... | Lite 系统组件 |
| `ohos_prebuilt_etc` | //build/ohos.gni | 预置文件 |

这些模板是 OH 构建系统提供的**标准接口**，而非 GN 本身的 API。

## 5.4 开发者视角的等价性

### BUILD.gn 编写

开发者在 OH 中编写 BUILD.gn 的方式与在其他 GN 项目中相同：

```gn
# 这是标准 GN 语法，在 OH 和其他 GN 项目中完全相同
executable("my_tool") {
  sources = [ "main.cc" ]
  deps = [ "//path/to:lib" ]
}
```

### GN 命令使用

| 命令 | 上游 | OH | 说明 |
|-----|-----|---|-----|
| `gn gen` | ✓ | ✓ | 生成构建文件 |
| `gn check` | ✓ | ✓ | 检查 BUILD.gn |
| `gn desc` | ✓ | ✓ | 描述目标 |
| `gn clean` | ✓ | ✓ | 清理构建 |
| `gn args` | ✓ | ✓ | 管理构建参数 |

**结论**: 从开发者视角，OH GN 与上游 GN 的**用户接口完全一致**。

## 5.5 不适用项

以下内容不适用于 GN：

- ❌ 运行时 API（GN 是构建时工具）
- ❌ ABI 兼容性（GN 不生成运行时库）
- ❌ 头文件接口（GN 是配置语言）
- ❌ 版本兼容性检查（GN 版本由构建系统管理）

## 5.6 总结

| 方面 | 差异程度 | 说明 |
|-----|---------|------|
| 用户接口 | 无差异 | BUILD.gn 语法、GN 命令完全一致 |
| 行为变更 | 轻微 | 仅内部路径表示变化 |
| 功能变更 | 无 | 所有核心功能保持等价 |
| 配置接口 | OH 扩展 | OH 提供额外的 .gni 模板 |

**最终结论**: GN 在 OH 中与上游版本保持**高度兼容**，开发者无需为 OH 学习新的 GN 语法或接口。
