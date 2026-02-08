# 04_Build_Configuration - 构建配置

## 4.1 GN 构建配置

### 4.1.1 BUILD.gn 文件

**位置**：`//base/hiviewdfx/blackbox_lite/BUILD.gn`

**内容**：
```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
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

import("//build/lite/ndk/ndk.gni")

static_library("blackbox_lite") {
  sources = [
    "blackbox_adapter.c",
    "blackbox_core.c",
    "blackbox_detector.c",
  ]
  defines = []
  cflags = [ "-Wall" ]
  deps = []
  include_dirs = [
    "//base/hiviewdfx/blackbox_lite",
    "//base/hiviewdfx/blackbox_lite/interfaces/native/kits",
    "//base/hiviewdfx/hiview_lite",
    "//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite",
    "//commonlibrary/utils_lite/include",
  ]
}
```

### 4.1.2 Target 详解

| 属性 | 值 | 说明 |
|------|-----|------|
| target 类型 | `static_library` | 静态库 |
| 输出名 | `libblackbox_lite.a` | GN 自动生成 |
| sources | 3 个 C 文件 | 核心代码 |
| defines | 空 | 无预定义宏 |
| cflags | `-Wall` | 启用所有警告 |
| deps | 空 | 无内部依赖 |

**sources 文件列表**：

| 文件 | 用途 |
|------|------|
| `blackbox_adapter.c` | WEAK 适配层实现 |
| `blackbox_core.c` | 核心逻辑 |
| `blackbox_detector.c` | 事件上报 |

**include_dirs 详解**：

| 路径 | 用途 |
|------|------|
| `//base/hiviewdfx/blackbox_lite` | 本地头文件 |
| `//base/hiviewdfx/blackbox_lite/interfaces/native/kits` | 公共接口 |
| `//base/hiviewdfx/hiview_lite` | hiview_lite 依赖 |
| `//base/hiviewdfx/hilog_lite/interfaces/native/kits/hilog_lite` | hilog_lite 依赖 |
| `//commonlibrary/utils_lite/include` | utils_lite 依赖 |

## 4.2 组件配置

### 4.2.1 bundle.json

**位置**：`//base/hiviewdfx/blackbox_lite/bundle.json`

**内容**：
```json
{
    "name": "@ohos/blackbox_lite",
    "description": "blackbox_lite provides the software blackbox capability.",
    "optional": "false",
    "version": "3.1",
    "license": "Apache License 2.0",
    "publishAs": "code-segment",
    "segment": {
        "destPath": "base/hiviewdfx/blackbox_lite"
    },
    "dirs": {},
    "scripts": {},
    "component": {
        "name": "blackbox_lite",
        "subsystem": "hiviewdfx",
        "adapted_system_type": ["mini"],
        "rom": "10KB",
        "ram": "~5KB",
        "deps": {
            "components": [
                "utils_lite",
                "liteos_m"
            ],
            "third_party": []
        },
        "build": {
            "sub_component": [
                "//base/hiviewdfx/blackbox_lite:blackbox_lite"
            ]
        }
    }
}
```

### 4.2.2 组件属性

| 属性 | 值 | 说明 |
|------|-----|------|
| name | `@ohos/blackbox_lite` | NPM 包名 |
| version | `3.1` | 版本号 |
| subsystem | `hiviewdfx` | 所属子系统 |
| adapted_system_type | `["mini"]` | 适配 Mini 系统 |
| rom | `10KB` | ROM 占用 |
| ram | `~5KB` | RAM 占用 |

### 4.2.3 依赖组件

| 组件 | 类型 | 用途 |
|------|------|------|
| `utils_lite` | 系统组件 | 链表、内存操作 |
| `liteos_m` | 系统组件 | 内核 API |

## 4.3 编译产物

### 4.3.1 产物清单

| 产物类型 | 输出路径 | 说明 |
|----------|----------|------|
| 静态库 | `out/.../libs/libblackbox_lite.a` | 主要构建产物 |
| 头文件 | `out/.../include/blackbox*.h` | NDK 头文件（若启用） |

### 4.3.2 运行时加载关系

```
应用程序/模块
    │
    ├─ 链接 libblackbox_lite.a (静态链接)
    │
    ├─ 依赖 libhiview_lite.z.so 或 .a
    │       │
    │       └──► 静态链接 blackbox_lite
    │
    └─ 依赖 libhilog_lite.z.so 或 .a
```

**说明**：blackbox_lite 作为静态库链接到使用方，无独立的运行时加载关系。

### 4.3.3 NDK 导出（可选）

若启用 `ndk.gni`，NDK 构建会导出以下头文件：

| 头文件 | 来源 |
|--------|------|
| `blackbox.h` | `interfaces/native/kits/blackbox.h` |
| `blackbox_adapter.h` | `interfaces/native/kits/blackbox_adapter.h` |

## 4.4 平台适配构建

### 4.4.1 适配层链接

平台需创建 `blackbox_adapter_impl.c`，实现所有 WEAK 函数，并在构建配置中链接：

```gn
# 平台 BUILD.gn 示例
static_library("platform_blackbox") {
  sources = [
    "path/to/blackbox_adapter_impl.c",
  ]
  deps = [
    "//base/hiviewdfx/blackbox_lite:blackbox_lite",
  ]
}
```

### 4.4.2 适配函数清单

平台必须实现的函数：

| 函数名 | 头文件位置 | 用途 |
|--------|------------|------|
| `SystemModuleDump` | `blackbox_adapter.h:33` | 系统 Dump |
| `SystemModuleReset` | `blackbox_adapter.h:34` | 系统复位 |
| `SystemModuleGetLastLogInfo` | `blackbox_adapter.h:35` | 获取日志信息 |
| `SystemModuleSaveLastLog` | `blackbox_adapter.h:36` | 保存日志 |
| `FullWriteFile` | `blackbox_adapter.h:37` | 文件写操作 |
| `GetFaultLogPath` | `blackbox_adapter.h:38` | 获取日志路径 |
| `RebootSystem` | `blackbox_adapter.h:39` | 系统重启 |

## 4.5 编译选项

### 4.5.1 条件编译

| 宏 | 说明 | 使用位置 |
|----|------|----------|
| `BLACKBOX_DEBUG` | 启用调试日志 | `blackbox_core.c:191` |
| `BLACKBOX_TEST` | 启用测试代码 | `blackbox_adapter.c:75` |

### 4.5.2 调试日志示例

**证据**：`blackbox_core.c:191-204`
```c
#ifdef BLACKBOX_DEBUG
static void PrintModuleOps(void)
{
    struct BBoxOps *temp = NULL;

    BBOX_PRINT_INFO("The following modules have been registered!\n");
    UTILS_DL_LIST_FOR_EACH_ENTRY(temp, &g_opsList, struct BBoxOps, opsList) {
        BBOX_PRINT_INFO("module: %s, Dump: %p, Reset: %p, "
            "GetLastLogInfo: %p, SaveLastLog: %p\n",
            temp->ops.module, temp->ops.Dump, temp->ops.Reset,
            temp->ops.GetLastLogInfo, temp->ops.SaveLastLog);
    }
}
#endif
```

---

## 参考文档

- [01_Overview](01_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构说明
- [03_API_Reference](03_API_Reference.md) - API 参考
- [06_Troubleshooting](06_Troubleshooting.md) - 常见问题
