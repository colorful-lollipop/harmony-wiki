# OH 构建适配

## 3.1 构建系统概述

### 上游构建系统

exfatprogs 上游项目使用 **autotools** 作为构建系统：

| 文件 | 用途 |
|------|------|
| `configure.ac` | autoconf 主配置文件 |
| `Makefile.am` | automake 构建模板 |
| `autogen.sh` | 生成 configure 脚本 |

典型构建流程：
```bash
./autogen.sh
./configure
make
make install
```

### OpenHarmony 构建系统

OpenHarmony 使用 **GN (Generate Ninja)** 作为构建系统，exfatprogs 被重新配置为使用 GN 构建：

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | OH GN 构建主配置文件 |

构建产物安装到 `system` 分区，作为系统级库提供。

## 3.2 BUILD.gn 结构详解

### 顶层配置

```gn
import("//build/ohos.gni")

group("exfatprogs") {
  deps = [
    ":exfatlabel",
    ":fsck.exfat",
    ":mkfs.exfat",
  ]
}
```

- **group**：定义默认构建目标，包含所有命令行工具
- **不包含 libexfat**：libexfat 作为独立目标，需显式依赖

### 通用配置块

```gn
config("exfat-defaults") {
  cflags = [
    "-std=gnu99",              # 使用 GNU C99 标准
    "-Wno-error",              # 警告不视为错误，允许编译通过
    "-D_FILE_OFFSET_BITS=64",  # 大文件支持（64位文件偏移）
    "-DPACKAGE=\"exfatprogs\"", # 包名称定义
    "-DVERSION=\"1.2.5\",      # 版本号定义
  ]
  include_dirs = [
    "dump",
    "fsck",
    "include",
    "label",
    "mkfs",
    "tune",
    "exfat2img",
  ]
}
```

**配置说明**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `-std=gnu99` | C99 | 使用 C99 标准，启用 GNU 扩展 |
| `-Wno-error` | 警告处理 | 将警告视为信息而非错误 |
| `-D_FILE_OFFSET_BITS=64` | 特性 | 支持大于 4GB 的文件 |
| `-DPACKAGE` | 元数据 | 定义包名称 |
| `-DVERSION` | 元数据 | 定义版本号 |
| include_dirs | 路径 | 指定头文件搜索路径 |

### 核心库构建

```gn
ohos_shared_library("libexfat") {
  configs = [ ":exfat-defaults" ]
  sources = [
    "lib/exfat_dir.c",
    "lib/exfat_fs.c",
    "lib/libexfat.c",
  ]
  include_dirs = [ "./libexfat" ]
  deps = []
  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "exfatprogs"
  install_images = [ "system" ]
}
```

**libexfat 配置要点**：

| 配置项 | 值 | 说明 |
|--------|-----|------|
| ohos_shared_library | 库类型 | 编译为共享库 (.so) |
| sources | 源文件列表 | 核心库源文件 |
| install_enable | true | 启用安装到目标系统 |
| subsystem_name | thirdparty | 属于 thirdparty 子系统 |
| part_name | exfatprogs | 组件名称 |
| install_images | system | 安装到 system 分区 |

### 可执行文件构建

#### mkfs.exfat

```gn
ohos_executable("mkfs.exfat") {
  configs = [ ":exfat-defaults" ]
  sources = [
    "mkfs/mkfs.c",
    "mkfs/upcase.c",
  ]
  include_dirs = [
    "./lib",
    "./mkfs",
  ]
  deps = [ ":libexfat" ]
  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "exfatprogs"
  install_images = [ "system" ]
}
```

#### fsck.exfat

```gn
ohos_executable("fsck.exfat") {
  configs = [ ":exfat-defaults" ]
  sources = [
    "fsck/fsck.c",
    "fsck/repair.c",
  ]
  include_dirs = [
    "./lib",
    "./mkfs",
    "./fsck",
  ]
  deps = [ ":libexfat" ]
  install_enable = true
  subsystem_name = "thirdparty"
  part_name = "exfatprogs"
  install_images = [ "system" ]
}
```

#### 其他工具

| 目标 | 源文件 | 依赖 | 安装 |
|------|--------|------|------|
| dump.exfat | dump/dump.c | libexfat | 不安装 |
| exfatlabel | label/label.c | libexfat | 不安装 |
| tune.exfat | tune/tune.c | libexfat | 不安装 |
| exfat2img | exfat2img/exfat2img.c | libexfat | 不安装 |

## 3.3 构建产物

### 安装产物

| 产物 | 路径 | 类型 |
|------|------|------|
| libexfat.so | system/lib64/ | 共享库 |
| mkfs.exfat | system/bin/ | 可执行文件 |
| fsck.exfat | system/bin/ | 可执行文件 |

### 未安装产物

以下工具构建但未安装到 system 分区：

- dump.exfat
- exfatlabel
- tune.exfat
- exfat2img

如需使用这些工具，可通过修改 `install_enable = true` 启用安装。

## 3.4 与上游构建的差异

### 差异对比表

| 维度 | 上游构建 | OH 构建 |
|------|----------|----------|
| 构建系统 | autotools | GN |
| 库类型 | 静态/动态可选 | 仅共享库 |
| 安装路径 | /usr/local/* | system/ |
| 默认目标 | 所有工具 | 命令行工具 |
| 编译选项 | configure 动态指定 | BUILD.gn 硬编码 |

### 关键差异详解

#### 1. 构建系统重构

**上游**：
- 使用 autoconf 检测系统环境
- 根据检测结果生成 Makefile
- 支持跨平台配置

**OH**：
- 使用 GN 预定义构建配置
- 无运行时环境检测
- 针对 OH 环境优化

#### 2. 库类型固定

**上游**：
```bash
./configure --enable-shared --enable-static  # 可选
```

**OH**：
- 仅编译为共享库 (`ohos_shared_library`)
- 便于其他模块动态链接
- 减少最终镜像大小

#### 3. 安装路径差异

**上游**：
```
/usr/local/bin/mkfs.exfat
/usr/local/bin/fsck.exfat
/usr/local/lib/libexfat.so
```

**OH**：
```
system/bin/mkfs.exfat
system/bin/fsck.exfat
system/lib64/libexfat.so
```

## 3.5 依赖关系

### 内部依赖

```
libexfat (基础库)
    │
    ├──► mkfs.exfat (依赖 libexfat)
    ├──► fsck.exfat (依赖 libexfat)
    ├──► dump.exfat (依赖 libexfat)
    ├──► exfatlabel (依赖 libexfat)
    ├──► tune.exfat (依赖 libexfat)
    └──► exfat2img (依赖 libexfat)
```

### 外部依赖

**libexfat 无外部依赖**：
- 不依赖 glibc 特定功能
- 不依赖 Linux 特有系统调用
- 使用标准 POSIX 接口

## 3.6 构建配置自定义

### 添加新的构建目标

如需添加新的可执行文件，可参考以下模板：

```gn
ohos_executable("new_tool") {
  configs = [ ":exfat-defaults" ]
  sources = [
    "path/to/source.c",
  ]
  include_dirs = [
    "./lib",
    "./path",
  ]
  deps = [ ":libexfat" ]
  # 可选配置
  # install_enable = true
  # subsystem_name = "thirdparty"
  # part_name = "exfatprogs"
}
```

### 启用调试构建

```gn
config("exfat-debug") {
  # 继承 exfat-defaults
  configs = [ ":exfat-defaults" ]
  # 添加调试选项
  cflags += [
    "-g",
    "-O0",
    "-DDEBUG",
  ]
}

ohos_shared_library("libexfat") {
  # 替换配置
  configs = [ ":exfat-debug" ]
  # ...
}
```

### 添加编译宏

如需启用特定功能，可添加编译宏：

```gn
config("exfat-extended") {
  configs = [ ":exfat-defaults" ]
  cflags += [
    "-DENABLE_EXTENDED_FEATURES",
    "-DOH_SPECIFIC_OPTION",
  ]
}
```

## 3.7 常见构建问题

### 问题一：头文件未找到

**症状**：
```
fatal error: exfat_ondisk.h: No such file or directory
```

**解决方案**：
确保 include_dirs 包含所有必要的目录：

```gn
config("exfat-defaults") {
  include_dirs = [
    "include",
    "lib",  # 添加缺失的目录
    # ...
  ]
}
```

### 问题二：符号未定义

**症状**：
```
undefined reference to `exfat_mount'
```

**解决方案**：
确认目标依赖 libexfat：

```gn
ohos_executable("my_tool") {
  deps = [ ":libexfat" ]  # 添加此依赖
  # ...
}
```

### 问题三：大文件支持失效

**症状**：
无法正确处理大于 4GB 的文件

**解决方案**：
验证 `_FILE_OFFSET_BITS=64` 宏已定义：

```gn
config("exfat-defaults") {
  cflags = [
    "-D_FILE_OFFSET_BITS=64",  # 确认此行存在
    # ...
  ]
}
```

## 3.8 版本升级构建适配

### 升级步骤

1. **下载新版本源码**
   ```bash
   wget https://github.com/exfatprogs/exfatprogs/archive/refs/tags/1.2.6.tar.gz
   tar -xzf 1.2.6.tar.gz
   ```

2. **替换源码目录**
   ```bash
   rm -rf exfatprogs/*
   cp -r exfatprogs-1.2.6/* exfatprogs/
   ```

3. **更新 BUILD.gn 版本号**
   ```gn
   config("exfat-defaults") {
     cflags = [
       # ...
       "-DVERSION=\"1.2.6\"",  # 更新版本号
       # ...
     ]
   }
   ```

4. **验证构建**
   ```bash
   hb build ...
   ```

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 新增源文件 | 新版本添加了功能文件 | 在 BUILD.gn sources 中添加 |
| 移除源文件 | 新版本移除了旧功能 | 从 BUILD.gn sources 中移除 |
| API 变更 | 函数签名或行为变化 | 更新调用代码 |
| 新增依赖 | 新功能需要新依赖 | 在 deps 中添加 |

## 3.9 构建系统维护建议

### 最佳实践

1. **保持配置简洁**：避免不必要的编译选项
2. **文档化变更**：每次 BUILD.gn 修改需记录原因
3. **定期同步**：上游 autotools 更新时检查是否需要同步 GN 配置
4. **测试验证**：重大修改后执行完整构建测试

### 代码审查清单

- [ ] 新增目标符合 part_name 命名规范
- [ ] 依赖关系正确无循环
- [ ] include_dirs 无冗余路径
- [ ] install_images 配置正确
- [ ] subsystem_name 归属正确
