# ncurses OpenHarmony 构建适配

## 构建系统概述

### 与典型 OH 库的差异

| 特性 | 典型 OH 第三方库 | ncurses |
|------|-----------------|---------|
| **构建系统** | GN (BUILD.gn) | Autotools (configure + make) |
| **配置文件** | BUILD.gn, bundle.json | ncurses.spec, configure.in |
| **依赖声明** | GN deps | Makefile 依赖 |
| **跨平台** | GN 内置支持 | 通过 Patch 添加 ohos 支持 |

### 构建流程

ncurses 在 OpenHarmony 中使用**传统的 autotools 构建流程**：

```
┌─────────────────────────────────────────────────────────┐
│                   ncurses 构建流程                       │
├─────────────────────────────────────────────────────────┤
│  1. 应用 Patch                                          │
│     ├── ncurses-kbs.patch                               │
│     ├── ncurses-urxvt.patch                             │
│     ├── ncurses-libs.patch                              │
│     ├── ncurses-config.patch                            │
│     ├── cross_compile_support_ohos.patch                │
│     └── backport-0002-CVE-2023-29491-env-access.patch   │
│                                                          │
│  2. 运行 configure                                      │
│     └── 检测 ohos 平台（通过 Patch 添加支持）            │
│                                                          │
│  3. 运行 make                                           │
│     └── 编译库和工具                                     │
│                                                          │
│  4. 安装                                                │
│     └── 生成 terminfo 数据库和库文件                     │
└─────────────────────────────────────────────────────────┘
```

---

## 构建配置文件

### ncurses.spec

`ncurses.spec` 是 OpenHarmony 构建系统的 RPM spec 格式配置文件，定义了：

```spec
name:          ncurses
Version:       6.5
Summary:       Terminal control library
License:       MIT
URL:           https://invisible-mirror.net/archives/ncurses/ncurses-6.5.tar.gz

Patch8:        ncurses-config.patch
Patch9:        ncurses-libs.patch
Patch11:       ncurses-urxvt.patch
Patch12:       ncurses-kbs.patch
Patch15:       backport-0002-CVE-2023-29491-env-access.patch
# OHOS_LOCAL
Patch99:       cross_compile_support_ohos.patch
```

**关键说明**:
- `Patch99` 明确标记为 `# OHOS_LOCAL`，表示这是 OpenHarmony 特有的 Patch
- Patch 编号不连续（8, 9, 11, 12, 15, 99），表明可能存在历史 Patch 已被移除或合并

### configure 脚本

ncurses 使用标准的 GNU autoconf 生成的 `configure` 脚本。主要配置选项：

| 选项 | 说明 | OH 构建中可能使用 |
|------|------|-------------------|
| `--prefix` | 安装前缀 | 是 |
| `--with-shared` | 构建共享库 | 是 |
| `--with-normal` | 构建静态库 | 是 |
| `--with-cxx` | 构建 C++ 库 | 是 |
| `--with-cxx-shared` | 构建 C++ 共享库 | 可能 |
| `--with-termlib` | 独立 terminfo 库 | 可能 |
| `--enable-widec` | 宽字符支持 | 建议启用 |
| `--with-strip-program` | strip 工具路径 | **OH 特有** |

### 交叉编译支持

`cross_compile_support_ohos.patch` 为 ncurses 添加了 OpenHarmony 交叉编译支持：

#### config.sub 修改

添加 `ohos` 到有效操作系统列表：

```bash
case $os in
    ... | ironclad* | ohos* )
        ;;
esac
```

添加 ohos 内核-OS-对象处理：

```bash
case $kernel-$os-$obj in
    ...
    *-ohos*-)
        ;;
esac
```

#### configure 修改

添加 `--with-strip-program` 选项：

```bash
if test "${with_strip_program+set}" = set; then
  INSTALL_OPT_S="$INSTALL_OPT_S --strip-program=$with_strip_program"
fi
```

这使得交叉编译时可以指定目标平台的 strip 工具。

---

## 关键编译选项

### 从 Patch 推断的 OH 特定配置

#### 1. 库链接参数 (ncurses-libs.patch)

**修改前**:
```makefile
SHLIB_LIST = $(SHLIB_DIRS) -lncurses@USE_LIB_SUFFIX@ @SHLIB_LIST@
LDFLAGS = $(TEST_ARGS) @LDFLAGS@ \
    @LD_MODEL@ $(TEST_LIBS) @LIBS@ $(CXXLIBS)
```

**修改后**:
```makefile
SHLIB_LIST = $(SHLIB_DIRS) -lncurses@USE_LIB_SUFFIX@ #@SHLIB_LIST@
LDFLAGS = @LDFLAGS@ @LD_MODEL@ @LIBS@ $(CXXLIBS)
```

**OH 适配目的**:
- 禁用 `@SHLIB_LIST@` 中的额外系统库链接
- 移除测试相关的链接参数
- 简化依赖关系，适配 OH 的库结构

#### 2. 配置脚本输出 (ncurses-config.patch)

**修改前**:
```bash
for opt in -L$libdir @EXTRA_PKG_LDFLAGS@ @LIBS@
```

**修改后**:
```bash
for opt in -L$libdir @LIBS@
```

**OH 适配目的**:
- 移除 `@EXTRA_PKG_LDFLAGS@` 中的额外链接标志
- 避免输出与 OH 构建系统冲突的硬编码路径

---

## 与上游构建系统的差异

### 主要差异点

| 方面 | 上游 ncurses | OpenHarmony 适配 |
|------|-------------|------------------|
| **目标平台** | 通用 Unix/Linux | 添加 ohos 支持 |
| **配置脚本** | 标准输出 | 简化输出，移除硬编码路径 |
| **库链接** | 完整系统库链接 | 精简链接参数 |
| **交叉编译** | 标准支持 | 增强的交叉编译支持 |
| **安装路径** | 标准 FHS | 适配 OH 目录结构 |

### 为什么需要这些差异

#### 1. 简化库链接

OpenHarmony 使用不同于传统 Linux 的库结构：
- 可能使用不同的 C 库实现
- 库路径结构与传统 Linux 不同
- 需要避免循环依赖

#### 2. 配置脚本适配

OH 的构建流程可能：
- 使用 sysroot 进行交叉编译
- 需要避免绝对路径硬编码
- 使用不同的 pkg-config 路径

#### 3. 平台支持

OpenHarmony 是新兴操作系统：
- 需要显式添加到 config.sub
- 需要特定的工具链配置

---

## 构建输出

### 生成的库文件

| 库名称 | 说明 |
|--------|------|
| `libncurses.so` | 核心 curses 共享库 |
| `libncurses.a` | 核心 curses 静态库 |
| `libncursesw.so` | 宽字符版本共享库 |
| `libncursesw.a` | 宽字符版本静态库 |
| `libncurses++.so` | C++ 封装共享库 |
| `libncurses++.a` | C++ 封装静态库 |
| `libform.so/a` | 表单库 |
| `libmenu.so/a` | 菜单库 |
| `libpanel.so/a` | 面板库 |

### 生成的工具

| 工具 | 功能 |
|------|------|
| `tic` | terminfo 编译器 |
| `infocmp` | terminfo 数据库比较 |
| `captoinfo` | termcap 转换器 |
| `tput` | 终端能力查询 |
| `tset`/`reset` | 终端初始化 |
| `clear` | 清屏 |
| `toe` | 终端类型列表 |
| `tabs` | 制表位设置 |

### terminfo 数据库

构建会生成二进制 terminfo 数据库：
- 终端类型定义（如 xterm, rxvt, linux 等）
- 存储在 `share/terminfo/` 目录下
- 按终端类型首字母分目录存储

---

## 构建建议

### 交叉编译示例

```bash
# 配置 - 交叉编译到 OpenHarmony
./configure \
    --host=arm-linux-ohos \
    --prefix=/system \
    --with-shared \
    --with-normal \
    --with-cxx \
    --enable-widec \
    --with-strip-program=llvm-strip

# 编译
make -j$(nproc)

# 安装到 sysroot
make install DESTDIR=$SYSROOT
```

### 推荐配置选项

针对 OpenHarmony 的推荐配置：

```bash
./configure \
    --prefix=/system \
    --with-shared \
    --with-normal \
    --with-cxx \
    --with-cxx-shared \
    --enable-widec \
    --enable-pc-files \
    --with-pkg-config-libdir=/system/lib/pkgconfig \
    --with-terminfo-dirs=/system/share/terminfo \
    --with-default-terminfo-dir=/system/share/terminfo
```

### 常见问题

#### 1. configure 失败：无法识别 ohos 平台

**原因**: 未应用 `cross_compile_support_ohos.patch`

**解决**: 确保所有 Patch 已正确应用

#### 2. 链接错误：找不到系统库

**原因**: ncurses-libs.patch 禁用了某些系统库链接

**解决**: 检查 OH 环境中是否确实需要这些库

#### 3. terminfo 数据库路径错误

**原因**: 默认路径与 OH 目录结构不匹配

**解决**: 使用 `--with-terminfo-dirs` 指定正确路径

---

## 总结

ncurses 在 OpenHarmony 中的构建适配通过以下方式实现：

1. **spec 文件**: 定义 Patch 应用顺序和构建配置
2. **Patch 修改**: 适配配置脚本、库链接和平台支持
3. **交叉编译**: 添加 ohos 平台识别和工具链支持

这些适配确保 ncurses 能够在 OpenHarmony 的构建环境中正确编译和运行，同时保持与上游代码的兼容性。
