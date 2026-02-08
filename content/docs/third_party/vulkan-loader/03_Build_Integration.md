# 03 - OpenHarmony 构建适配

## 3.1 BUILD.gn 结构概述

### 文件位置
- **主 BUILD.gn**: `/third_party/vulkan-loader/BUILD.gn`
- **调试 Layer BUILD.gn**: `/third_party/vulkan-loader/openharmony/debug_trace_layer/BUILD.gn`
- **测试 BUILD.gn**: `/third_party/vulkan-loader/openharmony/test/BUILD.gn`

### 构建目标

| 目标名 | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `vulkan_loader` | ohos_shared_library | libvulkan.so | 主 Vulkan Loader 库 |
| `vulkan_loader_test` | group | - | 测试组入口 |
| `vulkan_internal_config` | config | - | 内部编译配置 |
| `vulkan_loader_config` | config | - | 公共编译配置 |
| `asm_offset` | static_library | libasm_offset.a | 汇编偏移量（仅 ARM64/x86_64）|
| `gen_defines` | action | gen_defines.asm | 生成汇编定义 |

---

## 3.2 关键编译选项

### 编译定义 (defines)

```gn
# 内部配置 (vulkan_internal_config)
defines = [
    "VK_ENABLE_BETA_EXTENSIONS",           # 启用 Vulkan Beta 扩展
]

defines += [
    "SYSCONFDIR=\"/system/etc:/vendor/etc\"",  # 配置文件路径
    "VK_USE_PLATFORM_OHOS",                # 启用 OHOS 平台支持 ⭐
]

# 主目标定义 (vulkan_loader)
defines = [
    "HOOK_ENABLE",                         # 启用 Hook 功能
]

# 架构相关定义
if (support_unknown_function_handling) {
    defines += [ "UNKNOWN_FUNCTIONS_SUPPORTED=1" ]
}
```

### 关键定义说明

| 定义 | 影响范围 | 说明 |
|------|---------|------|
| `VK_USE_PLATFORM_OHOS` | 全模块 | 启用所有 OHOS 特有代码路径 |
| `VK_ENABLE_BETA_EXTENSIONS` | 全模块 | 启用 Vulkan Beta 扩展 |
| `HOOK_ENABLE` | 主库 | 启用 API Hook 功能 |
| `UNKNOWN_FUNCTIONS_SUPPORTED` | ARM64/x86_64 | 支持未知 Vulkan 函数 |
| `SYSCONFDIR` | 驱动搜索 | 驱动配置文件搜索路径 |

### 编译器标志 (cflags)

```gn
cflags = [
    "-Wno-conversion",       # 忽略隐式转换警告
    "-Wno-extra-semi",       # 忽略多余分号警告
    "-Wno-sign-compare",     # 忽略符号比较警告
    "-Wno-unreachable-code", # 忽略不可达代码警告
    "-Wno-unused-function",  # 忽略未使用函数警告
    "-Wno-unused-variable",  # 忽略未使用变量警告
    "-fPIC",                 # 生成位置无关代码
]

cflags_cc = [ "-std=c++17" ]  # C++17 标准

ldflags = [ "-Wl,-Bsymbolic" ]  # 符号绑定选项
```

### 安全特性

```gn
ohos_shared_library("vulkan_loader") {
    branch_protector_ret = "pac_ret"  # 启用指针认证（PAC-RET）⭐
    # ...
}
```

**说明**：`pac_ret` (Pointer Authentication Code - Return) 是 ARM64 的安全特性，防止返回地址被篡改。

---

## 3.3 包含路径

```gn
config("vulkan_loader_config") {
    include_dirs = [
        "loader/generated",    # 生成的代码
        "loader",              # Loader 核心代码
        "openharmony",         # OH 特有代码 ⭐
    ]
    defines = [ "LOADER_USE_UNSAFE_FILE_SEARCH=1" ]
}
```

**OH 特有路径**：`openharmony/` 目录包含 Bundle 管理器、HiLog 适配等 OH 特有代码。

---

## 3.4 依赖关系

### 外部依赖 (external_deps)

```gn
external_deps = [
    "bundle_framework:appexecfwk_base",    # Bundle 管理基础
    "bundle_framework:appexecfwk_core",    # Bundle 管理核心
    "c_utils:utils",                       # C 工具库
    "hilog:libhilog",                      # HiLog 日志 ⭐
    "init:libbegetutil",                   # 启动工具库 (syspara)
    "ipc:ipc_core",                        # IPC 核心
    "samgr:samgr_proxy",                   # 系统服务管理代理
    "vulkan-headers:vulkan_headers",       # Vulkan 头文件
]
```

### 依赖说明

| 依赖 | 用途 | OH 特有 |
|------|------|---------|
| bundle_framework | Bundle 管理器集成 | ✅ 是 |
| hilog | 日志系统 | ✅ 是 |
| libbegetutil | 系统参数 (syspara) | ✅ 是 |
| samgr | 系统服务连接 | ✅ 是 |
| vulkan-headers | Vulkan API 定义 | ⚪ 第三方 |

---

## 3.5 源文件列表

### 核心 Loader 源文件

| 文件 | 说明 |
|------|------|
| `loader/allocation.c/h` | 内存分配管理 |
| `loader/cJSON.c/h` | JSON 解析 |
| `loader/debug_utils.c/h` | 调试工具 |
| `loader/extension_manual.c/h` | 手动扩展处理 |
| `loader/gpa_helper.c/h` | GetProcAddr 辅助 |
| `loader/loader.c/h` | 核心加载逻辑 |
| `loader/loader_common.h` | 公共定义 |
| `loader/loader_environment.c/h` | 环境变量处理 ⭐ |
| `loader/loader_json.c/h` | JSON 清单解析 |
| `loader/log.c/h` | 日志系统 ⭐ |
| `loader/settings.c/h` | 设置管理 |
| `loader/terminator.c` | 终止器函数 |
| `loader/trampoline.c` | Trampoline 函数 |
| `loader/unknown_function_handling.c/h` | 未知函数处理 |
| `loader/wsi.c/h` | WSI 窗口系统集成 ⭐ |

### 生成代码

| 文件 | 说明 |
|------|------|
| `loader/generated/vk_layer_dispatch_table.h` | Layer 分发表 |
| `loader/generated/vk_loader_extensions.h` | Loader 扩展 |
| `loader/generated/vk_object_types.h` | 对象类型定义 |
| `loader/generated/vk_loader_extensions.c` | 扩展实现（大量）|

### OH 特有源文件

| 文件 | 说明 |
|------|------|
| `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.cpp` | Bundle 管理器帮助类 ⭐ |
| `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.h` | 头文件 |
| `openharmony/loader_hilog.h` | HiLog 日志适配 ⭐ |

### 架构特定汇编（ARM64/x86_64）

| 文件 | 说明 |
|------|------|
| `loader/asm_offset.c` | 汇编偏移量计算 |
| `loader/unknown_ext_chain_gas_aarch.S` | ARM64 未知扩展链 |
| `loader/unknown_ext_chain_gas_x86.S` | x86_64 未知扩展链 |

---

## 3.6 与上游 CMake 的差异

### 构建系统对比

| 特性 | 上游 CMake | OH BUILD.gn |
|------|-----------|-------------|
| **构建工具** | CMake | GN + Ninja |
| **目标类型** | SHARED_LIBRARY | ohos_shared_library |
| **平台检测** | CMake 变量 | GN 条件判断 |
| **依赖管理** | find_package | external_deps |
| **安装路径** | CMAKE_INSTALL_PREFIX | 自动（系统目录）|
| **测试** | CTest | GN test group |

### 上游 CMakeLists.txt 关键配置

```cmake
# 上游 CMakeLists.txt 中...
option(BUILD_TESTS "Build tests" ON)
option(BUILD_WSI_XCB_SUPPORT "Build XCB WSI support" ON)
option(BUILD_WSI_XLIB_SUPPORT "Build Xlib WSI support" ON)
option(BUILD_WSI_WAYLAND_SUPPORT "Build Wayland WSI support" ON)
```

**OH BUILD.gn 替代**：
- 不使用 CMake 选项，直接在 `defines` 中配置
- 不支持 XCB/Xlib/Wayland（OHOS 不使用这些窗口系统）
- 启用 `VK_USE_PLATFORM_OHOS` 替代

### 源文件选择差异

**上游 CMake**（条件编译）：
```cmake
if(UNIX)
    target_sources(vulkan PRIVATE loader/loader.c)
endif()
if(WIN32)
    target_sources(vulkan PRIVATE loader/loader_windows.c)
endif()
```

**OH BUILD.gn**（显式列表）：
```gn
ohos_shared_library("vulkan_loader") {
    sources = [
        "loader/loader.c",  # 直接列出所有文件
        # ...
    ]
}
```

---

## 3.7 特殊构建处理

### 汇编偏移量生成

**仅在 ARM64/x86_64 平台**：

```gn
if (defined(ar_path) && ar_path != "" && !is_win &&
    (current_cpu == "arm64" || current_cpu == "x86_64")) {
    
    support_unknown_function_handling = true
    
    # 1. 编译 asm_offset.c 为汇编
    static_library("asm_offset") {
        sources = [ "loader/asm_offset.c" ]
        cflags = [ "-S" ]  # 输出汇编而非目标文件
    }
    
    # 2. 解析汇编提取偏移量
    action("gen_defines") {
        script = "scripts/parse_asm_values.py"
        # 生成 gen_defines.asm
    }
}
```

**目的**：动态计算结构体偏移量，用于汇编代码访问 C 结构体。

### 测试构建

```gn
group("vulkan_loader_test") {
    testonly = true
    public_deps = [ "openharmony/test:test" ]
}
```

**特点**：
- `testonly = true` 确保测试代码不会进入正式版本
- 测试依赖在测试组中声明

---

## 3.8 构建命令示例

### 单独编译 vulkan-loader

```bash
./build.sh --product-name rk3568 --ccache --build-target vulkan_loader
```

### 通过 graphic_2d 编译

```bash
./build.sh --product-name rk3568 --ccache --build-target graphic_2d
```

### 编译测试

```bash
./build.sh --product-name rk3568 --ccache --build-target vulkan_loader_test
```

### 输出位置

```
out/rk3568/graphic/graphic_2d/libvulkan.so
out/rk3568/third_party/vulkan-loader/libvulkan.so
```

---

## 3.9 配置片段示例

### 完整的 vulkan_loader 目标配置

```gn
ohos_shared_library("vulkan_loader") {
    # 安全配置
    branch_protector_ret = "pac_ret"
    
    # 编译定义
    defines = [ "HOOK_ENABLE" ]
    
    # API 级别
    innerapi_tags = [ "llndk" ]  # 提供给应用层
    
    # 源文件（40+ 个文件）
    sources = [
        "loader/loader.c",
        "loader/wsi.c",
        "openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.cpp",
        # ...
    ]
    
    # 配置
    configs = [ ":vulkan_internal_config" ]
    public_configs = [ ":vulkan_loader_config" ]
    
    # 依赖
    external_deps = [
        "bundle_framework:appexecfwk_core",
        "hilog:libhilog",
        "vulkan-headers:vulkan_headers",
        # ...
    ]
    
    # 架构特定源文件
    if (support_unknown_function_handling) {
        if (current_cpu == "arm64") {
            sources += [ "loader/unknown_ext_chain_gas_aarch.S" ]
        } else if (current_cpu == "x86_64") {
            sources += [ "loader/unknown_ext_chain_gas_x86.S" ]
        }
        defines += [ "UNKNOWN_FUNCTIONS_SUPPORTED=1" ]
    }
    
    # 输出名
    output_name = "vulkan"
    output_extension = "so"
    
    # 部件信息
    part_name = "vulkan-loader"
    subsystem_name = "thirdparty"
    license_file = "//third_party/vulkan-loader/LICENSE.txt"
}
```

---

## 3.10 升级注意事项

### BUILD.gn 升级检查清单

升级上游版本时，检查 BUILD.gn 的以下部分：

- [ ] **新增源文件**：上游新增的 `.c/.h` 文件需加入 `sources`
- [ ] **新增定义**：上游新增的宏定义需加入 `defines`
- [ ] **生成代码同步**：`loader/generated/` 目录需重新生成
- [ ] **依赖变更**：上游依赖变化需同步到 `external_deps`
- [ ] **包含路径**：上游新增的头文件搜索路径需加入 `include_dirs`

### 典型升级问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 编译错误：未定义符号 | 新增源文件未加入 BUILD.gn | 将新文件加入 sources |
| 运行时崩溃 | 生成代码未更新 | 重新运行代码生成脚本 |
| OH 特有功能失效 | `VK_USE_PLATFORM_OHOS` 未定义 | 检查 defines |
| 日志不输出 | HiLog 依赖缺失 | 检查 external_deps |

---

*文档版本：v1.0*
*最后更新：2026-02-07*
