# Python 3.11.4 OH 构建适配

## 构建系统概述

Python 在 OpenHarmony 中保持其原生构建系统，不使用 GN/Ninja。该库使用传统的 autotools 构建流程：

```
configure → make → make install
```

### 与 OH 构建系统的关系

```
┌────────────────────────────────────────────────────────────┐
│                   OpenHarmony 构建系统                      │
│                         (GN/Ninja)                         │
└────────────────────────────┬───────────────────────────────┘
                             │
                             │ 使用 Python 作为构建工具
                             ▼
┌────────────────────────────────────────────────────────────┐
│               Python 3.11.4 (third_party/python)            │
│                    (Autotools/Configure)                    │
│                                                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │ configure.ac│ → │  configure  │ → │   Makefile  │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│         ↑                                              │
│         └──────────────────────────────────────────────┘
│                          autoreconf
└────────────────────────────────────────────────────────────┘
```

## 关键配置文件

### 1. configure.ac - 自动配置脚本源

**OHOS 相关修改:**

#### 目标平台检测

```m4
# 在 configure.ac 中，根据编译器预定义宏检测 OHOS 目标

cat > conftest.c <<EOF
...
# elif defined(__aarch64__) && defined(__AARCH64EL__)
#  if defined(__ILP32__)
        aarch64_ilp32-linux-gnu
#  elif defined(__OHOS__)              # ← OHOS 特定
#        aarch64-linux-ohos            # ← OHOS 目标三元组
#  else
        aarch64-linux-gnu
...
# elif defined(__ARM_EABI__) && !defined(__ARM_PCS_VFP)
#  if defined(__ARMEL__) && !defined(__OHOS__)  # ← OHOS 特定
        arm-linux-gnueabi
#  elif defined(__OHOS__)                        # ← OHOS 目标
#        arm-linux-ohos
...
EOF
```

#### MULTIARCH 检测修复

```m4
AC_MSG_CHECKING([for multiarch])
AS_CASE([$ac_sys_system],
  [Darwin*], [MULTIARCH=""],
  [FreeBSD*], [MULTIARCH=""],
  [MULTIARCH=$($CC $CFLAGS --print-multiarch 2>/dev/null)]  # ← 添加 $CFLAGS
)
```

**OH 价值**: 确保交叉编译时 MULTIARCH 路径正确，考虑编译器标志影响。

### 2. setup.py - 模块构建配置

**OHOS 禁用模块:**

```python
# setup.py 第 65 行附近
# This global variable is used to hold the list of modules to be disabled.
DISABLED_MODULE_LIST = ['_uuid', '_socket', 'zlib', '_ctypes', 'binascii']
```

**编译器路径调整:**

```python
def configure_compiler(self):
    if not CROSS_COMPILING:
        add_dir_to_list(self.compiler.library_dirs, '/usr/local/lib')
        add_dir_to_list(self.compiler.include_dirs, '/usr/local/include')
        self.add_multiarch_paths()  # ← 移动到交叉编译分支外
    # only change this for cross builds for 3.3, issues on Mageia
    if CROSS_COMPILING:
        self.add_cross_compiling_paths()
    # self.add_multiarch_paths()  # ← 原位置，已移动
    self.compiler.add_library('python%s' % sys.version.split()[0][:4])  # ← 添加 Python 库链接
```

### 3. config.sub - 目标规范识别

**OHOS 支持添加:**

```bash
# config.sub 第 1748 行附近
case $os in
     | skyos* | haiku* | ... | zephyr* | ohos*)  # ← 添加 | ohos*
    ;;
```

**内核-OS 组合支持:**

```bash
case $kernel-$os in
    ...
    *-ohos*)  # ← 添加 *-ohos* 组合
        ;;
```

## 构建流程

### 标准构建步骤

```bash
# 1. 获取代码
git clone https://gitee.com/openharmony/third_party_python.git
cd third_party_python

# 2. 生成配置脚本 (可选，预生成文件通常已存在)
autoreconf -if  # 如果需要重新生成 configure

# 3. 配置
./configure \
    --host=aarch64-linux-ohos \
    --build=x86_64-linux-gnu \
    --prefix=/path/to/install

# 4. 编译
make -j$(nproc)

# 5. 安装
make install
```

### OH 交叉编译配置

针对 OpenHarmony 的交叉编译示例:

```bash
# 配置环境变量
export CC=aarch64-linux-ohos-clang
export CXX=aarch64-linux-ohos-clang++
export AR=aarch64-linux-ohos-ar
export RANLIB=aarch64-linux-ohos-ranlib

# 运行配置
./configure \
    --host=aarch64-linux-ohos \
    --build=x86_64-linux-gnu \
    --target=aarch64-linux-ohos \
    --prefix=/usr \
    --exec-prefix=/usr \
    --without-ensurepip \
    --disable-shared \
    LDFLAGS="-L/path/to/ohos/sysroot/usr/lib"
```

## 与上游构建系统的差异

| 方面 | 上游 Python | OHOS 适配版本 |
|-----|------------|--------------|
| 目标平台 | linux-gnu, darwin, win32 | 增加 linux-ohos |
| 禁用模块 | 无 | _uuid, _socket, zlib, _ctypes, binascii |
| MULTIARCH | $CC --print-multiarch | $CC $CFLAGS --print-multiarch |
| 库链接 | 自动检测 | 显式添加 libpythonX.Y |

## 预构建 Python (prebuilts)

OH 使用预构建的 Python 工具链作为构建依赖:

```
prebuilts/python_llvm/
├── darwin-arm64/
├── darwin-x86/
├── linux-x86/
└── windows-x86/
```

**BUILD.gn 引用示例:**

```gn
# interface/sdk_c/third_party/musl/ndk_script/BUILD.gn
prebuilts_python = "//prebuilts/python_llvm"

# Darwin ARM64
args += [ "-p" ] + [ rebase_path("${prebuilts_python}/darwin-arm64") ]

# Linux x86
args += [ "-p" ] + [ rebase_path("${prebuilts_python}/linux-x86") ]
```

## 特殊处理说明

### 1. 模块禁用原因

| 模块 | 禁用原因 | 可能影响 |
|-----|---------|---------|
| `_uuid` | 依赖 libuuid | UUID 生成功能不可用 |
| `_socket` | TODO: 需确认 | 网络编程相关功能受限 |
| `zlib` | 版本冲突风险 | 压缩/解压需使用外部工具 |
| `_ctypes` | TODO: 需确认 | C 库调用接口不可用 |
| `binascii` | TODO: 需确认 | 二进制/ASCII 转换功能受限 |

### 2. 路径处理差异

MinGW Patch 引入的路径处理改进:

- **MSYSTEM 环境支持**: 自动检测 MSYS2/MinGW 环境
- **路径分隔符动态切换**: 根据环境使用 `/` 或 `\`
- **Unix 风格路径**: 在 Windows 上支持 POSIX 风格路径

### 3. 交叉编译注意事项

1. **必须设置 host**: `--host=aarch64-linux-ohos`
2. **CFLAGS 传递**: MULTIARCH 检测需要 CFLAGS 环境变量
3. **库路径**: 需要显式指定 OHOS sysroot 库路径
4. **禁用模块**: 部分模块在交叉编译时自动禁用

## 回归测试

### 构建测试

```bash
# 1. 清理
make distclean

# 2. 重新配置
./configure --host=aarch64-linux-ohos ...

# 3. 编译
make -j$(nproc)

# 4. 验证模块
./python -c "import sys; print(sys.version)"
./python -c "import _uuid"  # 应失败（预期）
```

### 升级测试清单

- [ ] configure.ac 修改正确应用
- [ ] setup.py 禁用模块列表保留
- [ ] config.sub 保留 ohos 支持
- [ ] 交叉编译成功
- [ ] 基本功能测试通过

## 参考文档

- [Python 构建指南](https://devguide.python.org/setup/)
- [Autotools 交叉编译](https://www.gnu.org/software/automake/manual/html_node/Cross_002dCompilation.html)
- [OpenHarmony 构建系统](https://gitee.com/openharmony/build)
