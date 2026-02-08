# OpenHarmony 构建适配

## 概述

nghttp2 在 OpenHarmony 中使用 **BUILD.gn** 构建系统，而非上游的 CMake 或 Autotools。这种适配是 OH 第三方库的标准做法。

**构建文件位置**: `lib/BUILD.gn`

---

## BUILD.gn 结构

### 整体架构

```gn
# 定义公共源文件列表
nghttp2_lib_sources = [ ... ]  # 26 个源文件

llhttp_sources = [ ... ]       # 3 个源文件

# 定义配置
config("nghttp2_lib_config") { ... }
config("llhttp_public_config") { ... }

# 条件编译：Lite 系统
if (defined(ohos_lite)) {
  # LiteOS 目标: static_library, shared_library, ndk_lib
} else {
  # 标准系统目标: ohos_static_library, ohos_shared_library
}
```

### 源文件列表

#### nghttp2 核心库 (26 个文件)

```gn
nghttp2_lib_sources = [
  "nghttp2_alpn.c",           # ALPN (Application-Layer Protocol Negotiation)
  "nghttp2_buf.c",            # 缓冲区管理
  "nghttp2_callbacks.c",      # 回调管理
  "nghttp2_debug.c",          # 调试功能
  "nghttp2_extpri.c",         # 扩展优先级
  "nghttp2_frame.c",          # HTTP/2 帧处理
  "nghttp2_hd.c",             # HPACK 编解码器
  "nghttp2_hd_huffman.c",     # HPACK Huffman 编码
  "nghttp2_hd_huffman_data.c", # Huffman 编码数据表
  "nghttp2_helper.c",         # 辅助函数
  "nghttp2_http.c",           # HTTP 语义验证
  "nghttp2_map.c",            # 哈希映射
  "nghttp2_mem.c",            # 内存管理
  "nghttp2_option.c",         # 选项设置
  "nghttp2_outbound_item.c",  # 出站项管理
  "nghttp2_pq.c",             # 优先队列
  "nghttp2_priority_spec.c",  # 优先级规范
  "nghttp2_queue.c",          # 队列实现
  "nghttp2_ratelim.c",        # 速率限制
  "nghttp2_rcbuf.c",          # 引用计数缓冲区
  "nghttp2_session.c",        # HTTP/2 会话管理
  "nghttp2_stream.c",         # HTTP/2 流管理
  "nghttp2_submit.c",         # 请求/响应提交
  "nghttp2_time.c",           # 时间工具
  "nghttp2_version.c",        # 版本信息
  "sfparse.c",                # RFC 8941 Structured Field 解析
]
```

#### llhttp 库 (3 个文件)

```gn
llhttp_sources = [
  "../third-party/llhttp/src/api.c",
  "../third-party/llhttp/src/http.c",
  "../third-party/llhttp/src/llhttp.c",
]
```

---

## Lite 系统构建目标 (ohos_lite)

### 目标列表

| 目标 | 类型 | 输出 | 用途 |
|------|------|------|------|
| `nghttp2_lib_static` | static_library | `libnghttp2.a` | LiteOS 静态链接 |
| `nghttp2_lib_shared` | shared_library | `libnghttp2.so` | LiteOS 动态链接 |
| `llhttp_lib_static` | static_library | 内部使用 | llhttp 静态库 |
| `nghttp2_lib_ndk` | ndk_lib | NDK 导出 | 应用开发 |

### 配置详情

```gn
# 头文件包含路径
include_dirs = [
  ".",
  "./includes",
]

# 预定义宏
defines = [ "HAVE_ARPA_INET_H=1" ]
```

### NDK 导出配置

```gn
ndk_lib("nghttp2_lib_ndk") {
  # LiteOS-M 使用静态库
  if (ohos_kernel_type == "liteos_m") {
    lib_extension = ".a"
    deps = [ ":nghttp2_lib_static" ]
  } else {
    lib_extension = ".so"
    deps = [ ":nghttp2_lib_shared" ]
  }
  
  # 头文件路径
  head_files = [
    "//third_party/nghttp2/lib",
    "//third_party/nghttp2/lib/includes",
  ]
}
```

---

## 标准系统构建目标

### 目标列表

| 目标 | 类型 | 输出 | 用途 |
|------|------|------|------|
| `nghttp2` | ohos_static_library | 内部使用 | 静态库 |
| `libllhttp` | ohos_static_library | 内部使用 | llhttp |
| `libnghttp2_shared` | ohos_shared_library | `libnghttp2.so` | **主要目标** |
| `libnghttp2` | ohos_combine_darwin_framework | Framework | iOS 专用 |

### nghttp2 静态库

```gn
ohos_static_library("nghttp2") {
  include_dirs = [ "includes" ]
  license_file = "//third_party/nghttp2/COPYING"
  defines = []
  
  # Linux/Mac/OHOS 特有定义
  if (is_linux || is_mac || is_ohos) {
    defines += [
      "HAVE_ARPA_INET_H=1",
      "HAVE_NETINET_IN_H=1",
    ]
  }
  
  sources = nghttp2_lib_sources
}
```

### libnghttp2_shared（主要目标）

这是 OpenHarmony 中最常用的 nghttp2 构建目标。

```gn
ohos_shared_library("libnghttp2_shared") {
  include_dirs = [ "includes" ]
  license_file = "//third_party/nghttp2/COPYING"
  
  # 公共配置
  public_configs = [ ":libnghttp2_shared_config_public" ]
  
  # 预定义宏
  defines = [ "HAVE_TIME_H=1" ]
  
  # 平台特有定义
  if (is_linux || is_mac || is_ohos) {
    defines += [
      "HAVE_ARPA_INET_H=1",
      "HAVE_NETINET_IN_H=1",
    ]
  }
  
  # iOS 特殊处理
  if (current_os == "ios") {
    ldflags = [
      "-Wl",
      "-install_name",
      "@rpath/libnghttp2.framework/libnghttp2",
    ]
    output_name = "nghttp2"
  }
  
  # 分支保护（ARM PAC）
  branch_protector_ret = "pac_ret"
  
  sources = nghttp2_lib_sources
  
  # 非 iOS 平台使用版本脚本
  if (target_os != "ios") {
    if (product_name != "ohos-sdk") {
      version_script = "libnghttp2_shared.map"
    }
    
    # 安装镜像
    install_images = [
      "system",
      "updater",
    ]
  }
  
  # 子系统和部件信息
  subsystem_name = "thirdparty"
  innerapi_tags = [
    "chipsetsdk_indirect",
    "platformsdk",
  ]
  part_name = "nghttp2"
}
```

### 公共配置

```gn
config("libnghttp2_shared_config_public") {
  # 公共头文件路径
  include_dirs = [
    "//third_party/nghttp2/lib/includes",
    "//third_party/nghttp2/lib/includes/nghttp2",
  ]
  
  # 编译选项
  cflags = [ "-Wno-deprecated-declarations" ]
}
```

### llhttp 静态库

```gn
ohos_static_library("libllhttp") {
  sources = llhttp_sources
  public_configs = [ ":llhttp_public_config" ]
}

config("llhttp_public_config") {
  include_dirs = [ "../third-party/llhttp/include" ]
}
```

---

## 关键编译选项

### 预定义宏

| 宏 | 定义位置 | 说明 |
|----|---------|------|
| `HAVE_ARPA_INET_H=1` | nghttp2_lib_config, libnghttp2_shared | 使用 arpa/inet.h |
| `HAVE_NETINET_IN_H=1` | libnghttp2_shared | 使用 netinet/in.h |
| `HAVE_TIME_H=1` | libnghttp2_shared | 使用 time.h |

### 编译器标志

| 标志 | 位置 | 说明 |
|------|------|------|
| `-Wno-deprecated-declarations` | libnghttp2_shared_config_public | 忽略废弃声明警告 |

### 安全特性

| 特性 | 值 | 说明 |
|------|-----|------|
| `branch_protector_ret` | `pac_ret` | ARM Pointer Authentication Code |

---

## 版本脚本

**文件**: `lib/libnghttp2_shared.map`

版本脚本控制动态库的符号导出，确保只有公共 API 对外可见。

### 结构

```
1.55.0 {
    global:
        # 导出的公共符号
        nghttp2_session_client_new;
        nghttp2_session_server_new;
        nghttp2_submit_request;
        nghttp2_submit_response;
        ... (约 200+ 个符号)
    local:
        *;  # 隐藏所有其他符号
};
```

### 版本号说明

- 版本脚本中的 `1.55.0` 是符号版本，与库版本 `1.66.0` 不同
- 升级库时，如果新增公共 API，需要更新版本脚本

---

## 与上游构建系统的差异

### 上游构建系统

nghttp2 上游提供：
- **CMake** (`CMakeLists.txt`)
- **Autotools** (`configure.ac`, `Makefile.am`)

### OH 适配差异

| 方面 | 上游 | OH |
|------|------|-----|
| 构建系统 | CMake/Autotools | GN/Ninja |
| 配置方式 | configure 脚本 | BUILD.gn 条件编译 |
| 可选特性 | 可禁用应用/示例 | 仅构建核心库 |
| 安装目标 | 系统目录 | system/updater 镜像 |
| 符号控制 | 默认导出全部 | 版本脚本精确控制 |

### 简化点

OH BUILD.gn 相比上游做了简化：

1. **不构建应用程序**: nghttp, nghttpd, nghttpx, h2load
2. **不构建示例**: examples/ 目录
3. **不构建文档**: doc/ 目录
4. **固定配置**: 无 configure 选项，配置硬编码在 BUILD.gn

---

## 使用指南

### 在其他模块中依赖 nghttp2

```gn
# 标准系统
external_deps += [
  "nghttp2:libnghttp2_shared",
]

# 或 ArkUI-X
is_arkui_x {
  deps = [
    "//third_party/nghttp2/lib:libnghttp2_shared",
  ]
}

# LiteOS
if (ohos_kernel_type == "liteos_m") {
  deps += [ "//third_party/nghttp2/lib:nghttp2_lib_static" ]
} else {
  deps += [ "//third_party/nghttp2/lib:nghttp2_lib_shared" ]
}
```

### 头文件引用

```c
// 方式一：使用完整路径
#include <nghttp2/nghttp2.h>

// 方式二：直接引用（通过 include_dirs 配置）
#include <nghttp2.h>
```

**推荐**: 使用方式一 `nghttp2/nghttp2.h`，更清晰。

### 链接方式

| 场景 | 目标 | 说明 |
|------|------|------|
| 标准系统应用 | libnghttp2_shared | 动态链接，节省空间 |
| LiteOS-M | nghttp2_lib_static | 静态链接，无动态库开销 |
| LiteOS-A | nghttp2_lib_shared | 动态链接 |
| iOS | libnghttp2 (Framework) | 框架格式 |

---

## bundle.json 配置

nghttp2 作为 OH 组件的配置：

```json
{
  "name": "@ohos/nghttp2",
  "version": "3.1",
  "component": {
    "name": "nghttp2",
    "subsystem": "thirdparty",
    "adapted_system_type": ["small", "standard"],
    "build": {
      "inner_kits": [
        {
          "name": "//third_party/nghttp2/lib:libnghttp2_shared",
          "header": {
            "header_files": [],
            "header_base": [
              "//third_party/nghttp2/lib/includes",
              "//third_party/nghttp2/lib/includes/nghttp2"
            ]
          }
        },
        {
          "name": "//third_party/nghttp2/lib:libllhttp",
          "header": {
            "header_files": [],
            "header_base": [
              "//third_party/nghttp2/third-party/llhttp/include"
            ]
          }
        }
      ]
    }
  }
}
```

### inner_kits 说明

- `libnghttp2_shared`: 主要的 HTTP/2 库
- `libllhttp`: HTTP/1.1 解析器（供 nghttpx 使用）

---

## 常见问题

### Q: 为什么需要两个头文件路径？

```
header_base: [
  "//third_party/nghttp2/lib/includes",         # 包含 nghttp2/
  "//third_party/nghttp2/lib/includes/nghttp2"  # 直接包含
]
```

**A**: 提供灵活性，支持两种引用方式：
- `#include <nghttp2/nghttp2.h>`
- `#include <nghttp2.h>`

### Q: 版本脚本版本号为什么是 1.55.0？

**A**: 这是符号版本，不是库版本。通常保持与首次引入该符号集的版本一致。升级库时，如果新增符号，可以考虑更新版本号。

### Q: 为什么 iOS 有特殊处理？

**A**: iOS 使用 Framework 格式而非 .so，需要：
- 特殊的 `install_name`
- 使用 `ohos_combine_darwin_framework` 目标

### Q: branch_protector_ret 是什么？

**A**: ARM 架构的指针认证（Pointer Authentication）保护，防止 ROP/JOP 攻击。`pac_ret` 表示保护返回地址。
