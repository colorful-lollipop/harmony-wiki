# 03 - OHOS 构建适配

## 3.1 BUILD.gn 结构概述

OpenHarmony 使用 GN (Generate Ninja) 构建系统，PulseAudio 的构建配置位于 `ohosbuild/` 目录下。

### 目录结构

```
ohosbuild/
├── BUILD.gn                    # 根配置，定义 pulseaudio_packages
├── ohos_paconfig.sh            # 配置脚本，生成 config.h
├── include/
│   └── log/
│       └── audio_log.h         # HiLog 日志头文件
├── src/
│   ├── BUILD.gn               # pulsecommon, 配置文件
│   ├── daemon/
│   │   └── BUILD.gn           # pulseaudio 主程序
│   ├── modules/
│   │   └── BUILD.gn           # 协议模块
│   ├── pulse/
│   │   └── BUILD.gn           # 客户端库 (pulse, pulse-simple)
│   ├── pulsecore/
│   │   └── BUILD.gn           # 核心库，协议实现
│   └── utils/
│       └── BUILD.gn           # 工具 (pacmd, pactl)
└── src/pulsecore/
    └── ltdl_stub.c            # 动态加载存根
```

---

## 3.2 根 BUILD.gn 详解

### 文件位置
`ohosbuild/BUILD.gn`

### 主要内容

```python
# 定义 pulseaudio_packages 组，包含所有子组件
group("pulseaudio_packages") {
  deps = [
    "../sonic:sonic",                          # Sonic 变速库
    "src:pa_client_config",                    # 客户端配置
    "src:pa_daemon_config",                    # 守护进程配置
    "src:pa_default_config",                   # 默认配置
    "src:pulsecommon",                         # 通用代码
    "src/daemon:pulseaudio",                   # 主守护进程
    "src/modules:native-modules",              # 协议模块
    "src/pulse:pulse",                         # 客户端库
    "src/pulse:pulse-simple",                  # 简单客户端库
    "src/pulsecore:cli",                       # CLI 支持
    "src/pulsecore:protocol-cli",              # CLI 协议
    "src/pulsecore:protocol-native",           # 原生协议
    "src/pulsecore:pulsecore",                 # 核心库
  ]
  
  # root 变体包含额外工具
  if (build_variant == "root") {
    deps += [
      "src/utils:pacmd",
      "src/utils:pactl",
    ]
  }
}

# 配置头文件生成操作
action("gen_config_header") {
  script = "ohos_paconfig.sh"
  args = [
    rebase_path("//third_party/pulseaudio", root_build_dir),
    rebase_path("${target_gen_dir}/", root_build_dir),
  ]
  outputs = [ "${target_gen_dir}/config.h" ]
}
```

### 特殊之处

| 特性 | 说明 |
|-----|------|
| `ohos_paconfig.sh` | 自定义脚本生成 config.h，替代上游的 configure |
| `build_variant` 条件 | root 变体额外包含调试工具 |

---

## 3.3 守护进程 BUILD.gn

### 文件位置
`ohosbuild/src/daemon/BUILD.gn`

### 关键配置

```python
config("daemon_config") {
  visibility = [ ":*" ]
  
  include_dirs = [
    "../../include",
    "../../../include",
    "../../../src/daemon",
    "../../../src",
  ]
  
  cflags = [
    "-Wall",
    "-Werror",
    "-Wno-unused-function",
    "-DHAVE_CONFIG_H",
    "-DHAVE_UNISTD_H",
  ]
}

ohos_source_set("pulseaudio_sources") {
  sources = [
    "../../../src/daemon/caps.c",
    "../../../src/daemon/cmdline.c",
    "../../../src/daemon/cpulimit.c",
    "../../../src/daemon/ohos_daemon-conf.c",    # OHOS 配置
    "../../../src/daemon/ohos_pa_main.c",        # OHOS 主入口
  ]
  
  configs = [ ":daemon_config" ]
  external_deps = [ "hilog:libhilog" ]
  part_name = "pulseaudio"
  subsystem_name = "thirdparty"
}

ohos_shared_library("pulseaudio") {
  ldflags = [ "-ffast-math" ]
  deps = [
    ":pulseaudio_sources",
    "../../src:pulsecommon",
    "../../src/pulse:pulse",
    "../../src/pulsecore:pulsecore",
  ]
  external_deps = [ "hilog:libhilog" ]
  part_name = "pulseaudio"
  subsystem_name = "thirdparty"
}
```

### 与上游构建的差异

| 方面 | 上游 (Meson) | OHOS (GN) |
|-----|-------------|-----------|
| 配置系统 | Meson + configure | GN + ohos_paconfig.sh |
| 主入口 | main.c | ohos_pa_main.c |
| 配置源 | daemon-conf.c | ohos_daemon-conf.c |
| 依赖管理 | 系统包管理 | external_deps |

---

## 3.4 PulseCore BUILD.gn

### 文件位置
`ohosbuild/src/pulsecore/BUILD.gn`

### 关键配置

```python
config("pulsecore_config") {
  cflags = [
    "-Wall",
    "-Werror",
    "-Wno-implicit-function-declaration",
    "-Wno-unused-function",
    "-Wno-uninitialized",
    "-DHAVE_CONFIG_H",
    "-D_GNU_SOURCE",
    "-D__INCLUDED_FROM_PULSE_AUDIO",
  ]
}

ohos_source_set("pulsecore_sources") {
  sources = [
    # ... 大量源文件 ...
    "../../../src/pulsecore/protocol-native.c",  # 原生协议实现
    "../../../src/pulsecore/ohos_socket-server.c", # OHOS socket 服务器
    "../../src/pulsecore/ltdl_stub.c",           # 动态加载存根
  ]
  
  configs = [ ":pulsecore_config" ]
  
  external_deps = [
    "bounds_checking_function:libsec_shared",
    "c_utils:utils",
    "hilog:libhilog",
    "init:libbegetutil",  # 提供 GetControlSocket
  ]
  
  # 条件编译：HiTrace 支持
  defines = []
  if (defined(global_parts_info) &&
      defined(global_parts_info.hiviewdfx_hitrace)) {
    defines += [ "FEATURE_HITRACE_METER" ]
    external_deps += [ "hitrace:hitrace_meter" ]
  }
}

ohos_shared_library("pulsecore") {
  sanitize = {
    integer_overflow = true  # 整数溢出检测
  }
  
  deps = [ ":pulsecore_sources", "../../src:pulsecommon" ]
  
  external_deps = [
    "bounds_checking_function:libsec_shared",
    "c_utils:utils",
    "hilog:libhilog",
    "init:libbegetutil",
  ]
  
  innerapi_tags = [
    "chipsetsdk",
    "platformsdk_indirect",
  ]
}
```

### 子库构建

```python
ohos_shared_library("cli") {
  sources = [ "../../../src/pulsecore/cli.c" ]
  configs = [ ":modules_internal_lib_config" ]
  deps = [ "../../src:pulsecommon", "../../src/pulsecore:pulsecore" ]
  external_deps = [ "hilog:libhilog" ]
}

ohos_shared_library("protocol-cli") {
  sources = [ "../../../src/pulsecore/protocol-cli.c" ]
  deps = [ "...", "../../src/pulsecore:cli" ]
}

ohos_shared_library("protocol-native") {
  sources = [ "../../../src/pulsecore/protocol-native.c" ]
  # 同样支持 FEATURE_HITRACE_METER
}
```

### 安全特性

| 特性 | 配置 | 说明 |
|-----|------|------|
| 整数溢出检测 | `sanitize.integer_overflow = true` | 运行时检测整数溢出 |
| 边界检查 | `bounds_checking_function:libsec_shared` | 内存安全增强 |

---

## 3.5 Sonic BUILD.gn

### 文件位置
`sonic/BUILD.gn`

### 配置

```python
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

ohos_shared_library("sonic") {
  branch_protector_ret = "pac_ret"  # 返回地址保护
  sources = [ "./sonic.c" ]
  configs = [ ":sonic_config" ]
  public_configs = [ ":sonic_include_config" ]
  innerapi_tags = [ "platformsdk" ]
  subsystem_name = "thirdparty"
  part_name = "pulseaudio"
}
```

### 安全特性

| 特性 | 配置 | 说明 |
|-----|------|------|
| PAC-RET | `branch_protector_ret = "pac_ret"` | ARM 指针认证，防止 ROP 攻击 |

---

## 3.6 与上游构建系统的差异

### 构建系统对比

| 特性 | 上游 (Meson) | OHOS (GN) |
|-----|-------------|-----------|
| 构建工具 | Meson + Ninja | GN + Ninja |
| 配置检测 | configure 脚本 | ohos_paconfig.sh + 静态配置 |
| 模块化 | 动态模块 (.so) | 静态链接库 |
| 依赖管理 | pkg-config | external_deps |
| 安装目标 | 系统目录 | 系统镜像 |

### 配置差异

上游使用 `meson_options.txt` 定义可选功能，OHOS 使用 GN 的 `defines` 和条件编译。

#### 上游配置示例 (Meson)
```python
# meson_options.txt
option('daemon', type: 'boolean', value: true)
option('client', type: 'boolean', value: true)
option('tests', type: 'boolean', value: true)
```

#### OHOS 配置 (GN)
```python
# 通过 defines 控制
if (defined(global_parts_info.hiviewdfx_hitrace)) {
  defines += [ "FEATURE_HITRACE_METER" ]
}
```

### 关键差异说明

1. **HAVE_NO_OHOS 宏**
   - 在 `ohos_paconfig.sh` 中定义
   - 用于条件编译禁用原生不兼容功能

2. **动态模块禁用**
   - 上游使用 libltdl 动态加载模块
   - OHOS 使用 `ltdl_stub.c` 提供空实现
   - 模块静态链接到主程序

3. **Init 集成**
   - 依赖 `init:libbegetutil` 获取控制 socket
   - 构建时就需要链接，不是运行时加载

---

## 3.7 特殊处理说明

### 3.7.1 动态加载替代 (ltdl_stub.c)

由于 OHOS 不使用动态模块加载，提供了存根实现：

```c
// ltdl_stub.c
// 提供 ltdl 函数的空实现
```

### 3.7.2 配置文件生成

`ohos_paconfig.sh` 生成 `config.h`，定义：
- `HAVE_NO_OHOS` - 启用 OHOS 适配
- 各种功能宏 (HAVE_UNISTD_H 等)
- 版本信息

### 3.7.3 禁用特性

通过 `HAVE_NO_OHOS` 宏禁用的特性：
- 动态模块加载 (libltdl)
- 某些系统调用 (close_allv)
- 文件权限掩码设置

---

## 3.8 编译选项总结

### 全局 CFLAGS

```
-Wall -Werror                    # 严格警告
-DHAVE_CONFIG_H                 # 使用配置头
-D_GNU_SOURCE                   # GNU 扩展
-D__INCLUDED_FROM_PULSE_AUDIO   # PA 内部编译标记
```

### 警告抑制

```
-Wno-unused-function            # 允许未使用函数
-Wno-implicit-function-declaration
-Wno-uninitialized
-Wno-sign-compare
```

### 链接选项

```
-ffast-math                     # 快速数学运算 (daemon)
```

### 条件定义

| 定义 | 条件 | 说明 |
|-----|------|------|
| `FEATURE_HITRACE_METER` | `global_parts_info.hiviewdfx_hitrace` 存在 | 启用性能跟踪 |
| `HAVE_NO_OHOS` | 始终定义 | 启用 OHOS 适配 |
