# libinput OpenHarmony 构建集成

> OH 如何构建 libinput，以及与上游构建系统的差异

---

## 📚 文档说明

**上游构建系统**：Meson
**OH 构建系统**：GN (Generate Ninja)

**主要差异**：
- OH 使用 GN 替代 Meson
- OH 使用 patch 机制动态修改源代码
- OH 添加了额外的安全编译选项
- OH 集成了 hilog 日志系统

---

## 🏗 构建系统对比

### 上游构建 (Meson)

#### 构建配置文件

| 文件 | 大小 | 用途 |
|------|------|------|
| `meson.build` | 34,339 行 | 主构建配置 |
| `meson_options.txt` | - | 编译选项 |

#### 典型构建命令

```bash
# 配置
meson setup builddir

# 编译
ninja -C builddir

# 安装
ninja -C builddir install
```

#### 构建特性

- 跨平台支持（Linux、FreeBSD）
- 自动依赖检测（libudev、libevdev、mtdev）
- 可选特性（调试工具、文档）
- 多后端支持（libudev、直接设备访问）

---

### OpenHarmony 构建 (GN)

#### 构建配置文件

| 文件 | 大小 | 用途 |
|------|------|------|
| `BUILD.gn` | 530 行 | 主构建配置 |
| `third_libinput.gni` | 24 行 | GN 变量定义 |
| `patch/BUILD.gn` | 82 行 | patch 应用配置 |
| `patch/apply_patch.sh` | 85 行 | patch 应用脚本 |

#### 典型构建命令

```bash
# 在 OH 根目录执行
./build.sh --product-name <product> --ccache

# 或直接使用 gn
gn gen out/ohos-arm-release
ninja -C out/ohos-arm-release
```

#### 构建特性

- GN 构建系统（Chromium/Flutter 风格）
- Patch 机制（构建时应用修改）
- 安全编译选项（CFI、PAC_RET）
- hilog 日志集成
- 笔设备特性开关

---

## 🔧 OH 构建配置详解

### 3.1 主 BUILD.gn

#### 目录结构

```gn
import("//build/ohos.gni")
import("third_libinput.gni")

defines = third_input_default_defines
gen_dst_dir = root_out_dir + "/diff_libinput_mmi"

config("libinput-third_config") {
  # 配置定义
}

config("libinput-third_public_config") {
  # 公共配置
}
```

#### 主要构建目标

##### 1. ohos_shared_library("libinput-third-mmi")

**产物**：共享库 `libinput-third-mmi.so`

**用途**：主要的输入处理库，供其他 OH 模块链接使用

**配置**：
```gn
ohos_shared_library("libinput-third-mmi") {
  sources = []  # 源文件来自 patch 生成
  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  configs = [ ":libinput-third_config" ]
  public_configs = [ ":libinput-third_public_config" ]
  deps = [ ":patch_gen_libinput-third-mmi" ]
  public_external_deps = [
    "input:mmi_libudev",
    "libevdev:libevdev",
    "mtdev:libmtdev-third-mmi",
  ]
  external_deps = [ "hilog:libhilog" ]
  license_file = "${third_libinput_root_path}/COPYING"
  part_name = "libinput"
  subsystem_name = "thirdparty"
}
```

**安全编译选项**：
- **PAC_RET**：指针认证返回地址保护（ARM64 特性）
- **CFI**：控制流完整性保护
- **CFI Cross DSO**：跨 DSO 的 CFI 保护

**依赖**：
- **内部依赖**：`mmi_libudev`（OH udev 实现）
- **公共外部依赖**：`libevdev`、`libmtdev-third-mmi`
- **外部依赖**：`libhilog`（OH 日志系统）

##### 2. ohos_source_set("patch_gen_libinput-third-mmi")

**产物**：源代码集合（由 patch 生成）

**用途**：收集所有 patch 后的源文件，作为共享库的输入

**源文件列表**（共 36 个）：
```gn
sources = [
  root_out_dir + "/diff_libinput_mmi/src/evdev-debounce.c",
  root_out_dir + "/diff_libinput_mmi/src/evdev-fallback.c",
  root_out_dir + "/diff_libinput_mmi/src/evdev-joystick.c",
  root_out_dir + "/diff_libinput_mmi/src/evdev-privacy-switch.c",
  // ... 32 个其他源文件
]
```

##### 3. 工具程序（7 个）

| 工具 | BUILD 目标 | 安装 | 用途 |
|------|-----------|------|------|
| **libinput-debug-mmi** | `ohos_executable` | ✅ | 事件调试工具 |
| **libinput-list-mmi** | `ohos_executable` | ✅ | 列出设备工具 |
| **libinput-tablet-mmi** | `ohos_executable` | ✅ | 平板调试工具 |
| **libinput-record-mmi** | `ohos_executable` | ✅ | 事件记录工具 |
| **libinput-analyze-mmi** | `ohos_executable` | ✅ | 分析工具 |
| **libinput-measure-mmi** | `ohos_executable` | ✅ | 测量工具 |
| **libinput-quirks-mmi** | `ohos_executable` | ✅ | 设备特性工具 |

**示例配置**：
```gn
ohos_executable("libinput-debug-mmi") {
  install_enable = true
  sources = []
  configs = [ ":libinput-third_config" ]
  public_configs = [ ":libinput-third_public_config" ]
  deps = [
    ":libinput-third-mmi",
    ":patch_gen_libinput-debug",
  ]
  public_external_deps = [
    "input:mmi_libudev",
    "libevdev:libevdev",
    "mtdev:libmtdev-third-mmi",
  ]
  part_name = "input"
  subsystem_name = "multimodalinput"
}
```

---

### 3.2 third_libinput.gni

**GN 变量定义文件**

```gn
third_libinput_root_path = rebase_path(".")

declare_args() {
  libinput_feature_pen = false
}

third_input_default_defines = []

if (libinput_feature_pen) {
  third_input_default_defines += [ "OHOS_BUILD_ENABLE_PEN" ]
}
```

**功能**：
- 定义 `libinput_feature_pen` 编译参数
- 当启用时，添加 `OHOS_BUILD_ENABLE_PEN` 宏定义
- 允许条件编译笔设备相关功能

---

### 3.3 patch/BUILD.gn

**Patch 应用配置**

```gn
import("//build/ohos.gni")

gen_src_dir = "//third_party/libinput"
gen_dst_dir = root_out_dir + "/diff_libinput_mmi"
patches_root_dir = gen_src_dir + "/patch"
build_gn_dir = "$patches_root_dir/diff_libinput_mmi/libinput"

action("apply_patch") {
  visibility = [ "*" ]
  script = "${gen_src_dir}/patch/apply_patch.sh"
  inputs = [ "$gen_src_dir" ]
  outputs = [
    "$gen_dst_dir/src/filter.c",
    "$gen_dst_dir/src/libinput.c",
    // ... 42 个输出文件
  ]
  args = [
    rebase_path(gen_src_dir, root_build_dir),
    rebase_path(gen_dst_dir, root_build_dir),
    rebase_path(build_gn_dir, root_build_dir),
  ]
}
```

**工作流程**：
1. 删除旧的输出目录（如果存在）
2. 创建新的输出目录
3. 复制源文件到输出目录
4. 应用 patch（`libinput_0000.diff`）
5. 生成 patch 后的源文件

---

### 3.4 patch/apply_patch.sh

**Patch 应用脚本**

```bash
#!/bin/bash
set -e

curdir=$(pwd)
source_dir=$1
out_dir=$2
path_file_dir=$3

echo "curdir: $curdir"
echo "source_dir: $source_dir"
echo "out_dir: $out_dir"
echo "path_file_dir: $path_file_dir"

# 删除旧输出
if [ -d "$out_dir" ]; then
    echo "remove $out_dir begin"
    rm -rf "$out_dir"
fi

# 创建输出目录
echo "mkdir out_dir: $out_dir"
mkdir -p $out_dir

# 复制源文件
echo "cp $source_dir/* to $out_dir/"
cp -fra $source_dir/* $out_dir

# 应用 patch
PATCH_FILE=$(realpath $(ls $path_file_dir/*.diff | tail -n 1))

echo "PATCH_FILE: $PATCH_FILE"

cd $out_dir
echo "pwd: $(pwd)"
patch -p1 -i $PATCH_FILE
if [ $? -ne 0 ]; then
    echo "patch fail. path_file_dir=$path_file_dir"
    exit 1
fi

cd $curdir
exit 0
```

**关键参数**：
- `$1` (source_dir)：源代码目录 (`third_party/libinput`)
- `$2` (out_dir)：输出目录 (`$root_out_dir/diff_libinput_mmi`)
- `$3` (path_file_dir)：patch 文件目录 (`patch/diff_libinput_mmi/libinput`)

**patch 选项**：
- `-p1`：剥离 1 级目录前缀
- `-i`：指定 patch 文件

---

## 📦 输出目录结构

### 生成目录

```
$root_out_dir/diff_libinput_mmi/
├── src/                          # patch 后的源文件（36 个）
│   ├── evdev.c
│   ├── evdev-joystick.c
│   ├── evdev-privacy-switch.c
│   ├── libinput.c
│   └── ...
├── include/                      # 头文件
│   ├── config.h
│   ├── libinput.h
│   └── ...
├── export_include/               # 公共 API 头文件
│   └── libinput.h
└── hm_src/                      # OH 特有实现
    ├── hm_missing.h
    └── hm_missing.c
```

### 构建产物

**共享库**：
```
out/ohos-arm-release/
└── libinput-third-mmi.so
```

**工具程序**（安装到系统）：
```
/system/bin/
├── libinput-debug-mmi
├── libinput-list-mmi
├── libinput-tablet-mmi
├── libinput-record-mmi
├── libinput-analyze-mmi
├── libinput-measure-mmi
└── libinput-quirks-mmi
```

**配置文件**（通过 prebuild_libinput 安装）：
```
/sys_prod/etc/libinput/quirks/
├── 10-generic-keyboard.quirks
├── 10-generic-lid.quirks
├── 10-generic-trackball.quirks
├── 30-vendor-*.quirks      (15 个厂商配置)
└── 50-system-*.quirks       (13 个系统配置)
```

---

## ⚙ 编译选项详解

### 5.1 核心编译标志

#### libinput-third_config

```gn
config("libinput-third_config") {
  visibility = [ ":*" ]

  include_dirs = [
    "$gen_dst_dir/src",
    "$gen_dst_dir/include",
    "$gen_dst_dir/hm_src",
  ]

  cflags = [
    "-Wno-unused-parameter",
    "-Wno-implicit-int",
    "-Wno-return-type",
    "-Wno-unused-function",
    "-Wno-string-conversion",
    "-DHAVE_LIBINPUT_LOG_CONSOLE_ENABLE",  # 启用控制台日志
    "-DHAVE_LIBINPUT_LOG_ENABLE",          # 启用日志
  ]
}
```

**cflags 说明**：
- `-Wno-*`：抑制特定的编译警告
- `-DHAVE_LIBINPUT_LOG_*`：启用 libinput 日志系统

**include_dirs**：
- `$gen_dst_dir/src`：内部源文件
- `$gen_dst_dir/include`：库头文件
- `$gen_dst_dir/hm_src`：OH 特有实现

#### libinput-third_public_config

```gn
config("libinput-third_public_config") {
  include_dirs = [
    "$gen_dst_dir/export_include",
    "$gen_dst_dir/include",
    "$gen_dst_dir/src",
  ]

  cflags = []
}
```

**include_dirs**：
- `$gen_dst_dir/export_include`：公共 API（导出给依赖者）
- `$gen_dst_dir/include`：内部 API
- `$gen_dst_dir/src`：源文件

---

### 5.2 安全编译选项

#### CFI (Control Flow Integrity)

```gn
sanitize = {
  cfi = true              # 控制流完整性
  cfi_cross_dso = true   # 跨 DSO 的 CFI
  debug = false
}
```

**作用**：
- **CFI**：在运行时验证间接调用的目标类型
- **CFI Cross DSO**：防止跨共享库的无效调用
- **目的**：防止控制流劫持攻击

**性能影响**：
- 增加编译时间
- 轻微增加运行时开销
- 显著提升安全性

#### PAC_RET (Pointer Authentication)

```gn
branch_protector_ret = "pac_ret"
```

**作用**：
- **PAC**：指针认证（ARM64 特性）
- **RET**：保护返回地址
- **目的**：防止返回地址覆盖攻击

**平台限制**：
- 仅在 ARM64 架构上有效
- 需要硬件支持

---

### 5.3 日志系统集成

#### hilog 依赖

```gn
external_deps = [ "hilog:libhilog" ]
```

**作用**：
- 集成 OH 的 hilog 日志系统
- 统一系统日志输出
- 支持日志分级和过滤

**日志宏**：
```c
-DHAVE_LIBINPUT_LOG_CONSOLE_ENABLE  // 启用控制台日志输出
-DHAVE_LIBINPUT_LOG_ENABLE         // 启用日志系统
```

**日志级别**：
- LOG_LEVEL_I：信息
- LOG_LEVEL_E：错误
- LOG_LEVEL_W：警告

---

### 5.4 特性编译选项

#### 笔设备支持

**GN 参数**：
```gn
declare_args() {
  libinput_feature_pen = false
}
```

**宏定义**：
```c
#ifdef OHOS_BUILD_ENABLE_PEN
// 笔设备相关代码
#endif // OHOS_BUILD_ENABLE_PEN
```

**启用方法**：
```bash
# 在 GN args 中添加
libinput_feature_pen = true

# 或在 build.sh 中
./build.sh --product-name <product> --args="libinput_feature_pen=true"
```

**作用**：
- 条件编译笔设备支持代码
- 减小不使用笔设备的系统大小
- 灵活的特性开关

---

## 🔗 依赖关系

### 外部依赖

| 依赖 | 版本 | 链接类型 | 用途 |
|------|--------|----------|------|
| **libevdev** | - | 公共 | 输入事件库（evdev 事件包装） |
| **mtdev** | 1.1.6 | 公共 | 多点触控协议库 |
| **hilog:libhilog** | - | 外部 | OH 日志系统 |
| **input:mmi_libudev** | - | 内部 | OH udev 实现 |

### 依赖分析

#### libevdev

**用途**：包装内核输入事件

**关键功能**：
- `libevdev_new()`：创建 libevdev 实例
- `libevdev_has_event_type()`：检查事件类型
- `libevdev_has_event_code()`：检查事件码
- `libevdev_get_event_value()`：获取事件值

**与 libinput 的关系**：
- libinput 通过 libevdev 访问内核输入事件
- libevdev 提供跨内核版本的抽象

#### mtdev

**用途**：多点触控协议处理

**关键功能**：
- 多点触控数据解析
- 触摸点坐标和压力处理
- 协议版本兼容性

**与 libinput 的关系**：
- libinput 使用 mtdev 处理多点触控设备
- 支持各种触摸板和触摸屏协议

#### hilog

**用途**：OH 统一日志系统

**关键功能**：
- 日志分级（DEBUG、INFO、WARN、ERROR）
- 日志过滤
- 日志持久化

**与 libinput 的关系**：
- libinput 通过 hilog 输出日志信息
- 支持日志系统级调试

---

## 🔄 构建流程

### 完整构建流程

```
1. gn gen out/ohos-arm-release
   ↓
2. 解析 BUILD.gn
   ↓
3. 执行 apply_patch action
   ├─ copy third_party/libinput/* → out/.../diff_libinput_mmi/
   ├─ apply libinput_0000.diff
   └─ 生成 patch 后的源文件
   ↓
4. 编译源文件 (ninja)
   ├─ 编译 src/*.c
   ├─ 编译 hm_src/*.c
   └─ 生成 libinput-third-mmi.so
   ↓
5. 链接依赖
   ├─ libevdev
   ├─ mtdev
   ├─ mmi_libudev
   └─ libhilog
   ↓
6. 安装产物
   ├─ libinput-third-mmi.so → /system/lib/
   ├─ 工具 → /system/bin/
   └─ quirks → /sys_prod/etc/libinput/quirks/
```

### 增量构建

```bash
# 首次构建
./build.sh --product-name <product>

# 增量构建（修改后）
ninja -C out/ohos-arm-release libinput

# 重新应用 patch（如修改了 patch）
rm -rf out/ohos-arm-release/diff_libinput_mmi
./build.sh --product-name <product>
```

---

## 🛠 开发和调试

### 开发环境设置

```bash
# 1. 配置 OH 开发环境
source build/envsetup.sh

# 2. 选择产品
hb set -p <product>

# 3. 构建
hb build -f //third_party/libinput:libinput-third-mmi
```

### 调试构建

```bash
# 启用调试符号
hb build -f //third_party/libinput:libinput-third-mmi --ccache --prefer-llvm
```

### 使用调试工具

**事件调试**：
```bash
# 实时显示输入事件
libinput-debug-mmi --verbose

# 监听特定设备
libinput-debug-mmi --device /dev/input/eventX
```

**事件记录**：
```bash
# 记录事件到文件
libinput-record-mmi --output events.dat

# 重放事件
libinput-debug-mmi --replay events.dat
```

**设备列表**：
```bash
# 列出所有输入设备
libinput-list-mmi

# 详细信息
libinput-list-mmi --verbose
```

---

## ⚠️ 常见构建问题

### 问题 1：Patch 应用失败

**症状**：
```
patch: ****: ****: No file to patch
patch fail. path_file_dir=...
```

**原因**：
- patch 文件路径错误
- 源文件已被修改

**解决方案**：
```bash
# 清理输出目录
rm -rf out/ohos-arm-release/diff_libinput_mmi

# 重新构建
./build.sh --product-name <product>
```

### 问题 2：依赖未找到

**症状**：
```
error: undefined reference to `libevdev_...`
```

**原因**：
- libevdev、mtdev 未构建
- 外部依赖路径错误

**解决方案**：
```bash
# 检查依赖是否构建
ls out/ohos-arm-release/lib*/libevdev*.so
ls out/ohos-arm-release/lib*/libmtdev*.so

# 先构建依赖
hb build -f //third_party/libevdev:libevdev
hb build -f //third_party/mtdev:libmtdev-third-mmi
```

### 问题 3：日志未输出

**症状**：
libinput 调试信息未显示

**原因**：
- hilog 日志未配置
- 日志级别不正确

**解决方案**：
```bash
# 启用 hilog
hdc shell hilog -R

# 检查日志
hdc shell hilog -T | grep libinput
```

---

## 📈 性能优化

### 编译优化

**默认优化级别**：`-O2`

**启用 LTO (Link Time Optimization)**：
```gn
cflags = [
  "-flto",
  "-ffat-lto-objects",
]
```

**作用**：
- 跨模块内联优化
- 减少代码大小
- 提升运行时性能

### 运行时性能

**libinput 性能优化**：
- 事件批处理
- 高效的指针加速算法
- 最小化内存分配

**监控工具**：
```bash
# 测量输入延迟
libinput-measure-mmi --delay

# 分析性能
libinput-analyze-mmi --performance
```

---

## 🔐 安全加固

### 编译时安全

**已启用**：
- ✅ CFI (控制流完整性)
- ✅ CFI Cross DSO
- ✅ PAC_RET (指针认证返回)
- ✅ Branch Protection

### 运行时安全

**建议措施**：
- 启用 ASLR (地址空间布局随机化）
- 限制文件权限
- 最小化特权操作

### 安全审计

**定期审计项**：
- 检查已知的 CVE
- 评估 patch 引入的新攻击面
- 审查输入事件处理逻辑

---

## 📚 相关文档

- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的使用情况

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
