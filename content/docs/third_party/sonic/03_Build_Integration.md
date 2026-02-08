# Sonic OH 构建适配

## 1. BUILD.gn 结构说明

### 1.1 完整配置

**文件路径**: `/Volumes/lexar/code/d/work/oh/third_party/sonic/BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

config("sonic_config") {
  visibility = [ ":*" ]

  include_dirs = [ "./" ]

  cflags = [
    "-Wall",
    "-Werror",
    "-Wno-implicit-function-declaration",
    "-Wno-sign-compare",
    "-Wno-unused-function",
    "-DHAVE_CONFIG_H",
    "-D_GNU_SOURCE",
  ]
}

config("sonic_include_config") {
  include_dirs = [ "./" ]
}

ohos_shared_library("sonic") {
  branch_protector_ret = "pac_ret"
  sources = [ "./sonic.c" ]

  license_file="./NOTICE"

  configs = [ ":sonic_config" ]

  public_configs = [ ":sonic_include_config" ]

  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "thirdparty"
  part_name = "sonic"
}
```

### 1.2 配置分层说明

```
BUILD.gn 结构
│
├── config("sonic_config")          # 私有编译配置
│   ├── visibility = [ ":*" ]      # 仅当前 BUILD.gn 可见
│   ├── include_dirs               # 头文件搜索路径
│   └── cflags                     # 编译器选项
│
├── config("sonic_include_config")  # 公开包含配置
│   └── include_dirs               # 对外暴露的头文件路径
│
└── ohos_shared_library("sonic")    # 共享库目标
    ├── branch_protector_ret       # 安全特性
    ├── sources                    # 源文件
    ├── license_file               # 许可证文件
    ├── configs                    # 应用的私有配置
    ├── public_configs             # 对外公开的配置
    ├── innerapi_tags              # API 级别标签
    ├── subsystem_name             # 子系统名称
    └── part_name                  # 部件名称
```

---

## 2. 关键编译选项分析

### 2.1 编译器警告选项

| 选项 | 说明 | OH 配置 | 分析 |
|------|------|---------|------|
| `-Wall` | 启用所有常见警告 | 启用 | 标准做法，提高代码质量 |
| `-Werror` | 将警告视为错误 | 启用 | 严格要求，确保无警告编译 |
| `-Wno-implicit-function-declaration` | 禁用隐式函数声明警告 | 禁用警告 | 可能上游代码使用了隐式声明 |
| `-Wno-sign-compare` | 禁用符号比较警告 | 禁用警告 | 有符号/无符号比较警告 |
| `-Wno-unused-function` | 禁用未使用函数警告 | 禁用警告 | 可能库中有未使用的辅助函数 |

**分析**:
- 前两个选项（`-Wall`、`-Werror`）体现了 OH 对代码质量的严格要求
- 后三个选项禁用了特定警告，可能是为了兼容上游代码的编码风格
- 这种做法在保证构建成功的同时，最大程度保持了代码原貌

### 2.2 预处理器定义

| 定义 | 说明 | 用途 |
|------|------|------|
| `-DHAVE_CONFIG_H` | 指示存在配置文件 | 上游代码可能通过 config.h 进行条件编译 |
| `-D_GNU_SOURCE` | 启用 GNU C 扩展 | 使用 GNU 特定的函数和功能 |

**分析**:
- `HAVE_CONFIG_H`: sonic 上游可能支持 autotools 构建系统，使用 config.h
- `_GNU_SOURCE`: 启用 GNU 扩展，可能用于 `M_PI` 等数学常量

### 2.3 头文件路径配置

```gn
include_dirs = [ "./" ]
```

**分析**:
- 使用相对路径 `"./"`，指向 sonic 库根目录
- 使用者通过 `#include "sonic.h"` 即可引用头文件
- 简单的扁平结构，符合小型库的惯例

---

## 3. 与上游构建系统的差异

### 3.1 上游构建系统

sonic 上游提供的是 **Makefile** 构建系统：

**原始 Makefile 关键部分**:
```makefile
# 默认编译选项
CFLAGS=-O3 -Wall -fPIC

# 共享库构建
libsonic.so: sonic.o
    $(CC) -shared -Wl,-soname,libsonic.so.0 -o libsonic.so.0.2.0 sonic.o -lm

# 安装
install: libsonic.so sonic.h
    install -D -m 644 sonic.h $(DESTDIR)$(INCLUDEDIR)/sonic.h
    install -D -m 755 libsonic.so.0.2.0 $(DESTDIR)$(LIBDIR)/libsonic.so.0.2.0
```

### 3.2 差异对比

| 方面 | 上游 Makefile | OH BUILD.gn | 说明 |
|------|---------------|-------------|------|
| **构建系统** | GNU Make | GN + Ninja | OH 使用统一的 GN 构建系统 |
| **编译选项** | `-O3 -Wall -fPIC` | `-Wall -Werror ...` | OH 更严格，禁用部分警告 |
| **优化级别** | `-O3` | 默认（通常是 -O2） | 可能略有性能差异 |
| **位置无关代码** | `-fPIC` | 默认（共享库自动启用）| BUILD.gn 不显式指定 |
| **链接选项** | `-shared -Wl,-soname...` | `ohos_shared_library` | OH 自动处理共享库链接 |
| **数学库** | 显式链接 `-lm` | 自动处理 | OH 自动链接数学库 |
| **安全特性** | 无 | `branch_protector_ret` | OH 添加安全增强 |
| **API 标签** | 无 | `platformsdk` | OH 特有的内部 API 标记 |
| **子系统** | 无 | `thirdparty` | OH 组件管理需要 |

### 3.3 OH 特有增强

#### 3.3.1 分支保护（Branch Protector）

```gn
branch_protector_ret = "pac_ret"
```

**说明**:
- 使用 ARM Pointer Authentication (PAC) 技术
- 保护返回地址，防止 ROP（Return-Oriented Programming）攻击
- 这是 OH 的安全加固措施，上游版本不具备

#### 3.3.2 API 级别标记

```gn
innerapi_tags = [ "platformsdk" ]
```

**说明**:
- `platformsdk`: 表示这是平台 SDK 内部 API
- 供 OH 系统组件使用，不对外暴露给应用开发者
- 这是 OH 的 API 管理策略

#### 3.3.3 许可证声明

```gn
license_file = "./NOTICE"
```

**说明**:
- 显式声明许可证文件位置
- OH 构建系统会处理许可证合规性

---

## 4. 构建产物

### 4.1 输出文件

使用 OH 构建系统编译 sonic 时，将生成：

| 产物 | 文件名 | 说明 |
|------|--------|------|
| 共享库 | `libsonic.so` | 主要构建产物 |
| 头文件 | `sonic.h` | 通过 `public_configs` 暴露 |

### 4.2 安装位置

```
/system/lib/          # libsonic.so
/system/include/      # sonic.h（通过 SDK 引用）
```

---

## 5. 使用方式

### 5.1 依赖声明

其他模块在 BUILD.gn 中引用 sonic：

```gn
ohos_executable("my_audio_app") {
  sources = [ "main.cpp" ]
  deps = [
    "//third_party/sonic:sonic",  # 声明依赖
  ]
  # public_configs 会自动继承 sonic_include_config
}
```

### 5.2 头文件引用

```c
#include "sonic.h"  // 无需完整路径，已配置 include_dirs

// 使用 sonic API
sonicStream stream = sonicCreateStream(44100, 2);
```

---

## 6. 构建适配总结

### 6.1 适配复杂度评估

| 评估项 | 等级 | 说明 |
|--------|------|------|
| **整体复杂度** | 低 | 单层 BUILD.gn，配置简单 |
| **代码修改** | 无 | 无需修改源代码 |
| **配置调整** | 中 | 调整编译选项以兼容 OH 要求 |
| **维护成本** | 低 | 结构简单，易于维护 |

### 6.2 关键适配点

1. **编译选项适配**: 禁用部分上游代码触发的警告，同时保持 `-Wall -Werror` 严格性
2. **安全增强**: 添加 `branch_protector_ret` 保护
3. **API 标记**: 标记为 `platformsdk` 内部 API
4. **子系统归属**: 归入 `thirdparty` 子系统

### 6.3 与上游的兼容性

- **API 完全兼容**: 未修改任何 API 定义
- **行为一致**: 算法实现未改变
- **可平滑升级**: 升级上游版本时，只需保持 BUILD.gn 不变
