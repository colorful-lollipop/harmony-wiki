# Patch 详细分析

## 1. Patch 清单总览

### 1.1 结论先行

**glslang 在 OpenHarmony 中是「无 Patch」导入。**

这与许多其他第三方库不同，glslang 的 OH 适配**完全通过编译宏和构建系统配置**完成，没有修改任何源代码文件。

### 1.2 Patch 统计

| 项目 | 数量 |
|------|------|
| Patch 文件 | 0 |
| 修改的源文件 | 0 |
| OH 特有宏 (`OH_SDK`) | 10 处 |
| 构建配置调整 | 多处 |

---

## 2. OH_SDK 宏使用分析

虽然无传统意义上的 Patch，但 `OH_SDK` 宏在代码中被用于条件编译。以下是详细分析：

### 2.1 OH_SDK 宏定义位置

`OH_SDK` 宏在以下 BUILD.gn 目标中被定义：

```gn
# BUILD.gn (根目录)

# 在 glslang_validator 目标中
ohos_executable("glslang_validator") {
  defines = [
    "ENABLE_OPT=1",
    "OH_SDK",  # ← 定义宏
  ]
  ...
}

# 在 spirv-remap 目标中  
ohos_executable("spirv-remap") {
  defines = [
    "ENABLE_OPT=1",
    "OH_SDK",  # ← 定义宏
  ]
  ...
}

# 在 glslang_sources_common 模板中 (条件定义)
template("glslang_sources_common") {
  if (build_ohos_sdk) {
    defines += [ "OH_SDK" ]  # ← 条件定义
    cflags += [ "-std=c++17" ]
  }
}
```

### 2.2 OH_SDK 宏使用详情

#### 2.2.1 StandAlone/spirv-remap.cpp

| 行号 | 代码 | 用途分析 |
|------|------|----------|
| 40 | `#ifndef OH_SDK` | 排除某些功能 |
| 97 | `#ifndef OH_SDK` | 排除某些功能 |
| 101 | `#endif // OH_SDK` | 结束条件块 |
| 372 | `#ifdef OH_SDK` | 包含 OH 特定代码 |
| 381 | `#endif // OH_SDK` | 结束条件块 |
| 407 | `#ifndef OH_SDK` | 排除某些功能 |

**推测用途**: 
- `#ifndef OH_SDK` 块可能包含特定于桌面平台的功能（如完整的文件系统操作、调试功能等）
- `#ifdef OH_SDK` 块可能包含移动端/嵌入式友好的替代实现

#### 2.2.2 glslang/OSDependent/Unix/ossource.cpp

| 行号 | 代码 | 用途分析 |
|------|------|----------|
| 42 | `#if !defined(__Fuchsia__) && !defined(OH_SDK)` | 排除 Fuchsia 特定代码 |

**分析**:
- 这里 `OH_SDK` 与 `__Fuchsia__` 并列使用
- 表明 OpenHarmony 和 Fuchsia 在某些系统调用上有相似的处理方式
- 被排除的代码可能是 Fuchsia 特有的线程本地存储 (TLS) 或其他 OS 抽象层实现

---

## 3. 构建系统适配

### 3.1 适配方式对比

传统 Patch vs glslang 的适配方式：

| 适配方式 | 传统库 | glslang |
|----------|--------|---------|
| 代码修改 | .patch 文件 | ❌ 无 |
| 编译宏 | 可能使用 | ✅ `OH_SDK` |
| 构建系统 | GN 适配 | ✅ 双重 BUILD.gn |
| 配置文件 | 修改 | ✅ 新增 OH 风格配置 |

### 3.2 构建系统差异

#### 上游 CMakeLists.txt → OH BUILD.gn 的主要变化

| 方面 | CMake (上游) | GN (OH) |
|------|--------------|---------|
| 构建目标 | 动态/静态库可选 | 静态库 + 可执行文件 |
| SPIRV-Tools | 外部可选依赖 | 完全禁用 (`ENABLE_OPT=0`) |
| 安装 | CMake install | part/subsystem 标记 |
| 测试 | Google Test | XTS 框架 |
| 产物命名 | 标准命名 | `libdeqp_*` 前缀 (CTS 兼容) |

### 3.3 关键编译选项

| 选项 | 上游默认 | OH 配置 | 差异原因 |
|------|----------|---------|----------|
| `ENABLE_HLSL` | 可选 | 强制启用 | CTS 需要 HLSL 支持 |
| `ENABLE_OPT` | 可选 | 强制禁用 | 减少 spirv-tools 依赖 |
| `GLSLANG_OSINCLUDE_UNIX` | 自动检测 | 强制定义 | 明确使用 Unix 实现 |
| C++ 标准 | C++17 | C++17 | 一致 |
| RTTI | 通常启用 | 禁用 | 减小二进制体积 |
| 异常 | 通常启用 | 禁用 | 嵌入式友好 |

---

## 4. 无 Patch 的优势与风险

### 4.1 优势

| 优势 | 说明 |
|------|------|
| **升级简单** | 直接替换上游代码，无需重新应用 Patch |
| **维护成本低** | 无需维护 Patch 文件，减少维护负担 |
| **代码纯净** | 与上游保持完全一致，便于问题定位 |
| **社区同步** | 更容易跟上上游开发节奏 |

### 4.2 风险与注意事项

| 风险 | 说明 | 缓解措施 |
|------|------|----------|
| **功能受限** | 禁用 SPIRV-Tools 优化 | 确保 CTS 测试不依赖优化 |
| **宏控制复杂** | `OH_SDK` 宏分散在多处 | 文档化所有使用位置 |
| **构建适配** | 双重 BUILD.gn 可能冲突 | 升级时同时检查两套配置 |

---

## 5. 升级指南

### 5.1 升级步骤

由于 glslang 无 Patch，升级流程简化：

```bash
# 1. 备份当前配置
cp -r third_party/glslang/BUILD.gn third_party/glslang/BUILD.gn.bak
cp -r third_party/glslang/glslang/BUILD.gn third_party/glslang/glslang/BUILD.gn.bak
cp -r third_party/glslang/SPIRV/BUILD.gn third_party/glslang/SPIRV/BUILD.gn.bak

# 2. 替换上游代码（保留 BUILD.gn）
# ... 执行代码替换 ...

# 3. 检查新版本的 BUILD.gn 是否需要更新
# 对比上游 BUILD.gn 模板与 OH 版本的差异

# 4. 构建测试
# ... 执行构建 ...

# 5. 运行 CTS 测试验证
# ... 运行 vk-gl-cts ...
```

### 5.2 升级检查清单

- [ ] 检查 `ShaderLang.h` 公共 API 是否有破坏性变更
- [ ] 检查新的源文件是否需要在 BUILD.gn 中添加
- [ ] 检查 `OH_SDK` 宏的使用是否需要调整
- [ ] 运行 vk-gl-cts 确保测试通过
- [ ] 验证 `glslang_validator` 和 `spirv-remap` 可执行文件功能

---

## 6. 总结

### 6.1 Patch 分析结论

| 项目 | 结论 |
|------|------|
| **Patch 数量** | 0 |
| **修改方式** | 编译宏 + 构建配置 |
| **侵入性** | 低（非侵入式） |
| **维护难度** | 低 |

### 6.2 关键适配点

1. **`OH_SDK` 宏** - 控制平台特定代码路径
2. **双重 BUILD.gn** - 兼容 Chromium/Fuchsia 和 OH 构建系统
3. **禁用 SPIRV-Tools** - 简化依赖链
4. **C++17 + 禁用异常/RTTI** - 嵌入式优化

### 6.3 维护建议

1. **保持无 Patch 策略** - 继续使用编译宏进行适配
2. **文档化宏使用** - 每次升级时检查 `OH_SDK` 使用位置
3. **测试覆盖** - 确保 vk-gl-cts 测试覆盖所有使用场景
4. **版本跟踪** - 记录与 Vulkan SDK 版本的对应关系
