# 02_Patches - Patch 详细分析

## 概述

本文档详细记录 **OpenHarmony 对 Mesa3D 25.0.1 版本的所有 Patch**。这些 Patch 主要用于：
1. **CI 构建工具链适配** - 解决 Python 3 兼容性、依赖管理等问题
2. **测试框架适配** - dEQP/SKQP 在 Android/OH 环境下的运行支持
3. **构建配置优化** - 减少不必要的编译依赖

所有 Patch 文件位于: `.gitlab-ci/container/patches/`

---

## Patch 汇总表

| # | Patch 文件 | 修改模块 | 修改目的 | OH 关联 | 上游状态 |
|---|-----------|---------|---------|---------|---------|
| 1 | `build-skqp_fetch_gn.patch` | Skia GN 工具 | 更新 GN 获取方式，支持 Python 3 | CI 工具链 | 未上游 |
| 2 | `build-skqp_git-sync-deps.patch` | 依赖同步工具 | Python 3 兼容 + 增强依赖管理 | CI 工具链 | **强烈建议上游** |
| 3 | `build-skqp_is_clang.py.patch` | GN 构建配置 | 修复 GN 路径引用 | CI 构建 | **可上游** |
| 4 | `build-skqp_nima.patch` | DEPS 依赖源 | 切换 Nima 依赖源 (googlesource → GitHub) | 依赖可用性 | 已上游 |
| 5 | `build-skqp_gl.patch` | SKQP 报告 | 添加 kGL 后端统计支持 | 测试报告 | 未上游 |
| 6 | `build-skqp_BUILD.gn.patch` | BUILD.gn | 修复 skia_private config 可见性 | 构建配置 | 未上游 |
| 7 | `build-angle_deps_Make-more-sources-conditional.patch` | DEPS 依赖管理 | 按需加载依赖，减少下载量 | CI 性能 | **已上游** |
| 8 | `build-deqp-gl_Build-Don-t-build-Vulkan-utilities-for-GL-builds.patch` | dEQP GL 构建 | GL 构建时禁用 Vulkan 工具 | 构建优化 | **已上游** |
| 9 | `build-deqp-gl_Android-prints-to-stdout-instead-of-logcat.patch` | dEQP 输出 | 测试输出重定向到 stdout | CI 测试 | **有条件上游** |
| 10 | `build-deqp-gles_Allow-running-on-Android-from-the-command-line.patch` | dEQP GLES Android | 允许命令行运行 Android 测试 | CI 测试 | **有条件上游** |
| 11 | `build-deqp-gles_Android-prints-to-stdout-instead-of-logcat.patch` | dEQP 输出 | 测试输出重定向到 stdout | CI 测试 | **有条件上游** |
| 12 | `build-deqp-gl_Allow-running-on-Android-from-the-command-line.patch` | dEQP GL Android | 允许命令行运行 Android 测试 | CI 测试 | **有条件上游** |

---

## Patch 详细分析

### 类别 1: SKQP 构建工具链适配

---

#### Patch 1: build-skqp_fetch_gn.patch

**修改文件**: `bin/fetch-gn`

**原始问题**:
- 原脚本从 `chromium-gn.storage-download.googleapis.com` 下载 GN
- 使用固定的 SHA1 校验
- 仅支持 Linux/Mac/Windows
- 只支持 Python 2

**修改内容**:
```python
# 1. 改为从 Chrome 基础设施包下载
url = 'https://chrome-infra-packages.appspot.com/dl/gn/gn/{}-{}/+/git_revision:{}'

# 2. 支持 Python 2/3 兼容
if sys.version_info[0] < 3:
    from urllib2 import urlopen
else:
    from urllib.request import urlopen

# 3. 支持更多 CPU 架构
cpu = {'amd64': 'amd64', 'arm64': 'arm64', 'x86_64': 'amd64', 'aarch64': 'arm64'}

# 4. 移除 SHA1 硬编码校验
```

**OH 价值**: 确保 CI 构建工具链的 Python 3 兼容性

**上游化建议**: **低优先级**。这是 Mesa CI 特定的 GN 获取方式，上游 Skia 使用不同的分发方式。

---

#### Patch 2: build-skqp_git-sync-deps.patch ⭐ (强烈建议上游)

**修改文件**: `tools/git-sync-deps`

**原始问题**:
1. Python 2 语法 (`execfile`, `basestring`, `print` 语句)
2. 仅支持 SHA1 格式的 git 引用
3. Windows 上找不到 `git` 命令
4. 字节/字符串解码问题

**修改内容**:
```python
# Python 2 → 3 迁移
execfile(f)  →  exec(open(f).read())
basestring   →  str
print x      →  print(x)

# Windows 支持增强
if sys.platform == 'win32':
    git_cmd = ['git.bat'] + git_cmd[1:]

# Git 引用类型扩展
commit = subprocess.check_output(['git', 'rev-parse', dep['revision']]).decode().strip()

# 移除 SHA1 强制校验，支持任意 commit/branch/tag
```

**OH 价值**: 确保 CI 依赖同步工具的 Python 3 兼容性

**上游化建议**: **强烈推荐**。Python 3 迁移是普遍需求，Skia 社区会欢迎此改进。

---

#### Patch 3: build-skqp_is_clang.py.patch

**修改文件**: `gn/BUILDCONFIG.gn`

**原始问题**:
```
# 原代码
exec_script("gn/is_clang.py", ...)
```

相对路径解析失败。

**修改内容**:
```python
# 改为绝对路径引用
exec_script("//gn/is_clang.py", ...)
```

**OH 价值**: 修复 GN 构建路径解析问题

**上游化建议**: **可以上游**。这是标准的 GN 路径规范，简单的 bug fix。

---

#### Patch 4: build-skqp_nima.patch

**修改文件**: `DEPS`

**原始问题**:
`Nima-Cpp` 和 `Nima-Math-Cpp` 从 `skia.googlesource.com` 无法访问（已下线/移除）。

**修改内容**:
```python
# 修改前
'third_party/nima':
    Var('chromium_git') + '/external/github.com/nima/nima.git',

# 修改后
'third_party/nima':
    'https://github.com/nima/Nima-Cpp',
```

**OH 价值**: 解决依赖源可用性问题

**上游化建议**: **无需**。此 Patch 模拟了 Skia 仓库的 revert 操作，上游已修复。

---

#### Patch 5: build-skqp_gl.patch

**修改文件**: `tools/skqp/src/skqp.cpp`

**原始问题**:
SKQP 报告生成器不统计 OpenGL (非 ES) 后端的结果。

**修改内容**:
```cpp
// 新增 kGL 后端统计
int glErrorCount = 0, glesErrorCount = 0, vkErrorCount = 0, gl = 0, gles = 0, vk = 0;

switch (run.fBackend) {
    case SkQP::SkiaBackend::kGL: ++gl; break;
    case SkQP::SkiaBackend::kGLES: ++gles; break;
    case SkQP::SkiaBackend::kVulkan: ++vk; break;
}

// 报告输出
write(&htmOut, SkStringPrintf("<p>gl errors: %d (of %d)</br>\n"
                              "gles errors: %d (of %d)</br>\n"
                              "vk errors: %d (of %d)</p>\n",
                              glErrorCount, gl, glesErrorCount, gles,
                              vkErrorCount, vk));
```

**OH 价值**: 增强 SKQP 测试报告的完整性

**上游化建议**: **可考虑上游**。对桌面端 OpenGL 测试也有价值。

---

#### Patch 6: build-skqp_BUILD.gn.patch

**修改文件**: `BUILD.gn`

**原始问题**:
```gn
config("skia_private") {
    visibility = [ ":*" ]  # 问题：: 语法在某些 GN 版本不解析
```

**修改内容**:
```gn
config("skia_private") {
    visibility = [ "*" ]   # 改为标准通配符
```

**OH 价值**: 修复 GN 可见性配置

**上游化建议**: **可上游**。修复 GN 兼容性。

---

### 类别 2: DEPS 依赖管理优化

---

#### Patch 7: build-angle_deps_Make-more-sources-conditional.patch ⭐ (已上游)

**修改文件**: `DEPS`

**原始问题**:
构建时无条件下载所有依赖，包括大量不需要的 LLVM 副本，耗时过长。

**修改内容**:
```python
# 新增条件变量
vars = {
    'angle_enable_cl': True,
    'angle_enable_vulkan': True,
    'angle_enable_wgpu': True,
    'build_angle_deqp_tests': True,
    'build_angle_perftests': True,
    'build_with_swiftshader': True,
    # ...
}

# 依赖改为条件加载
'third_party/clspv/src': {
    'condition': 'angle_enable_cl and angle_enable_vulkan and not build_with_chromium',
},

'third_party/llvm/src': {
    'condition': '(build_with_swiftshader or (angle_enable_cl and angle_enable_vulkan)) and not build_with_chromium',
},
```

**优化效果**:
- 按需下载依赖，减少 CI 时间
- 避免下载不需要的 LLVM (节省数 GB 带宽)

**上游化建议**: **已上游**。通过 chromium-review.googlesource.com 提交。

---

### 类别 3: dEQP 测试框架适配

---

#### Patch 8: build-deqp-gl_Build-Don-t-build-Vulkan-utilities-for-GL-builds.patch ⭐ (已上游)

**修改文件**:
- `framework/platform/CMakeLists.txt`
- `framework/platform/android/tcuAndroidPlatform.cpp`
- `framework/platform/lnx/tcuLnxPlatform.cpp`
- `framework/platform/surfaceless/tcuSurfacelessPlatform.cpp`

**原始问题**:
即使只构建 GL 测试，也会编译大量 Vulkan 平台代码。

**修改内容**:
```cmake
# CMakeLists.txt: 移除 Vulkan 相关文件
if (DE_OS_IS_WIN32)
    list(REMOVE_ITEM TCUTIL_PLATFORM_SRCS
        win32/tcuWin32VulkanPlatform.hpp
        win32/tcuWin32VulkanPlatform.cpp)
elseif (DE_OS_IS_UNIX AND ...)
    list(REMOVE_ITEM TCUTIL_PLATFORM_SRCS
        lnx/tcuLnxVulkanPlatform.hpp
        lnx/tcuLnxVulkanPlatform.cpp)
```

```cpp
// 移除 Vulkan 平台类
class VulkanPlatform { /* 删除 */ };

// 移除 vkutil 依赖
target_link_libraries(tcutil-platform vkutil)  # 删除此行
```

**OH 价值**: 减少不必要的编译时间和产物大小

**上游化建议**: **已上游**

---

#### Patch 9 & 11: build-deqp-*_Android-prints-to-stdout-instead-of-logcat.patch

**修改文件**: `framework/qphelper/qpDebugOut.c`

**原始问题**:
dEQP 在 Android 上使用 `__android_log_print` 输出到 logcat，CI 环境难以捕获输出。

**修改内容**:
```c
// 修改前
#if (DE_OS == DE_OS_ANDROID)
    __android_log_print(ANDROID_LOG_INFO, "dEQP", "%s", str);

// 修改后
#if (0)  // 强制使用 stdout
    printf("%s", str);
```

**OH 价值**: 允许 CI 捕获测试输出用于自动化验证

**上游化建议**: **有条件上游**。建议添加 CMake 选项控制，而非硬编码。

---

#### Patch 10 & 12: build-deqp-*_Allow-running-on-Android-from-the-command-line.patch

**修改文件**:
- `CMakeLists.txt`
- `framework/platform/android/tcuAndroidNativeActivity.cpp`
- `framework/platform/android/tcuAndroidPlatform.cpp`

**原始问题**:
dEQP 在 Android 上强制构建为共享库 (.so) 并通过 JNI/Activity 调用，无法直接命令行运行。

**修改内容**:
```cmake
# CMakeLists.txt: 移除 Android 特殊处理
if (DE_OS_IS_ANDROID)
    add_executable(tcutil ${TCUTIL_PLATFORM_SRCS})
else()
    add_library(tcutil SHARED ${TCUTIL_PLATFORM_SRCS})
endif()
```

```cpp
// tcuAndroidPlatform.cpp: 添加 activity 空值检查
class Platform : public tcu::Android::Platform {
    vk::Library *createLibrary(const char *libraryPath) const {
        if (!m_activity) {
            TCU_THROW(NotSupportedError,
                "Activity not available, cannot create Vulkan library");
        }
        return new VulkanLibrary(libraryPath);
    }
};
```

**OH 价值**: 允许在没有 Activity 的环境中运行 GPU 测试

**上游化建议**: **有条件上游**。建议添加 `DEQP_ANDROID_CMDLINE` CMake 选项。

---

## Patch 分类统计

```
┌─────────────────────────────────┬──────────┐
│ 分类                            │ Patch 数 │
├─────────────────────────────────┼──────────┤
│ SKQP 构建工具链适配              │    4     │
│   - GN 获取脚本                  │    1     │
│   - 依赖同步工具 (Python 3)      │    1     │
│   - 构建配置修复                 │    2     │
├─────────────────────────────────┼──────────┤
│ DEPS 依赖管理优化                │    2     │
│   - 条件加载依赖                 │    1     │
│   - 依赖源切换                   │    1     │
├─────────────────────────────────┼──────────┤
│ dEQP Android 测试适配            │    4     │
│   - 命令行运行支持 (GL/GLES)     │    2     │
│   - stdout 输出 (GL/GLES)       │    2     │
├─────────────────────────────────┼──────────┤
│ 构建配置修复                     │    2     │
│   - skia_private 可见性          │    1     │
│   - SKQP 报告增强                │    1     │
└─────────────────────────────────┴──────────┘
```

---

## Patch 上游化建议汇总

| Patch | 上游化优先级 | 建议 |
|-------|------------|------|
| build-skqp_fetch_gn.patch | 低 | Mesa CI 特定需求 |
| **build-skqp_git-sync-deps.patch** | **高** | Python 3 兼容，社区欢迎 |
| build-skqp_is_clang.py.patch | 高 | 标准 GN 规范，简单的 fix |
| build-skqp_nima.patch | 无需 | 上游已修复 |
| build-skqp_gl.patch | 中 | 增强测试报告 |
| build-skqp_BUILD.gn.patch | 中 | GN 兼容性修复 |
| build-angle_deps_Make-more-sources-conditional.patch | 已上游 | - |
| build-deqp-gl_Build-Don-t-build-Vulkan-utilities-for-GL-builds.patch | 已上游 | - |
| build-deqp-gl_Android-prints-to-stdout-instead-of-logcat.patch | 中 | 建议添加 CMake 选项 |
| build-deqp-gles_Allow-running-on-Android-from-the-command-line.patch | 中 | 建议添加 CMake 选项 |
| build-deqp-gles_Android-prints-to-stdout-instead-of-logcat.patch | 中 | 建议添加 CMake 选项 |
| build-deqp-gl_Allow-running-on-Android-from-the-command-line.patch | 中 | 建议添加 CMake 选项 |

---

## 升级注意事项

### 上游版本升级时

1. **检查 Patch 冲突**:
   - 使用 `git apply --check` 验证每个 Patch
   - 关注上游已接受的 Patch (如 #7, #8)

2. **回归测试**:
   - 运行 dEQP 测试验证框架
   - 验证 SKQP 报告生成

3. **依赖更新**:
   - 检查 DEPS 文件变更
   - 验证条件加载逻辑

### 手动维护 Patch

```bash
# 查看 Patch 应用状态
git apply --check .gitlab-ci/container/patches/*.patch

# 应用所有 Patch
git am .gitlab-ci/container/patches/*.patch

# 撤销所有 Patch
git am --abort  # 或 git reset --hard
```
