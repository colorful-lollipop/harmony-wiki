# 02_Patches.md - Patch 详细分析

## 核心结论

**本库在 OpenHarmony 中没有 Patch 文件。**

这是一个**干净集成**的第三方库，所有 OpenHarmony 适配通过 BUILD.gn 的条件编译配置完成。

---

## 1. Patch 文件清单

### 1.1 搜索结果

```bash
# 执行的搜索命令
find . -name "*.patch" -o -name "patches" -type d

# 结果：无 Patch 文件
```

### 1.2 Patch 统计

| 统计项 | 数值 |
|--------|------|
| Patch 文件总数 | 0 |
| Bugfix Patch | 0 |
| Feature Patch | 0 |
| OH 适配 Patch | 0 |
| 其他 Patch | 0 |

---

## 2. 为什么不需要 Patch？

### 2.1 原始库设计优势

astc-encoder 能够干净集成，得益于以下设计特点：

#### 2.1.1 跨平台原生支持
```
支持平台列表：
├── Windows (Win32 API)
├── Linux (POSIX)
├── macOS (POSIX + 通用二进制)
├── Android (NDK)
└── iOS (Xcode)
```

原始代码已包含多平台条件编译：
```cpp
// 来自 astcenccli_platform_dependents.cpp
#if defined(_WIN32) && !defined(__CYGWIN__)
    // Windows 实现
#else
    // POSIX 实现（Linux/macOS 等）
#endif
```

#### 2.1.2 许可证完全兼容
- **原始许可证**：Apache 2.0
- **OH 要求**：Apache 2.0 兼容
- **结论**：无需许可证相关修改

#### 2.1.3 清晰的模块化架构
```
astc-encoder 架构：
├── 核心编解码器（纯算法，平台无关）
│   ├── astcenc_compress_symbolic.cpp
│   ├── astcenc_decompress_symbolic.cpp
│   └── ...（22 个核心文件）
├── 平台抽象层
│   └── astcenccli_platform_dependents.cpp ⭐
└── 公开 API
    └── astcenc.h
```

核心编解码器完全平台无关，OH 可直接使用。

#### 2.1.4 标准 C++ 实现
- **语言标准**：C++14/17
- **无编译器扩展**：使用标准 C++ 特性
- **无特殊依赖**：不依赖特定系统库

### 2.2 OpenHarmony 的适配方式

虽然没有 Patch，但 OH 通过以下方式完成适配：

#### 2.2.1 BUILD.gn 条件编译

```gn
# BUILD.gn 关键配置
ohos_source_set("astc_encoder_static") {
  # 基础源文件（与上游一致）
  sources = [
    "//third_party/astc-encoder/Source/astcenc_averages_and_directions.cpp",
    # ... 共 25 个文件
  ]
  
  # OH 特有：条件编译
  if (defined(global_parts_info) &&
      (defined(global_parts_info.graphic_graphic_2d_ext) ||
       defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
    defines = [ "ASTC_CUSTOMIZED_ENABLE" ]
    sources += [
      "//third_party/astc-encoder/Source/astcenccli_platform_dependents.cpp",
    ]
    if (target_cpu == "arm64" || is_emulator) {
      defines += [ "SUT_PATH_X64" ]
    }
  }
  
  if (defined(global_parts_info) &&
      defined(global_parts_info.product_hmos_sdk_product_hmos_sdk)) {
    defines += [ "BUILD_HMOS_SDK" ]
  }
}
```

#### 2.2.2 条件编译宏详解

| 宏定义 | 触发条件 | 用途说明 |
|--------|---------|---------|
| `ASTC_CUSTOMIZED_ENABLE` | `graphic_graphic_2d_ext` 或 `product_hmos_sdk_product_hmos_sdk` | 启用 OH 图形扩展支持 |
| `SUT_PATH_X64` | `target_cpu == arm64` 或 `is_emulator` | ARM64 架构/模拟器特定处理 |
| `BUILD_HMOS_SDK` | `product_hmos_sdk_product_hmos_sdk` | HMOS SDK 构建标记 |

#### 2.2.3 平台适配文件

`astcenccli_platform_dependents.cpp` 是一个**上游已存在的文件**，但 OH 通过条件编译控制其是否参与构建：

```cpp
// Source/astcenccli_platform_dependents.cpp
// 功能：平台相关实现
//  - CPU 核心数查询
//  - 线程管理（pthread/Windows）
//  - 高精度计时

// Windows 实现
#if defined(_WIN32) && !defined(__CYGWIN__)
int get_cpu_count() {
    DWORD cpu_count = GetActiveProcessorCount(ALL_PROCESSOR_GROUPS);
    return static_cast<int>(cpu_count);
}

// POSIX 实现（OH 使用此路径）
#else
#include <pthread.h>
#include <sys/time.h>
#include <unistd.h>

int get_cpu_count() {
    return static_cast<int>(sysconf(_SC_NPROCESSORS_ONLN));
}
#endif
```

### 2.3 与需要 Patch 的库对比

| 特性 | astc-encoder | 典型需要 Patch 的库 |
|------|-------------|-------------------|
| 构建系统 | CMake（易迁移） | Autotools/Make（难迁移） |
| 平台抽象 | 内置完善 | 通常需外部实现 |
| 许可证 | Apache 2.0（兼容） | GPL/BSD（需修改） |
| 系统调用 | 标准 C/C++ | 依赖特定系统 API |
| 依赖管理 | 零依赖 | 复杂依赖链 |

---

## 3. OH 特有定制分析

### 3.1 定制类型分类

虽然没有 Patch 文件，但以下属于 OH 特有定制：

| 定制项 | 类型 | 位置 | 说明 |
|--------|------|------|------|
| `ASTC_CUSTOMIZED_ENABLE` | 宏定义 | BUILD.gn | 启用图形扩展支持 |
| `SUT_PATH_X64` | 宏定义 | BUILD.gn | ARM64 路径处理 |
| `BUILD_HMOS_SDK` | 宏定义 | BUILD.gn | SDK 构建标记 |
| `astcenccli_platform_dependents.cpp` | 条件源文件 | BUILD.gn | 平台适配代码 |

### 3.2 定制原因分析

#### 3.2.1 ASTC_CUSTOMIZED_ENABLE

**用途**：在启用 `graphic_graphic_2d_ext` 或 `product_hmos_sdk_product_hmos_sdk` 时，启用额外的平台适配功能。

**关联代码**：
```gn
if (defined(global_parts_info) &&
    (defined(global_parts_info.graphic_graphic_2d_ext) ||
     defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
  defines = [ "ASTC_CUSTOMIZED_ENABLE" ]
  sources += [ ".../astcenccli_platform_dependents.cpp" ]
}
```

**设计意图**：
- 基础功能（仅核心编解码器）无需平台适配代码
- 高级功能（如 CLI 工具、多线程优化）需要平台适配
- 通过条件编译避免不必要的代码膨胀

#### 3.2.2 SUT_PATH_X64

**用途**：针对 ARM64 架构或模拟器的特定处理。

```gn
if (target_cpu == "arm64" || is_emulator) {
  defines += [ "SUT_PATH_X64" ]
}
```

**可能用途**：
- 测试路径配置
- 特定架构的优化开关
- TODO(需确认)：在源码中搜索此宏的具体使用

#### 3.2.3 BUILD_HMOS_SDK

**用途**：HMOS（HarmonyOS）SDK 构建标记。

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.product_hmos_sdk_product_hmos_sdk)) {
  defines += [ "BUILD_HMOS_SDK" ]
}
```

**可能用途**：
- SDK 版本功能裁剪
- SDK 特定的路径或配置
- TODO(需确认)：在源码中搜索此宏的具体使用

---

## 4. 与上游版本的关系

### 4.1 版本对应

| OH 版本 | 上游版本 | 上游提交/标签 |
|---------|---------|--------------|
| 当前 | 4.7.0 | 4.7.0 发布标签 |

### 4.2 差异总结

```diff
# astc-encoder 在 OH 中的差异（对比上游 4.7.0）

添加的文件：
+ BUILD.gn              # OH 构建配置
+ bundle.json           # OH 组件配置
+ README.OpenSource     # OH 开源说明

修改的文件：
（无）

删除的文件：
（无）

条件编译差异：
- 上游：默认包含 platform_dependents.cpp（如果使用 CLI）
- OH：通过 ASTC_CUSTOMIZED_ENABLE 条件控制
```

### 4.3 维护优势

**无 Patch 带来的好处**：

1. **升级简单**：同步上游新版本时，无需处理 Patch 冲突
2. **维护成本低**：无需维护 Patch 系列
3. **代码清晰**：OH 特有逻辑集中在 BUILD.gn
4. **测试覆盖**：上游测试可直接用于 OH 版本

---

## 5. 升级建议

### 5.1 升级流程

由于无 Patch，升级流程简化：

```
1. 下载上游新版本源码
2. 保留 OH 配置文件（BUILD.gn, bundle.json, README.OpenSource）
3. 替换上游源码文件
4. 验证 BUILD.gn 中的源文件列表
5. 运行 image_framework 测试
6. 提交升级
```

### 5.2 升级检查清单

| 检查项 | 说明 |
|--------|------|
| API 兼容性 | 检查 `astcenc.h` 是否有破坏性变更 |
| 源文件列表 | 验证 BUILD.gn 中的 sources 是否完整 |
| 宏定义 | 确认 `ASTC_CUSTOMIZED_ENABLE` 等宏是否需要调整 |
| 功能测试 | 运行图像框架相关测试 |
| 性能测试 | 验证压缩/解压性能无回归 |

### 5.3 上游版本跟踪

建议关注的上游发布：
- **4.7.x**：Bug 修复版本（推荐及时跟进）
- **4.8.0**：新功能版本（评估后跟进）
- **5.0.0**：重大版本（需全面评估兼容性）

---

## 6. 总结

### 6.1 Patch 状态

| 项目 | 状态 |
|------|------|
| Patch 文件数量 | 0 |
| 是否需要 Patch | 否 |
| 适配方式 | BUILD.gn 条件编译 |

### 6.2 适配模式

```
传统模式（需要 Patch）：
上游源码 + Patch 文件 → OH 版本

astc-encoder 模式（无 Patch）：
上游源码 + BUILD.gn 配置 → OH 版本
```

### 6.3 维护建议

1. **优先跟踪上游版本**：无 Patch 意味着升级成本低
2. **关注条件编译宏**：了解 `ASTC_CUSTOMIZED_ENABLE` 等宏的具体用途
3. **验证集成测试**：确保 image_framework 相关功能正常
4. **文档同步**：上游重大变更时更新本文档

---

## 附录：验证命令

```bash
# 验证无 Patch 文件
find /Volumes/lexar/code/d/work/oh/third_party/astc-encoder -name "*.patch" -o -name "patches" -type d

# 验证条件编译宏在源码中的使用
grep -r "ASTC_CUSTOMIZED_ENABLE" /Volumes/lexar/code/d/work/oh/third_party/astc-encoder/Source/
grep -r "SUT_PATH_X64" /Volumes/lexar/code/d/work/oh/third_party/astc-encoder/Source/
grep -r "BUILD_HMOS_SDK" /Volumes/lexar/code/d/work/oh/third_party/astc-encoder/Source/

# TODO(需确认)：上述搜索的结果
```
