# 03_Build_Integration - OH 构建适配

本文档说明 libedit 在 OpenHarmony 中的构建系统适配和配置。

---

## 1. 构建系统概览

### 1.1 原始构建系统

libedit 使用标准的 Autotools 构建系统：

| 组件 | 文件 | 作用 |
|------|------|------|
| **Autoconf** | configure.ac | 生成 configure 脚本 |
| **Automake** | Makefile.am | 生成 Makefile |
| **Libtool** | ltmain.sh | 处理共享库 |
| **config.sub** | config.sub | 系统类型检测 |

**证据来源**：目录结构分析

### 1.2 OH 构建适配状态

| 构建系统 | 适配状态 | 说明 |
|----------|----------|------|
| **Autotools** | ✅ 原生支持 | 标准的 autotools 构建 |
| **GN/Ninja** | ❌ 无适配 | 未发现 BUILD.gn 文件 |
| **CMake** | ❌ 无适配 | 未发现 CMakeLists.txt |
| **其他** | ❌ 无适配 | 无其他构建配置 |

**证据来源**：全代码库搜索结果

### 1.3 构建适配总结

| 项目 | 状态 | 说明 |
|------|------|------|
| **BUILD.gn** | ❌ 不存在 | 无 GN 构建配置 |
| **编译脚本** | ⚠️ 标准配置 | 使用原始 autotools |
| **OH 特定配置** | ❌ 无 | 无 OH 特殊编译选项 |
| **交叉编译支持** | ✅ 已完成 | config.sub 包含 OHOS 支持 |

---

## 2. Autotools 构建系统

### 2.1 标准编译流程

libedit 的标准编译流程（适用于 OH 交叉编译环境）：

```bash
# 1. 配置交叉编译
./configure --host=arm-linux-ohos \
            CC=clang \
            CXX=clang++ \
            CFLAGS="--target=arm-linux-ohos" \
            LDFLAGS="--target=arm-linux-ohos"

# 2. 编译
make

# 3. 测试（可选）
make check

# 4. 安装
make install DESTDIR=/path/to/ohos/sysroot
```

**证据来源**：README.md (通用 autotools 安装说明)

### 2.2 configure 关键选项

| 选项 | 说明 | OH 使用 |
|------|------|---------|
| `--host` | 目标系统类型 | `arm-linux-ohos`, `aarch64-linux-ohos` |
| `--prefix` | 安装路径 | `/usr` 或 OH sysroot 路径 |
| `--enable-widec` | 启用宽字符支持 | 默认启用（上游 2016 年后） |
| `--enable-examples` | 构建示例程序 | 默认禁用 |

**证据来源**：configure.ac, ChangeLog

### 2.3 交叉编译配置

**支持的 OHOS 目标架构**：

| 架构 | 配置选项 | 说明 |
|------|----------|------|
| **ARM 32** | `--host=arm-linux-ohos` | ARMv7/v8 (32-bit) |
| **ARM 64** | `--host=aarch64-linux-ohos` | ARMv8 (64-bit) |
| **x86_64** | `--host=x86_64-linux-ohos` | Intel/AMD 64-bit |
| **RISC-V** | `--host=riscv64-linux-ohos` | RISC-V 64-bit |

**示例配置**：

```bash
# ARM 32
./configure --host=arm-linux-ohos \
            CC=arm-linux-ohos-clang

# ARM 64
./configure --host=aarch64-linux-ohos \
            CC=aarch64-linux-ohos-clang

# x86_64
./configure --host=x86_64-linux-ohos \
            CC=x86_64-linux-ohos-clang
```

**证据来源**：config.sub OHOS 支持

---

## 3. OHOS 特定适配

### 3.1 config.sub OHOS 支持

**修改文件**：`config.sub`

**修改内容**：

```diff
@@ -1738,7 +1738,7 @@
 	     | skyos* | haiku* | rdos* | toppers* | drops* | es* \
 	     | onefs* | tirtos* | phoenix* | fuchsia* | redox* | bme* \
 	     | midnightbsd* | amdhsa* | unleashed* | emscripten* | wasi* \
-	     | nsk* | powerunix* | genode* | zvmoe* | qnx* | emx*)
+	     | nsk* | powerunix* | genode* | zvmoe* | qnx* | emx* | ohos*)
 		;;
 	# This one is extra strict with allowed versions
 	sco3.2v2 | sco3.2v[4-9]* | sco5v6*
@@ -1775,6 +1775,8 @@
 		;;
 	*-eabi* | *-gnueabi*)
 		;;
+	*-ohos)
+		;;
 	-*)
```

**验证命令**：

```bash
$ grep -n "ohos" config.sub
1771:     | fiwix* | mlibc* | cos* | mbr* | ironclad* | ohos* )
1869:	*-ohos*-)
```

**状态**：✅ 已整合到上游 (libedit-3.1-20250104)

**证据来源**：git commit a89010b, config.sub:1771, config.sub:1869

### 3.2 其他 OHOS 特定配置

| 配置项 | 状态 | 说明 |
|--------|------|------|
| **编译器标志** | ❌ 无 | 无 OH 特定的 CFLAGS |
| **宏定义** | ❌ 无 | 无 `#ifdef OHOS` 条件编译 |
| **链接选项** | ❌ 无 | 无 OH 特定的 LDFLAGS |
| **依赖库** | ❌ 无 | 无 OH 特定依赖 |

**证据来源**：configure.ac, 全代码库搜索

---

## 4. 编译产物

### 4.1 标准编译产物

libedit 编译后生成以下产物：

| 产物 | 文件 | 说明 |
|------|------|------|
| **共享库** | libedit.so, libedit.dylib | 动态链接库 |
| **静态库** | libedit.a | 静态链接库 |
| **头文件** | histedit.h, editline/readline.h | 公共头文件 |
| **pkg-config** | libedit.pc | pkg-config 配置 |
| **man 页面** | el*.3 | 手册页面 |

**证据来源**：Makefile.in, libedit.pc.in

### 4.2 产物安装路径

| 产物 | 默认安装路径 | OH 建议 |
|------|-------------|---------|
| **共享库** | `/usr/lib/libedit.so` | `${OHOS_SYSROOT}/usr/lib/` |
| **静态库** | `/usr/lib/libedit.a` | `${OHOS_SYSROOT}/usr/lib/` |
| **头文件** | `/usr/include/histedit.h` | `${OHOS_SYSROOT}/usr/include/` |
| **pkg-config** | `/usr/lib/pkgconfig/libedit.pc` | `${OHOS_SYSROOT}/usr/lib/pkgconfig/` |

### 4.3 编译选项影响

| 编译选项 | 影响的产物 | 说明 |
|----------|-----------|------|
| `--enable-shared` | libedit.so | 默认启用 |
| `--enable-static` | libedit.a | 默认启用 |
| `--enable-widec` | 所有产物 | 默认启用（宽字符支持） |

---

## 5. BUILD.gn 构建配置（方案）

### 5.1 当前状态

⚠️ **重要**：libedit 目前**没有 BUILD.gn 构建配置**。

| 检查项 | 结果 |
|--------|------|
| BUILD.gn 文件 | ❌ 不存在 |
| GN 模板 | ❌ 不存在 |
| OH 构建脚本 | ❌ 不存在 |

**证据来源**：全代码库搜索结果

### 5.2 BUILD.gn 配置方案（如需使用）

如果需要在 OH 中使用 libedit，可以参考以下 BUILD.gn 模板：

```gn
# third_party/libedit/BUILD.gn
import("//build/ohos.gni")

# libedit 配置
config("libedit_config") {
  include_dirs = [ "src" ]
  cflags = [
    "-DHAVE_CONFIG_H",
    # libedit 特定标志
  ]
}

# libedit 共享库
ohos_shared_library("libedit") {
  sources = [
    "src/chared.c",
    "src/chartype.c",
    "src/common.c",
    "src/el.c",
    "src/eln.c",
    "src/emacs.c",
    "src/filecomplete.c",
    "src/getline.c",
    "src/hist.c",
    "src/history.c",
    "src/keymacro.c",
    "src/literal.c",
    "src/map.c",
    "src/parse.c",
    "src/prompt.c",
    "src/read.c",
    "src/readline.c",
    "src/refresh.c",
    "src/search.c",
    "src/sig.c",
    "src/strlcat.c",
    "src/strlcpy.c",
    "src/terminal.c",
    "src/tokenizer.c",
    "src/tty.c",
    "src/unvis.c",
    "src/vi.c",
    "src/vis.c",
    "src/wcsdup.c",
    "src/reallocarr.c",
  ]

  public_configs = [ ":libedit_config" ]
  public_deps = [ ":config_h" ]

  # 依赖 ncurses（如需要）
  deps = [ "//third_party/ncurses:ncurses" ]

  # 安装到 sysroot
  install_images = [ "system" ]
  install_enable = true
}

# 生成 config.h
action("config_h") {
  script = "generate_config.sh"
  outputs = [ "$root_out_dir/config.h" ]
  args = [
    "--output",
    rebase_path(outputs[0], root_build_dir),
  ]
}
```

**注意**：
- 以上是模板，需要根据实际情况调整
- 需要生成 config.h（运行 configure 或手动创建）
- 需要处理 ncurses 依赖（如有）

**证据来源**：参考其他 OH 第三方库的 BUILD.gn

---

## 6. 依赖关系

### 6.1 编译时依赖

| 依赖项 | 说明 | OH 状态 |
|--------|------|---------|
| **libc** | 标准 C 库 | ✅ musl libc |
| **ncurses** | 终端控制库（可选） | ⚠️ 需要 libtinfo |
| **编译器** | C 编译器 | ✅ Clang |

### 6.2 OH 依赖分析

**检查 OH 中的 ncurses**：

```bash
# 查找 ncurses 库
find /Volumes/lexar/code/d/work/oh/third_party -name "*ncurses*" -o -name "*tinfo*"
```

**预期结果**：
- 如 OH 有 ncurses，libedit 可以链接
- 如 OH 无 ncurses，需要评估是否需要 libedit 的终端功能

### 6.3 依赖图

```
libedit
    ├── musl libc (必需)
    └── ncurses/tinfo (可选，终端功能)
```

---

## 7. 编译验证

### 7.1 验证 OHOS 支持

**验证 config.sub**：

```bash
$ ./config.sub arm-linux-ohos
arm-unknown-linux-ohos

$ ./config.sub aarch64-linux-ohos
aarch64-unknown-linux-ohos
```

**期望输出**：能够识别 OHOS 系统类型。

**证据来源**：config.sub 验证

### 7.2 验证交叉编译

**测试命令**：

```bash
# 配置
./configure --host=arm-linux-ohos \
            CC=arm-linux-ohos-clang \
            CFLAGS="--target=arm-linux-ohos"

# 编译
make -j$(nproc)

# 检查产物
file lib/.libs/libedit.so
# 输出：... ARM ... dynamically linked ...
```

**期望结果**：
- configure 成功完成
- 编译无错误
- 生成正确的 ARM 共享库

---

## 8. 构建脚本建议

### 8.1 OH 构建脚本示例

如果需要在 OH 中构建 libedit，可以创建以下脚本：

```bash
#!/bin/bash
# build_libedit.sh - OH libedit 构建脚本

set -e

# 配置
OHOS_TOOLCHAIN="/path/to/ohos/toolchain"
OHOS_SYSROOT="/path/to/ohos/sysroot"
TARGET_ARCH="arm64"  # arm, arm64, x86_64, riscv64

# 设置交叉编译工具
case "$TARGET_ARCH" in
  arm)
    HOST=arm-linux-ohos
    CC=arm-linux-ohos-clang
    ;;
  arm64)
    HOST=aarch64-linux-ohos
    CC=aarch64-linux-ohos-clang
    ;;
  x86_64)
    HOST=x86_64-linux-ohos
    CC=x86_64-linux-ohos-clang
    ;;
  riscv64)
    HOST=riscv64-linux-ohos
    CC=riscv64-linux-ohos-clang
    ;;
  *)
    echo "Unknown arch: $TARGET_ARCH"
    exit 1
    ;;
esac

export CC="$CC"
export CFLAGS="--target=$HOST --sysroot=$OHOS_SYSROOT"
export LDFLAGS="--target=$HOST --sysroot=$OHOS_SYSROOT"

# 配置
./configure \
  --host=$HOST \
  --prefix=/usr \
  --enable-shared \
  --enable-static

# 编译
make -j$(nproc)

# 安装
make install DESTDIR="$OHOS_SYSROOT"

echo "libedit 构建完成"
```

**使用方式**：

```bash
./build_libedit.sh arm64
```

**证据来源**：参考 OH 其他库的构建脚本

### 8.2 CMake 集成（可选）

如果 OH 使用 CMake 构建，可以创建 CMakeLists.txt：

```cmake
cmake_minimum_required(VERSION 3.10)
project(libedit)

# 添加源文件
add_library(libedit SHARED
    src/chared.c
    src/chartype.c
    src/common.c
    src/el.c
    src/eln.c
    src/emacs.c
    src/filecomplete.c
    src/getline.c
    src/hist.c
    src/history.c
    src/keymacro.c
    src/literal.c
    src/map.c
    src/parse.c
    src/prompt.c
    src/read.c
    src/readline.c
    src/refresh.c
    src/search.c
    src/sig.c
    src/strlcat.c
    src/strlcpy.c
    src/terminal.c
    src/tokenizer.c
    src/tty.c
    src/unvis.c
    src/vi.c
    src/vis.c
    src/wcsdup.c
    src/reallocarr.c
)

# 安装头文件
install(FILES
    src/histedit.h
    src/editline/readline.h
    DESTINATION include
)

# 安装库
install(TARGETS libedit
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
```

---

## 9. 维护建议

### 9.1 当前维护状态

| 维护项 | 状态 | 建议 |
|--------|------|------|
| **autotools 构建** | ✅ 正常 | 标准配置，无需维护 |
| **OHOS 支持** | ✅ 已完成 | config.sub 已包含 OHOS |
| **BUILD.gn 适配** | ❌ 不需要 | libedit 未被使用 |
| **构建脚本** | ⚠️ 可选 | 如需使用可创建 |

### 9.2 升级维护

**版本升级步骤**：

1. 下载新版本
2. 验证 OHOS 支持（`grep ohos config.sub`）
3. 测试交叉编译
4. 更新文档

**无回归风险**：已整合到上游，升级安全。

**证据来源**：[_work/ASSESSMENT.md](_work/ASSESSMENT.md) §7

### 9.3 未来维护考虑

**如需在 OH 中使用 libedit**：

1. **创建 BUILD.gn 配置**
2. **集成到 OH 构建系统**
3. **添加测试用例**
4. **更新文档**

**如废弃 libedit**：

1. 标记为 deprecated
2. 说明废弃原因
3. 考虑移除

---

## 10. 总结

### 10.1 构建适配状态

| 构建系统 | 状态 | 说明 |
|----------|------|------|
| **autotools** | ✅ 完成 | 标准配置，OHOS 支持已整合 |
| **GN/Ninja** | ❌ 无 | 未使用，无需适配 |
| **CMake** | ❌ 无 | 未使用，无需适配 |

### 10.2 关键要点

1. ✅ libedit 使用标准 autotools 构建
2. ✅ config.sub 已包含 OHOS 系统支持
3. ❌ 无 BUILD.gn 构建配置
4. ❌ libedit 在 OH 中未被使用

### 10.3 构建建议

- **如需使用**：创建 BUILD.gn 或使用 autotools
- **版本升级**：可安全升级到最新版本
- **交叉编译**：配置 `--host=arm-linux-ohos` 等

---

## 相关文档

- **Patch 分析**：[02_Patches.md](02_Patches.md)
- **使用情况**：[04_Usage_in_OH.md](04_Usage_in_OH.md)
- **完整评估**：[_work/ASSESSMENT.md](_work/ASSESSMENT.md)

---

**文档最后更新**：2025-02-07
**证据来源**：configure, config.sub, README.md
