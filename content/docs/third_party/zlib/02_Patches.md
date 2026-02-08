# Patch 详细分析

本文档详细分析 OpenHarmony 对 zlib 的所有 Patch，包括修改内容、修改目的和升级建议。

## Patch 清单

| Patch 文件 | 修改文件 | 修改类型 | 修改目的 |
|-----------|---------|---------|---------|
| [huawei_zlib_CMakeList.patch](#huawei_zlib_cmakelistpatch) | CMakeLists.txt | 构建适配 | 禁用 MINGW 代码和测试二进制文件 |

---

## huawei_zlib_CMakeList.patch

### 基本信息

```yaml
文件: huawei_zlib_CMakeList.patch
修改目标: CMakeLists.txt
修改行数: ~50 行
修改类型: 代码注释/禁用
适配目的: 适配 OpenHarmony 构建系统
```

### 修改内容详解

#### 修改 1: 禁用 MINGW DLL 资源代码

**原始代码位置**: CMakeLists.txt 第 167-187 行

```cmake
# 原始代码
if(MINGW)
    # This gets us DLL resource information when compiling on MinGW.
    if(NOT CMAKE_RC_COMPILER)
        set(CMAKE_RC_COMPILER windres.exe)
    endif()

    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
                       COMMAND ${CMAKE_RC_COMPILER}
                            -D GCC_WINDRES
                            -I ${CMAKE_CURRENT_SOURCE_DIR}
                            -I ${CMAKE_CURRENT_BINARY_DIR}
                            -o ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
                            -i ${CMAKE_CURRENT_SOURCE_DIR}/win32/zlib1.rc)
    set(ZLIB_DLL_SRCS ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj)
endif(MINGW)
```

**修改后代码**:

```cmake
#if(MINGW)
    # This gets us DLL resource information when compiling on MinGW.
#    if(NOT CMAKE_RC_COMPILER)
#        set(CMAKE_RC_COMPILER windres.exe)
#    endif()
#
#    add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
#                       COMMAND ${CMAKE_RC_COMPILER}
#                            -D GCC_WINDRES
#                            -I ${CMAKE_CURRENT_SOURCE_DIR}
#                            -I ${CMAKE_CURRENT_BINARY_DIR}
#                            -o ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
#                            -i ${CMAKE_CURRENT_SOURCE_DIR}/win32/zlib1.rc)
#    set(ZLIB_DLL_SRCS ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj)
#endif(MINGW)
```

**修改分析**:

| 属性 | 说明 |
|-----|-----|
| **原始问题** | MINGW 代码使用 `windres.exe` 编译 Windows 资源文件 |
| **OH 问题** | OpenHarmony 不使用 MINGW 交叉编译，此代码在 OH 构建中无效 |
| **解决方案** | 注释掉整个 MINGW 条件块 |
| **影响范围** | 仅影响 CMake 构建路径，不影响 GN 构建 |

#### 修改 2: 禁用测试可执行文件

**原始代码位置**: CMakeLists.txt 第 230-275 行

```cmake
# 原始代码 - 示例程序
add_executable(example test/example.c)
target_link_libraries(example zlib)
add_test(example example)

add_executable(minigzip test/minigzip.c)
target_link_libraries(minigzip zlib)

# 原始代码 - 64位示例程序
if(HAVE_OFF64_T)
    add_executable(example64 test/example.c)
    target_link_libraries(example64 zlib)
    set_target_properties(example64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
    add_test(example64 example64)

    add_executable(minigzip64 test/minigzip.c)
    target_link_libraries(minigzip64 zlib)
    set_target_properties(minigzip64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
endif()
```

**修改后代码**:

```cmake
#add_executable(example test/example.c)
#target_link_libraries(example zlib)
#add_test(example example)

#add_executable(minigzip test/minigzip.c)
#target_link_libraries(minigzip zlib)

#if(HAVE_OFF64_T)
#    add_executable(example64 test/example.c)
#    target_link_libraries(example64 zlib)
#    set_target_properties(example64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
#    add_test(example64 example64)

#   add_executable(minigzip64 test/minigzip.c)
#    target_link_libraries(minigzip64 zlib)
#    set_target_properties(minigzip64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
#endif()
```

**修改分析**:

| 属性 | 说明 |
|-----|-----|
| **原始问题** | 构建 `example`, `minigzip`, `example64`, `minigzip64` 测试程序 |
| **OH 问题** | 1. OH 使用自己的测试框架 (XTS)<br>2. 不需要构建示例二进制文件<br>3. 减少构建产物体积 |
| **解决方案** | 注释掉所有测试可执行文件构建 |
| **影响范围** | 仅影响 CMake 测试构建 |

### Patch 应用方式

```bash
# prepare.sh 脚本内容
#!/bin/bash
# Copyright (c) Huawei Technologies Co., Ltd. 2020. All rights reserved.

git checkout CMakeLists.txt
git apply huawei_zlib_CMakeList.patch
```

**应用时机**:
- 首次克隆仓库后运行 `prepare.sh`
- 升级上游版本后重新应用

### 修改目的总结

| 修改项 | OH 需求 | 实现方式 |
|-------|--------|---------|
| 禁用 MINGW 代码 | OH 不使用 MINGW 编译 | 注释条件块 |
| 禁用测试程序 | OH 使用 XTS 测试框架 | 注释 add_executable |

### OH 价值

1. **构建精简**: 避免构建不必要的测试二进制文件
2. **跨平台**: 移除 Windows 特定的编译逻辑
3. **标准化**: 统一使用 GN 构建系统

### 回归风险评估

| 风险项 | 等级 | 说明 |
|-------|-----|-----|
| **API 兼容性** | 无风险 | 不修改任何 API |
| **功能完整性** | 无风险 | 不修改核心算法 |
| **升级复杂度** | 低 | Patch 可自动重应用 |
| **构建系统** | 低 | 仅影响 CMake，OH 使用 GN |

### 升级建议

1. **自动重应用**:
   ```bash
   ./prepare.sh  # 自动重置并应用 Patch
   ```

2. **冲突解决** (如果出现):
   - Patch 仅涉及注释代码，通常无冲突
   - 如 CMakeLists.txt 大幅变更，可手动重新注释对应代码块

3. **上游推送建议**:
   - 此 Patch 为 OH 特有需求，**不建议推向上游**
   - 上游需要保持 CMakeLists.txt 的完整功能

---

## Patch 分类总结

| 分类 | Patch 数量 | 说明 |
|-----|-----------|-----|
| **Bugfix** | 0 | 无 bug 修复类 Patch |
| **Feature** | 0 | 无功能增强类 Patch |
| **OH 构建适配** | 1 | huawei_zlib_CMakeList.patch |
| **性能优化** | 0 | 性能优化通过 BUILD.gn 配置实现 |

## 与其他 OH 第三方库对比

| 库名 | Patch 数量 | 主要 Patch 类型 |
|-----|-----------|----------------|
| zlib | 1 | 构建适配 |
| curl | 较多 | 功能增强、Bugfix |
| openssl | 多 | 安全修复、功能适配 |
| libpng | 少 | 构建适配 |

**结论**: zlib 的 OH 适配工作量较小，主要集中在构建系统层面，无需修改核心算法代码。

---

## 相关文件

- `huawei_zlib_CMakeList.patch` - Patch 文件本身
- `prepare.sh` - Patch 应用脚本
- `BUILD.gn` - OH 主构建配置（替代 CMakeLists.txt）
