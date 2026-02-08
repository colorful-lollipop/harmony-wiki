# 03 - OH 构建适配

> alsa-utils 在 OpenHarmony 中的构建系统适配

---

## 1. BUILD.gn 结构

### 1.1 整体结构

```gn
import("//build/ohos.gni")

# 路径常量定义
ASOUND_STATE_DIR = "/var/lib/alsa"
ASOUND_LOCK_DIR = "/var/run"

# 公共配置块
config("alsa_utils_config") {
  cflags = [ ... ]
  if (use_musl) { ... }
}

# 可执行文件目标
ohos_executable("aconnect")    { ... }
ohos_executable("amixer")      { ... }
ohos_executable("aplay")       { ... }
ohos_executable("speaker-test") { ... }
ohos_executable("alsactl")     { ... }

# 聚合目标
group("alsa-utils") {
  deps = [":aconnect", ":alsactl", ":amixer", ":aplay", ":speaker-test"]
}
```

### 1.2 配置层级

```
┌─────────────────────────────────┐
│  config("alsa_utils_config")   │  # 公共配置
│  - 编译器标志                   │
│  - 预处理器宏                   │
│  - musl 适配                   │
└──────────────┬──────────────────┘
               │ 应用于所有目标
    ┌──────────┴──────────┐
    │                     │
┌───▼────┐   ┌───▼────┐  ...
│ aplay  │   │ amixer │
└───┬────┘   └───┬────┘
    │           │
    └─────┬─────┘
          │
    ┌─────▼─────┐
    │ alsa-utils│  # group 目标
    └───────────┘
```

---

## 2. 关键适配点

### 2.1 路径常量定义

```gn
ASOUND_STATE_DIR = "/var/lib/alsa"
ASOUND_LOCK_DIR = "/var/run"
ASOUND_LOCK_DIR = "/var/run"  # 重复定义,覆盖前面的 /var/lock
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `ASOUND_STATE_DIR` | `/var/lib/alsa` | ALSA 状态文件目录 |
| `ASOUND_LOCK_DIR` | `/var/run` | 锁文件目录 |

**用途**:
- 状态文件: `/var/lib/alsa/asound.state` (alsactl 保存的声卡配置)
- 锁文件: `/var/run/asound.state.lock` (防止并发修改)

### 2.2 公共配置块

```gn
config("alsa_utils_config") {
  cflags = [
    "-Wno-sign-compare",
    "-Wno-implicit-function-declaration",
    "-Wno-parentheses",
    "-Wno-string-conversion",
    "-Wno-string-plus-int",
    "-Wno-asm-operand-widths",
    "-Wno-pointer-sign",
    "-Wno-deprecated-declarations",
    "-Wno-implicit-int",
    "-Wno-switch",
    "-Wno-incompatible-pointer-types-discards-qualifiers",
    "-Wno-int-conversion",
    "-Wno-absolute-value",
    "-Wno-unused-function",
    "-Wno-unused-label",
    "-Wno-unused-const-variable",
    "-Wno-visibility",
    "-Wno-incompatible-pointer-types",
    "-Wno-sometimes-uninitialized",
    "-Wno-format",
    "-Wno-tautological-constant-out-of-range-compare",
    "-Wno-implicit-fallthrough",
    "-Wno-error",
    "-DHAVE_CONFIG_H",
    "-D_GNU_SOURCE",
    "-D__USE_GNU",
    "-DCURSESINC=\"\"",
    "-DSYS_ASOUNDRC=\"$ASOUND_STATE_DIR/asound.state\"",
    "-DSYS_LOCKFILE=\"$ASOUND_LOCK_DIR/asound.state.lock\"",
  ]

  if (use_musl) {
    cflags += [ "-Wno-bool-operation" ]
  }
}
```

#### 2.2.1 编译器警告抑制

| 标志 | 说明 | 为什么需要 |
|------|------|-----------|
| `-Wno-error` | 不将警告视为错误 | 适配 OH 编译环境的严格性 |
| `-Wno-implicit-function-declaration` | 允许隐式函数声明 | ALSA 代码风格,不完全符合现代标准 |
| `-Wno-sign-compare` | 允许有符号/无符号比较 | ALSA 代码习惯 |
| `-Wno-deprecated-declarations` | 允许使用废弃声明 | ALSA 依赖旧 API |
| ... (其他 19 个) | ... | 适配 ALSA 代码风格 |

#### 2.2.2 预处理器宏

| 宏 | 值 | 说明 |
|----|-----|------|
| `HAVE_CONFIG_H` | 1 | 启用 `include/aconfig.h` 配置头文件 |
| `_GNU_SOURCE` | 1 | 启用 GNU 扩展特性 |
| `__USE_GNU` | 1 | GNU 特性标识 |
| `CURSESINC` | `""` | curses 头文件路径 (空表示禁用) |
| `SYS_ASOUNDRC` | `/var/lib/alsa/asound.state` | ALSA 状态文件路径 |
| `SYS_LOCKFILE` | `/var/run/asound.state.lock` | 锁文件路径 |

#### 2.2.3 musl libc 适配

```gn
if (use_musl) {
  cflags += [ "-Wno-bool-operation" ]
}
```

| 条件 | 说明 |
|------|------|
| `use_musl` | OH 构建系统全局变量,区分 musl libc 和 glibc |
| `-Wno-bool-operation` | musl 编译器对 bool 操作有不同警告 |

### 2.3 目标配置模式

每个可执行文件目标遵循统一模式:

```gn
ohos_executable("tool_name") {
  sources = [                    # 显式列出所有源文件
    "tool/tool.c",
    "tool/helper.c",
    ...
  ]

  include_dirs = [               # 三层头文件路径
    "//third_party/alsa-utils/tool",    # 模块私有头文件
    "//third_party/alsa-utils/include", # 公共头文件
    "//third_party/alsa-lib/include",   # 依赖库头文件
  ]

  configs = [                    # 引用公共配置
    ":alsa_utils_config"
  ]

  deps = [                       # 依赖项
    "../alsa-lib:libasound"
  ]

  # OH 特定属性
  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
  install_enable = true          # 可选
  install_images = ["system"]    # 可选
  symlink_target_name = ["arecord"]  # 可选
}
```

---

## 3. 编译目标详解

### 3.1 目标清单

| 目标 | 源文件数 | install_enable | install_images | symlink |
|------|----------|----------------|----------------|---------|
| aconnect | 1 | false | - | - |
| amixer | 2 | ✅ true | system | - |
| aplay | 1 | ✅ true | system | arecord |
| speaker-test | 3 | false | - | - |
| alsactl | 10 | ✅ true | system | - |

### 3.2 依赖关系

```
alsa-utils (group)
    ├── aconnect ────────┐
    ├── amixer ────────┐ │
    ├── aplay ─────────┐││
    ├── speaker-test ┐ │││
    └── alsactl ────┐│ │││
                   ││ │││
                   alsa-lib:libasound
                       │
                  pthread, rt, m
```

### 3.3 目标详细配置

#### 3.3.1 aplay

```gn
ohos_executable("aplay") {
  sources = [ "aplay/aplay.c" ]

  include_dirs = [
    "//third_party/alsa-utils/aplay",
    "//third_party/alsa-utils/include",
    "//third_party/alsa-lib/include",
  ]

  configs = [ ":alsa_utils_config" ]
  deps = [ "../alsa-lib:libasound" ]

  symlink_target_name = [ "arecord" ]  # 创建 arecord 链接

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
  install_images = [ "system" ]
}
```

**特点**:
- 单源文件编译
- 创建 `arecord` 符号链接
- 安装到 system 分区

#### 3.3.2 amixer

```gn
ohos_executable("amixer") {
  sources = [
    "amixer/amixer.c",
    "amixer/volume_mapping.c",
  ]

  include_dirs = [
    "//third_party/alsa-utils/amixer",
    "//third_party/alsa-utils/include",
    "//third_party/alsa-lib/include",
  ]

  configs = [ ":alsa_utils_config" ]
  deps = [ "../alsa-lib:libasound" ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
  install_images = [ "system" ]
}
```

**特点**:
- 2 个源文件 (主程序 + 音量映射)
- 安装到 system 分区

#### 3.3.3 alsactl

```gn
ohos_executable("alsactl") {
  sources = [
    "alsactl/alsactl.c",
    "alsactl/clean.c",
    "alsactl/daemon.c",
    "alsactl/info.c",
    "alsactl/init_parse.c",
    "alsactl/init_ucm.c",
    "alsactl/lock.c",
    "alsactl/monitor.c",
    "alsactl/state.c",
    "alsactl/utils.c",
  ]

  include_dirs = [
    "//third_party/alsa-utils/alsactl",
    "//third_party/alsa-utils/include",
    "//third_party/alsa-lib/include",
  ]

  configs = [ ":alsa_utils_config" ]
  deps = [ "../alsa-lib:libasound" ]

  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
  install_images = [ "system" ]
}
```

**特点**:
- 10 个源文件 (最复杂的工具)
- 支持守护进程模式
- UCM (Use Case Manager) 支持
- 安装到 system 分区

#### 3.3.4 speaker-test

```gn
ohos_executable("speaker-test") {
  sources = [
    "speaker-test/pink.c",
    "speaker-test/speaker-test.c",
    "speaker-test/st2095.c",
  ]

  include_dirs = [
    "//third_party/alsa-utils/speaker-test",
    "//third_party/alsa-utils/include",
    "//third_party/alsa-lib/include",
  ]

  configs = [ ":alsa_utils_config" ]
  deps = [ "../alsa-lib:libasound" ]

  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
}
```

**特点**:
- 3 个源文件 (测试信号生成 + 主程序)
- 不安装 (仅用于调试)

#### 3.3.5 aconnect

```gn
ohos_executable("aconnect") {
  sources = [ "seq/aconnect/aconnect.c" ]

  include_dirs = [
    "//third_party/alsa-utils/include",
    "//third_party/alsa-lib/include",
  ]

  configs = [ ":alsa_utils_config" ]
  deps = [ "../alsa-lib:libasound" ]

  subsystem_name = "thirdparty"
  part_name = "alsa-utils"
}
```

**特点**:
- 单源文件编译
- MIDI 连接管理工具
- 不安装 (MIDI 调试用)

---

## 4. 功能裁剪

### 4.1 上游支持的工具

```
alsactl, alsamixer, amixer, amidi, aplay, arecord,
iecset, speaker-test, axfer, alsaloop, alsabat,
aconnect, aseqdump, aseqnet, aplaymidi, alsaucm,
alsatplg, nhlt, alsaconf, alsa-info
```

### 4.2 OH 实际编译的工具

```
alsactl, amixer, aplay, speaker-test, aconnect
```

### 4.3 裁剪对照表

| 工具 | 上游 | OH | 裁剪原因 |
|------|------|----|---------|
| **编译的工具** |
| alsactl | ✅ | ✅ | 核心配置工具,调试必需 |
| amixer | ✅ | ✅ | 命令行混音器,调试必需 |
| aplay | ✅ | ✅ | 音频播放,调试必需 |
| arecord | ✅ | ✅ | 音频录制,aplay 链接 |
| speaker-test | ✅ | ✅ | 扬声器测试,调试必需 |
| aconnect | ✅ | ✅ | MIDI 连接管理,MIDI 调试 |
| **未编译的工具** |
| alsamixer | ✅ | ❌ | 需要 ncurses 库,未集成 |
| amidi | ✅ | ❌ | MIDI 工具,OH 不常用 |
| alsaloop | ✅ | ❌ | PCM 回环,调试用,优先级低 |
| alsaucm | ✅ | ❌ | 用例管理器,复杂度高 |
| alsatplg | ✅ | ❌ | 拓扑编译器,专业用途 |
| alsabat | ✅ | ❌ | 需要 fftw3 库,未集成 |
| axfer | ✅ | ❌ | 音频传输工具,aplay 增强版 |
| iecset | ✅ | ❌ | IEC958 设置工具,专业用途 |
| aplaymidi | ✅ | ❌ | MIDI 播放,专业用途 |
| aseqdump | ✅ | ❌ | MIDI 调试,专业用途 |
| aseqnet | ✅ | ❌ | MIDI 网络工具,专业用途 |
| alsaconf | ✅ | ❌ | Shell 脚本,不适合 OH |
| alsa-info | ✅ | ❌ | 诊断脚本,不适合 OH |
| nhlt | ✅ | ❌ | NHLT 工具,Intel 特定 |

### 4.4 裁剪统计

| 统计项 | 数值 |
|--------|------|
| 上游工具总数 | 19 |
| OH 编译工具数 | 5 |
| 裁剪工具数 | 14 |
| 裁剪率 | **74%** |

### 4.5 裁剪原因分类

| 原因类别 | 工具数 | 工具列表 |
|----------|--------|----------|
| **依赖缺失** | 3 | alsamixer (ncurses), alsabat (fftw3), ... |
| **专业用途** | 7 | amidi, iecset, aplaymidi, aseqdump, aseqnet, alsatplg, nhlt |
| **优先级低** | 2 | alsaloop, axfer |
| **不适合 OH** | 2 | alsaconf (shell), alsa-info (script) |

---

## 5. 与上游构建系统的差异

### 5.1 构建系统对比

| 维度 | 上游 (Autotools) | OH 适配 (GN) |
|------|------------------|--------------|
| **构建工具** | autoconf + automake + libtool | GN + Ninja |
| **配置方式** | `./configure` 运行时检测 | 硬编码在 BUILD.gn |
| **依赖检测** | 自动检测库和头文件 | 硬编码依赖路径 |
| **功能开关** | `--enable/disable-xxx` | 手动编辑 BUILD.gn |
| **条件编译** | `AM_CONDITIONAL` | 手动排除子目录 |
| **路径配置** | `--with-xxx-dir` | 硬编码常量 |
| **systemd 集成** | `HAVE_SYSTEMD` | 不支持 |
| **NLS 支持** | `ENABLE_NLS` | 禁用 |

### 5.2 configure.ac vs BUILD.gn

#### 5.2.1 上游: configure.ac (494 行)

```ac
# 功能检测
AM_CONDITIONAL(ALSAMIXER, test "x$enable_alsamixer" = "xyes")
AM_CONDITIONAL(ALSALOOP, test "x$enable_alsaloop" = "xyes")
AM_CONDITIONAL(BAT, test "x$enable_bat" = "xyes")

# 库检测
PKG_CHECK_MODULES([ALSA], [alsa >= 1.1.6])
AC_CHECK_LIB([fftw3f], [fftwf_execute], ...)

# 头文件检测
AC_CHECK_HEADER([alsa/mixer.h], ...)
AC_CHECK_HEADER([curses.h], ...)

# 功能检测
AC_CHECK_FUNCS([clock_gettime], ...)
```

#### 5.2.2 OH: BUILD.gn (178 行)

```gn
# 硬编码依赖
deps = [ "../alsa-lib:libasound" ]

# 硬编码编译选项
cflags = [
  "-DHAVE_CONFIG_H",
  "-D_GNU_SOURCE",
  ...
]

# 手动选择编译目标
ohos_executable("aplay") { ... }
# (仅选择需要的工具,不编译其他)
```

### 5.3 Makefile.am vs BUILD.gn

#### 5.3.1 上游: Makefile.am

```make
# 自动化构建
bin_PROGRAMS = alsactl alsamixer amixer aplay arecord ...

alsactl_SOURCES = alsactl/alsactl.c alsactl/clean.c ...
alsactl_LDADD = @ALSA_LIBS@ @CURSES_LIBS@

# 条件编译
if ALSAMIXER
bin_PROGRAMS += alsamixer
alsamixer_SOURCES = alsamixer/...
alsamixer_LDADD = @ALSA_LIBS@ @CURSES_LIBS@
endif
```

#### 5.3.2 OH: BUILD.gn

```gn
# 显式指定每个目标
ohos_executable("alsactl") {
  sources = [
    "alsactl/alsactl.c",
    "alsactl/clean.c",
    ...
  ]
  deps = [ "../alsa-lib:libasound" ]
}

# 仅编译需要的工具,不添加 alsamixer
```

### 5.4 配置文件对比

| 文件 | 上游用途 | OH 用途 |
|------|----------|---------|
| `configure.ac` | 动态检测配置 | 不使用 |
| `Makefile.am` | 自动化构建规则 | 参考 (手动转换为 GN) |
| `include/aconfig.h` | configure 生成 | 直接使用 (预生成) |
| `include/config.h.in` | 配置模板 | 不使用 |

---

## 6. OH 模板属性说明

### 6.1 常用属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `subsystem_name` | string | 所属子系统 |
| `part_name` | string | 部件名称 |
| `install_enable` | bool | 是否安装到镜像 |
| `install_images` | list | 安装到哪些分区 |
| `symlink_target_name` | list | 符号链接目标名称 |

### 6.2 属性使用示例

#### 6.2.1 安装到 system 分区

```gn
ohos_executable("amixer") {
  ...
  install_enable = true          # 启用安装
  install_images = ["system"]    # 安装到 system 分区
}
```

**说明**:
- `install_enable = true`: 启用安装
- `install_images = ["system"]`: 安装到 `/system/bin/`

#### 6.2.2 创建符号链接

```gn
ohos_executable("aplay") {
  ...
  symlink_target_name = [ "arecord" ]
}
```

**说明**:
- 创建 `/system/bin/arecord` -> `/system/bin/aplay` 符号链接
- aplay 和 arecord 功能相同,链接即可

#### 6.2.3 子系统和部件

```gn
ohos_executable("amixer") {
  ...
  subsystem_name = "thirdparty"  # thirdparty 子系统
  part_name = "alsa-utils"      # alsa-utils 部件
}
```

**说明**:
- 用于 OH 部件管理和模块化编译
- bundle.json 中定义了相同的 subsystem 和 part_name

### 6.3 install_enable 对比

| 工具 | install_enable | 原因 |
|------|----------------|------|
| aplay | ✅ true | 核心工具,必须安装 |
| amixer | ✅ true | 核心工具,必须安装 |
| alsactl | ✅ true | 配置工具,必须安装 |
| speaker-test | ❌ false | 调试工具,可选安装 |
| aconnect | ❌ false | MIDI 工具,可选安装 |

---

## 7. 编译流程

### 7.1 编译入口

#### 7.1.1 方法 1: 直接编译

```bash
./build.sh --product-name [PRODUCT_NAME] --ccache \
    --build-target third_party/alsa-utils:alsa-utils
```

**说明**:
- 直接编译 alsa-utils 组
- 自动编译依赖的 alsa-lib

#### 7.1.2 方法 2: 在 bundle.json 中添加

```json
{
  "component": {
    "name": "alsa-utils",
    "build": {
      "sub_component": [
        "//third_party/alsa-utils:alsa-utils"
      ]
    }
  }
}
```

**说明**:
- 添加到组件的编译子部件
- 编译整个产品时会自动编译

### 7.2 编译依赖顺序

```
1. alsa-lib (必须先编译)
   ↓
2. alsa-utils
   ├── aconnect
   ├── amixer
   ├── aplay
   ├── speaker-test
   └── alsactl
```

### 7.3 编译产物

| 工具 | 输出位置 | 是否安装 |
|------|----------|----------|
| aconnect | `out/.../aconnect` | ❌ |
| amixer | `out/.../amixer` + `/system/bin/amixer` | ✅ |
| aplay | `out/.../aplay` + `/system/bin/aplay` | ✅ |
| arecord | `/system/bin/arecord` (symlink) | ✅ |
| speaker-test | `out/.../speaker-test` | ❌ |
| alsactl | `out/.../alsactl` + `/system/bin/alsactl` | ✅ |

---

## 8. 总结

### 8.1 关键适配点总结

| 适配点 | OH 方式 | 优势 |
|--------|---------|------|
| **构建系统** | GN/Ninja | 快速,跨平台 |
| **功能选择** | 手动编辑 BUILD.gn | 精确控制 |
| **依赖管理** | 硬编码依赖 | 简单直接 |
| **路径配置** | 编译器宏定义 | 灵活可配置 |
| **警告抑制** | 大量 `-Wno-*` | 适配 ALSA 代码风格 |

### 8.2 与上游差异总结

| 差异项 | 影响 |
|--------|------|
| **无动态检测** | 需要手动维护依赖列表 |
| **无功能开关** | 需要手动编辑 BUILD.gn |
| **硬编码路径** | 灵活性降低,但更简单 |
| **裁剪 74% 工具** | 减小体积,但功能减少 |
| **无 Patch** | 升级简单,维护成本低 |

### 8.3 优势与挑战

| 优势 | 挑战 |
|------|------|
| ✅ 构建简单快速 | ⚠️ 手动维护依赖列表 |
| ✅ 无 Patch 冲突 | ⚠️ 功能裁剪较多 |
| ✅ 易于升级 | ⚠️ 缺少上游动态检测 |
| ✅ 维护成本低 | ⚠️ 配置不够灵活 |

---

## 9. 附录

### 9.1 相关文档

- [02_Patches.md](02_Patches.md) - Patch 分析 (无 Patch)
- [01_Overview.md](01_Overview.md) - 原始库简介
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用场景

### 9.2 参考命令

```bash
# 编译整个组
./build.sh --product-name ohos-arm64 --ccache \
    --build-target third_party/alsa-utils:alsa-utils

# 编译单个工具
./build.sh --product-name ohos-arm64 --ccache \
    --build-target third_party/alsa-utils:aplay

# 查看构建依赖
gn desc out/ohos-arm64 //third_party/alsa-utils:alsa-utils deps
```

### 9.3 构建脚本示例

```bash
#!/bin/bash
# build_alsa_utils.sh

PRODUCT_NAME="rk3568"
BUILD_TARGET="third_party/alsa-utils:alsa-utils"

echo "Building alsa-utils for ${PRODUCT_NAME}..."

./build.sh \
    --product-name ${PRODUCT_NAME} \
    --ccache \
    --build-target ${BUILD_TARGET} \
    || { echo "Build failed!"; exit 1; }

echo "Build success!"
echo "Output: out/${PRODUCT_NAME}/system/bin/aplay"
echo "         out/${PRODUCT_NAME}/system/bin/amixer"
echo "         out/${PRODUCT_NAME}/system/bin/alsactl"
```

---

**最后更新**: 2026-02-08
