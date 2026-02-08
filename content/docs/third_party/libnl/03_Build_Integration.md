# 03 - OpenHarmony 构建适配

本文档说明 libnl 在 OpenHarmony 中的构建系统集成方式。

## 构建系统概述

OpenHarmony 使用 **GN (Generate Ninja)** 构建系统，与 libnl 上游使用的 **GNU Autotools** 不同。因此需要进行构建适配。

## BUILD.gn 结构

### 文件位置
```
third_party/libnl/BUILD.gn
```

### 整体结构

```gn
# 1. 版权头
# 2. 条件导入
# 3. 安装脚本执行
# 4. Public Config 定义
# 5. Flex/Bison Action 规则
# 6. ohos_shared_library 目标
```

## 关键配置详解

### 1. 安装脚本执行

```gn
libnl_path = rebase_path("//third_party/libnl")
exec_script("install.sh", [ "$libnl_path" ])
```

**作用**: 在构建前执行 `install.sh`，完成以下工作：
1. 解压源码包 `libnl-libnl3_11_0.tar.gz`
2. 重命名为 `libnl` 目录
3. 运行 `./autogen.sh && ./configure`
4. 应用 Patch `solve-oh-compile-problem3_11_0.patch`

### 2. Public Config

```gn
config("libnl_share_public_config") {
  include_dirs = [ "//third_party/libnl/libnl/include/" ]
}
```

**作用**: 定义对外暴露的头文件路径，供依赖模块使用。

### 3. Flex/Bison 语法生成

libnl 使用 Flex 和 Bison 处理路由规则语法，需要特殊构建规则：

#### build_grammar (pktloc 词法分析)
```gn
action("build_grammar") {
  script = "/usr/bin/env"
  outputs = [ "$target_out_dir/gen/lib/route/pktloc_grammar.c" ]
  args = [
    "flex",
    "--header-file=$grammer_hh",
    "-o", "pktloc_grammar.c",
    "libnl/lib/route/pktloc_grammar.l",
  ]
}
```

#### pktloc_syntax (pktloc 语法分析)
```gn
action("pktloc_syntax") {
  script = "/usr/bin/env"
  outputs = [ "$target_out_dir/gen/lib/route/pktloc_syntax.c" ]
  args = [
    "bison",
    "-y", "-d",
    "-o", "pktloc_syntax.c",
    "libnl/lib/route/pktloc_syntax.y",
  ]
}
```

#### ematch_grammar 和 ematch_syntax
类似地处理 ematch（扩展匹配）语法。

### 4. 共享库目标

```gn
ohos_shared_library("libnl_share") {
  branch_protector_ret = "pac_ret"  // ARM64 分支保护
  
  // 包含生成的语法文件路径
  grammer_outputs = get_target_outputs(":build_grammar")
  grammer_path = get_path_info(grammer_outputs[0], "dir")
  
  // 头文件路径
  include_dirs = [
    "libnl",
    "libnl/include",
    "libnl/lib",
    "libnl/lib/route/cls",
    "libnl/lib/route",
    // ... 生成的文件路径
  ]
  
  // 对外暴露的配置
  public_configs = [ ":libnl_share_public_config" ]
  
  // 源文件列表（约 150+ 个文件）
  sources = [
    "libnl/lib/addr.c",
    "libnl/lib/attr.c",
    // ...
  ]
  
  // 依赖的 Action 目标
  deps = [
    ":build_grammar",
    ":ematch_grammar",
    ":ematch_syntax",
    ":pktloc_syntax",
  ]
  
  // 宏定义
  defines = [ "NL_DEBUG" ]
  
  // 编译选项
  cflags = [
    "-Wno-error",
    "-D_BSD_SOURCE",
    "-D_GNU_SOURCE",
    "-DNL_DEBUG",
    "-DSYSCONFDIR=\"/etc/libnl\"",
    "-D_NL_SYSCONFDIR_LIBNL=\"/usr/local/ect/libnl\"",
    "-D_NL_PKGLIBDIR==\"/usr/local/lib/libnl\"",
  ]
  
  // 安装到 system 和 updater 镜像
  install_images = [ "system", "updater" ]
  
  // 标记为芯片 SDK 内部 API
  innerapi_tags = [ "chipsetsdk" ]
}
```

## install.sh 详解

### 文件内容

```bash
#!/bin/bash
set -e
cd $1
touch test.lock
(
    flock -x 180
    if [ -d "libnl" ];then
        rm -rf libnl
    fi
    tar xvf libnl-libnl3_11_0.tar.gz
    mv libnl-libnl3_11_0 libnl
    cd $1/libnl
    ./autogen.sh
    ./configure
    patch -p1 < $1/solve-oh-compile-problem3_11_0.patch --fuzz=0 --no-backup-if-mismatch
    exit 0
)180>test.lock
```

### 工作流程

```
┌─────────────────┐
│  开始构建 libnl  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 执行 install.sh  │
└────────┬────────┘
         ▼
┌─────────────────┐     ┌─────────────────┐
│ 获取文件锁       │──NO──→ 等待锁释放     │
│ (flock -x 180)  │     └─────────────────┘
└────────┬────────┘
      YES│
         ▼
┌─────────────────┐
│ 清理旧源码       │
│ rm -rf libnl    │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 解压源码包       │
│ tar xvf ...     │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 重命名目录       │
│ mv libnl-xxx    │
│    libnl        │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 运行 autogen.sh  │
│ (生成 configure) │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 运行 configure   │
│ (生成 config.h)  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ 应用 OH Patch    │
│ patch -p1 < ...  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ GN 继续构建      │
│ (编译 .c 文件)   │
└─────────────────┘
```

### 锁机制说明

使用 `flock` 实现进程间互斥：
- `180` 是文件描述符（使用 test.lock）
- `-x` 表示排他锁
- 防止并行构建时解压冲突

## 与上游构建系统的差异

| 方面 | 上游 (Autotools) | OpenHarmony (GN) |
|------|-----------------|------------------|
| **构建工具** | autoconf, automake, libtool | GN + Ninja |
| **配置方式** | `./configure` 运行时检测 | 静态配置在 BUILD.gn 中 |
| **依赖处理** | pkg-config | GN deps |
| **语法生成** | 内置 flex/bison 支持 | 显式 Action 规则 |
| **库类型** | 支持静态/动态/模块 | 仅动态库 (ohos_shared_library) |
| **安装路径** | `make install` 指定 | `install_images` 指定 |

## 特殊处理说明

### 1. 禁用动态库加载

上游 libnl 支持运行时加载扩展模块（通过 `dlopen`），但 OH 版本禁用了此功能：

```c
// src/lib/utils.c 中被注释掉
// snprintf(path, sizeof(path), "%s/%s/%s.so", ...)
```

**原因**: OpenHarmony 的沙箱环境限制了动态库加载。

### 2. 配置路径硬编码

```gn
cflags = [
  "-DSYSCONFDIR=\"/etc/libnl\"",
  "-D_NL_SYSCONFDIR_LIBNL=\"/usr/local/ect/libnl\"",
]
```

**说明**: 这些路径在运行时实际不可用，libnl 在 OH 中以库形式被调用，不读取配置文件。

### 3. 调试支持

```gn
defines = [ "NL_DEBUG" ]
```

启用调试输出，便于开发阶段排查问题。

## 构建依赖

构建 libnl 需要以下工具：

| 工具 | 用途 | 是否必需 |
|------|------|---------|
| `flex` | 词法分析器生成 | 是 |
| `bison` | 语法分析器生成 | 是 |
| `autoconf` | 生成 configure | 是 |
| `automake` | 生成 Makefile.in | 是 |
| `libtool` | 库管理 | 是 |

## 升级维护指南

### 升级上游版本步骤

1. **替换源码包**:
   ```bash
   # 下载新版本 tar.gz
   mv libnl-new.tar.gz third_party/libnl/
   # 更新 install.sh 中的文件名
   ```

2. **更新 Patch**:
   ```bash
   # 对比新旧版本差异
   diff -urN libnl-old libnl-new
   # 更新 solve-oh-compile-problem3_11_0.patch
   ```

3. **更新 BUILD.gn**:
   - 检查源文件列表是否有新增/删除
   - 更新 `sources` 数组

4. **验证构建**:
   ```bash
   hb build //third_party/libnl:libnl_share
   ```

5. **测试依赖模块**:
   - 构建 `wpa_supplicant`
   - 构建 `drivers/peripheral/wlan`
   - 运行 WLAN 功能测试
