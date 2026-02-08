# 03 - OpenHarmony 构建适配

## 3.1 BUILD.gn 结构说明

### 文件位置
```
third_party/popt/BUILD.gn
```

### 完整配置
```gn
# Copyright (c) 2022-2025 Huawei Device Co., Ltd.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
import("//build/lite/config/component/lite_component.gni")
import("//build/ohos.gni")

config("popt_config") {
  include_dirs = [ "//third_party/popt/src" ]
}

popt_sources = [
  "./src/popt.c",
  "./src/popthelp.c",
  "./src/poptint.c",
  "./src/poptparse.c",
]

ohos_static_library("popt_static") {
  sources = popt_sources
  output_name = "popt"
  public_configs = [ ":popt_config" ]
  defines = [
    "HAVE_CONFIG_H",
    "_GNU_SOURCE",
    "_REENTRANT",
  ]
  cflags_c = [
    "-Wall",
    "-Os",
    "-g",
    "-W",
    "-ffunction-sections",
    "-fdata-sections",
    "-Wno-unused-const-variable",
    "-Wno-unused-parameter",
    "-Wno-gnu-alignof-expression",
  ]
  ldflags = [
    "-Wl",
    "--gc-sections",
  ]
}
```

## 3.2 构建配置详解

### 3.2.1 源文件选择

#### 包含的文件 (4 个)
| 文件 | 功能 | 大小 (约) |
|------|------|----------|
| `popt.c` | 核心解析逻辑，主 API 实现 | 44KB |
| `popthelp.c` | 帮助信息生成 (`--help`, `--usage`) | 24KB |
| `poptint.c` | 内部辅助函数 | 4KB |
| `poptparse.c` | 参数字符串解析 | 5KB |

#### 排除的文件
| 文件 | 功能 | 排除原因 |
|------|------|---------|
| `lookup3.c` | hash 函数 | gptfdisk 不需要 |
| `poptconfig.c` | 配置文件解析 | 减少攻击面，gptfdisk 不需要配置文件支持 |

### 3.2.2 编译选项分析

#### 预处理器定义
```gn
defines = [
  "HAVE_CONFIG_H",    # 使用 config.h 中的配置
  "_GNU_SOURCE",      # 启用 GNU 扩展
  "_REENTRANT",       # 启用可重入函数
]
```

| 定义 | 作用 |
|------|------|
| `HAVE_CONFIG_H` | 告知源码使用 `config.h` 中的宏定义 |
| `_GNU_SOURCE` | 启用 GNU C 扩展 (如 `stpcpy` 等) |
| `_REENTRANT` | 启用线程安全函数 |

#### 编译器标志
```gn
cflags_c = [
  "-Wall",                          # 启用大多数警告
  "-Os",                            # 优化代码大小
  "-g",                             # 保留调试信息
  "-W",                             # 额外警告
  "-ffunction-sections",            # 每个函数单独 section
  "-fdata-sections",                # 每个数据单独 section
  "-Wno-unused-const-variable",     # 禁用特定警告
  "-Wno-unused-parameter",
  "-Wno-gnu-alignof-expression",
]
```

| 标志 | 说明 |
|------|------|
| `-Os` | 大小优化，适合嵌入式场景 |
| `-ffunction-sections` | 配合 `--gc-sections` 去除未使用函数 |
| `-fdata-sections` | 配合 `--gc-sections` 去除未使用数据 |
| `-Wno-*` | 禁用 popt 代码触发的非关键警告 |

#### 链接器标志
```gn
ldflags = [
  "-Wl",
  "--gc-sections",    # 去除未使用的 section
]
```

**效果**: 大幅减小最终二进制大小，只保留实际使用的函数。

### 3.2.3 公共配置
```gn
config("popt_config") {
  include_dirs = [ "//third_party/popt/src" ]
}
```

依赖者通过 `public_configs` 自动获取头文件路径，无需手动指定。

## 3.3 与上游构建系统的差异

### 3.3.1 上游构建系统
上游使用 **GNU Autotools**：
- `configure.ac` - 自动配置脚本
- `Makefile.am` - Automake 输入
- 生成 `configure` 脚本和 `Makefile`

### 3.3.2 构建系统对比

| 方面 | 上游 (Autotools) | OH (GN) |
|------|-----------------|---------|
| 构建工具 | autoconf/automake | gn + ninja |
| 配置方式 | ./configure | config.h + BUILD.gn |
| 库类型 | 动态 (.so) + 静态 (.a) | 仅静态 |
| 安装路径 | /usr/lib, /usr/include | 组件内 |
| 测试支持 | make check | 未启用 |
| 文档生成 | make install | 不包含 |

### 3.3.3 配置映射

上游 `./configure` 检测的功能映射到 `config.h`：

| 功能检测 | config.h 宏 | 状态 |
|----------|------------|------|
| fnmatch.h | `HAVE_FNMATCH_H` | 启用 |
| glob.h | `HAVE_GLOB_H` | 启用 |
| iconv | `HAVE_ICONV` | 启用 |
| libintl | `HAVE_LIBINTL_H` | 启用 |
| secure_getenv | `HAVE_SECURE_GETENV` | 启用 |
| vasprintf | `HAVE_VASPRINTF` | 启用 |

## 3.4 config.h 详细分析

### 3.4.1 文件位置
```
third_party/popt/src/config.h
```

### 3.4.2 OH 特定的配置

#### Clang 兼容性处理
```c
#ifndef __BUILD_LINUX_WITH_CLANG
#define HAVE_MCHECK_H 1
#endif
```

当使用 Clang 编译器时，`mcheck.h` (内存检查) 被禁用，避免兼容性问题。

#### 启用的功能
```c
#define HAVE_FNMATCH_H 1      /* 文件名匹配 */
#define HAVE_GLOB_H 1         /* glob 模式匹配 */
#define HAVE_ICONV 1          /* 字符编码转换 */
#define HAVE_LIBINTL_H 1      /* 国际化支持 */
#define HAVE_SECURE_GETENV 1  /* 安全环境变量获取 */
#define HAVE_VASPRINTF 1      /* 可变参数格式化输出 */
#define HAVE_STPCPY 1         /* 字符串复制优化 */
#define HAVE_SRANDOM 1        /* 随机数生成 */
```

#### 禁用的功能
```c
#undef HAVE_INTTYPES_H
#undef HAVE_MEMORY_H
#undef HAVE_STRINGS_H
#undef HAVE_SYS_TYPES_H
#undef PACKAGE_URL
```

这些功能在 OH 环境中不可用或不需要。

### 3.4.3 版本信息
```c
#define PACKAGE_VERSION  "1.16"
```

**注意**: 这里定义的版本 1.16 与实际源码版本 1.19 不一致，建议更新。

## 3.5 特殊处理说明

### 3.5.1 功能裁剪

OH 构建裁剪了以下功能以减小体积和攻击面：

| 功能 | 文件 | 状态 | 原因 |
|------|------|------|------|
| 配置文件解析 | poptconfig.c | ❌ 未包含 | gptfdisk 不需要 |
| hash 函数 | lookup3.c | ❌ 未包含 | 未使用 |

### 3.5.2 国际化支持

popt 支持国际化，OH 构建启用了：
- `HAVE_LIBINTL_H` - 国际化库
- `HAVE_ICONV` - 字符编码转换

这确保了帮助信息可以根据系统语言环境正确显示。

## 3.6 依赖关系

### 3.6.1 构建依赖
```
popt_static (本库)
  └── ohos.gni, lite_component.gni (构建模板)
```

### 3.6.2 运行时依赖
popt 是**零依赖**库，仅使用：
- C 标准库 (libc)
- 可选: libintl (国际化)

## 3.7 使用方式

### 3.7.1 作为依赖引入
```gn
ohos_executable("my_tool") {
  external_deps = [
    "popt:popt_static",
  ]
  # 自动获得头文件路径
}
```

### 3.7.2 头文件引用
```c
#include <popt.h>
```

无需指定完整路径，BUILD.gn 的 `public_configs` 已配置好包含目录。

## 3.8 总结

| 方面 | 说明 |
|------|------|
| 构建复杂度 | 低 (仅 4 个源文件) |
| 配置方式 | config.h + BUILD.gn |
| 优化策略 | 大小优化 + 死代码消除 |
| 功能裁剪 | 配置文件解析 |
| 依赖关系 | 零运行时依赖 |
