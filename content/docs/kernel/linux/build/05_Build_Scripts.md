# 构建脚本

> **更新时间**: 2026-02-06

---

## 文档目的

本文档详细分析 OpenHarmony Linux Kernel 构建系统中的 Shell 脚本、Makefile 和 Python 脚本，包括其功能、参数、输入输出和依赖关系。

---

## Shell 脚本分析

### 1. build_kernel.sh

**文件路径**: `/Volumes/lexar/code/d/work/oh/kernel/linux/build/build_kernel.sh`
**行数**: 52
**目的**: 内核编译完成后，将内核镜像复制到 OpenHarmony 构建输出目录

#### 参数

| 参数 | 位置 | 说明 | 示例值 |
|------|------|--------|--------|
| `$1` | L19 | kernel_build_script_dir | `"//kernel/linux/build"` |
| `$2` | L23-24 | KERNEL_OBJ_TMP_PATH | `"out/kernel/OBJ/linux-5.10"` |
| `$3` | L25 | GN target output dir | `"out/xxx/packages/phone/images"` |
| `$4` | L24 | build_type | `"standard"` |
| `$5` | L24 | target_cpu | `"arm"`/`"arm64"`/`"riscv64"`/`"x86_64"`/`"loongarch64"` |
| `$6` | L24 | product_path | `"vendor/hisilicon/hispark_taurus"` |
| `$7` | L24 | device_name | `"hispark_taurus"` |
| `$8` | L24 | kernel_version | `"linux-5.10"` |

**代码证据**:
```bash
# build_kernel.sh:18-24
set -e

#$1 - kernel build script work dir
#$2 - kernel build script stage dir
#$3 - GN target output dir

echo build_kernel
pushd ${1}
./kernel_module_build.sh ${2} ${4} ${5} ${6} ${7} ${8}
```

#### 主要功能

1. **调用内核构建**: 调用 `kernel_module_build.sh` 执行实际编译

2. **清理时间戳**: 删除 `kernel.timestamp` 以触发完整构建

**代码证据**:
```bash
# build_kernel.sh:26
rm -rf ${3}/../../../kernel.timestamp
```

3. **复制内核镜像**: 根据架构复制对应的内核镜像

**代码证据**:
```bash
# build_kernel.sh:29-49
kernel_image=""
if [ "$5" == "arm" ];then
    kernel_image="uImage"
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/uImage ${3}/uImage
    # hispark_phoenix 特殊处理
    if [ "$7" == "hispark_phoenix"  ];then
        cp ${2}/kernel/OBJ/${8}/arch/arm/boot/dts/hi3751v350.dtb ${3}/dtbo.img
        cat ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage ${3}/dtbo.img > ${3}/zImage-dtb
    else
        cp ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage-dtb ${3}/zImage-dtb
    fi

elif [ "$5" == "arm64" ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm64/boot/Image ${3}/Image

elif [ "$5" == "riscv64" ];then
    cp ${2}/kernel/OBJ/${8}/arch/riscv/boot/Image ${3}/Image

elif [ "$5" == "loongarch64" ];then
    cp ${2}/kernel/OBJ/${8}/vmlinuz.efi ${3}/vmlinuz.efi

elif [ "$5" == "x86_64" ];then
    cp ${2}/kernel/OBJ/${8}/arch/x86/boot/bzImage ${3}/bzImage
fi
```

#### 输出文件

| 架构 | 设备 | 输出文件 | 源路径 |
|------|------|---------|--------|
| arm | 通用 | `uImage` | `arch/arm/boot/uImage` |
| arm | hispark_taurus | `zImage-dtb` | `arch/arm/boot/zImage-dtb` |
| arm | hispark_phoenix | `dtbo.img`, `zImage-dtb` | `arch/arm/boot/dts/hi3751v350.dtb`, `arch/arm/boot/zImage` |
| arm64 | - | `Image` | `arch/arm64/boot/Image` |
| riscv64 | - | `Image` | `arch/riscv/boot/Image` |
| x86_64 | - | `bzImage` | `arch/x86/boot/bzImage` |
| loongarch64 | - | `vmlinuz.efi` | `vmlinuz.efi` |

---

### 2. kernel_module_build.sh

**文件路径**: `/Volumes/lexar/code/d/work/oh/kernel/linux/build/kernel_module_build.sh`
**行数**: 72
**目的**: 内核构建的主入口脚本，设置环境变量并调用 kernel.mk

#### 参数

| 参数 | 位置 | 说明 | 示例值 |
|------|------|--------|--------|
| `$1` | L18 | OUT_DIR | `"out/rk3568"` |
| `$2` | L19 | BUILD_TYPE | `"standard"` 或 `"small"` |
| `$3` | L20 | KERNEL_ARCH | `"arm"`/`"arm64"`/`"riscv64"`/`"x86_64"`/`"loongarch64"` |
| `$4` | L21 | PRODUCT_PATH | `"vendor/hisilicon/hispark_taurus"` |
| `$5` | L22 | DEVICE_NAME | `"hispark_taurus"` 或 `"rk3568"` |
| `$6` | L23 | KERNEL_VERSION | `"linux-5.10"` 或 `"linux-4.19"` |

**代码证据**:
```bash
# kernel_module_build.sh:18-23
export OUT_DIR=$1
export BUILD_TYPE=$2
export KERNEL_ARCH=$3
export PRODUCT_PATH=$4
export DEVICE_NAME=$5
export KERNEL_VERSION=$6
```

#### 主要功能

1. **环境变量设置**: 导出所有构建相关环境变量

**代码证据**:
```bash
# kernel_module_build.sh:18-32
export OUT_DIR=$1
export BUILD_TYPE=$2
export KERNEL_ARCH=$3
export PRODUCT_PATH=$4
export DEVICE_NAME=$5
export KERNEL_VERSION=$6

export OHOS_ROOT_PATH=$(pwd)/../../..
```

2. **内核镜像类型判断**: 根据架构确定内核镜像名称

**代码证据**:
```bash
# kernel_module_build.sh:34-44
kernel_image=""
if [ "$KERNEL_ARCH" == "arm" ];then
    kernel_image="uImage"
elif [ "$KERNEL_ARCH" == "arm64" ];then
    kernel_image="Image"
elif [ "$KERNEL_ARCH" == "x86_64" ];then
    kernel_image="bzImage"
elif [ "$KERNEL_ARCH" == "loongarch64" ];then
    kernel_image="vmlinuz.efi"
fi
export KERNEL_IMAGE=${kernel_image}
```

3. **特殊设备处理**: 为 `hispark_phoenix` 设置 SDK 源码目录

**代码证据**:
```bash
# kernel_module_build.sh:46-48
if [ "$DEVICE_NAME" == "hispark_phoenix"  ];then
export SDK_SOURCE_DIR=${OHOS_ROOT_PATH}/device/soc/hisilicon/hi3751v350/sdk_linux/source
fi
```

4. **输出路径计算**: 根据 `BUILD_TYPE` 计算不同的输出路径

**代码证据**:
```bash
# kernel_module_build.sh:25-30
if [ "$BUILD_TYPE" == "small" ];then
    LINUX_KERNEL_OUT=${OUT_DIR}/kernel/${KERNEL_VERSION}
elif [ "$BUILD_TYPE" == "standard" ];then
    LINUX_KERNEL_OUT=${OUT_DIR}/kernel/src_tmp/${KERNEL_VERSION}
fi
LINUX_KERNEL_OBJ_OUT=${OUT_DIR}/kernel/OBJ/${KERNEL_VERSION}
```

5. **调用 Makefile**: 执行 `kernel.mk` 进行内核构建

**代码证据**:
```bash
# kernel_module_build.sh:58
make -f kernel.mk
```

6. **输出验证**: 检查内核镜像是否存在

**代码证据**:
```bash
# kernel_module_build.sh:60-65
if [ -f "${LINUX_KERNEL_IMAGE_FILE}" ];then
    echo "Image: ${LINUX_KERNEL_IMAGE_FILE} build success"
else
    echo "Image: ${LINUX_KERNEL_IMAGE_FILE} build failed!!!"
    exit 1
fi
```

7. **特殊设备输出**: 为 `hispark_taurus` 复制额外的输出

**代码证据**:
```bash
# kernel_module_build.sh:67-70
if [ "$5" == "hispark_taurus" ];then
    cp -rf ${LINUX_KERNEL_IMAGE_FILE} ${OUT_DIR}/uImage_${DEVICE_NAME}_smp
fi
```

#### 环境变量导出

| 变量名 | 值 | 用途 |
|--------|-----|--------|
| `OUT_DIR` | `$1` | 输出根目录 |
| `BUILD_TYPE` | `$2` | 构建变体 (standard/small) |
| `KERNEL_ARCH` | `$3` | 目标架构 |
| `PRODUCT_PATH` | `$4` | 产品配置路径 |
| `DEVICE_NAME` | `$5` | 设备名称 |
| `KERNEL_VERSION` | `$6` | 内核版本 |
| `OHOS_ROOT_PATH` | `$(pwd)/../../..` | OpenHarmony 根目录 |
| `KERNEL_IMAGE` | 根据架构确定 | 内核镜像文件名 |
| `LINUX_KERNEL_OUT` | 根据构建类型计算 | 内核源码输出目录 |
| `LINUX_KERNEL_OBJ_OUT` | `${OUT_DIR}/kernel/OBJ/${KERNEL_VERSION}` | 编译对象目录 |
| `LINUX_KERNEL_IMAGE_FILE` | 根据 `KERNEL_ARCH` 确定 | 最终内核镜像路径 |

---

### 3. check_build.sh

**文件路径**: `/Volumes/lexar/code/d/work/oh/kernel/linux/build/check_build.sh`
**行数**: 43
**目的**: 通过时间戳比较实现增量构建，避免不必要的完整重建

#### 参数

| 参数 | 位置 | 说明 | 示例值 |
|------|------|--------|--------|
| `$1` | L32 | kernel_dir | `kernel/linux/linux-5.10` |
| `$2` | L33 | output_image | `out/.../packages/phone/images/uImage` |
| `$3` | L34 | timestamp_file | `out/.../kernel.timestamp` |

**代码证据**:
```bash
# check_build.sh:32-34
echo $1 for check kernel dir
echo $2 for output image
echo $3 for timestamp
```

#### 主要功能

**递归时间戳检查**: 遍历内核源码目录，检查是否有文件比输出镜像新

**代码证据**:
```bash
# check_build.sh:18-30
function readfile () {
    for file in $1/*
    do
        if [ -d "$file" ];then
	    readfile $file $2 $3  # 递归处理子目录
        elif [ "$file" -nt "$2" ]; then  # $file 比 $2 新
            echo $file is update
            touch $3;  # 触发重建
            return
        fi
    done
}
```

**条件执行**:
1. 如果输出镜像存在，检查是否需要重建
2. 如果时间戳文件比输出镜像新，删除旧镜像
3. 如果输出镜像不存在，不做任何操作（由后续构建流程创建）

**代码证据**:
```bash
# check_build.sh:35-41
if [ -e "$2" ]; then
    readfile $1 $2 $3
    if [ "$3" -nt "$2" ]; then
        echo "need update $2"
        rm -rf $2;
    fi
fi
```

#### 工作流程

```
开始
  │
  ├─→ 输出镜像存在？
  │   │
  │   ├─→ 否 → 退出（由后续流程创建）
  │   │
  │   └─→ 是 → 递归遍历源码目录
  │           │
  │           ├─→ 遍历每个文件/目录
  │           │   │
  │           │   ├─→ 是目录 → 递归处理
  │           │   │
  │           │   └─→ 是文件 → 比较时间戳
  │           │       │
  │           │       ├─→ 文件 ≤ 镜像 → 继续
  │           │       │
  │           │       └─→ 文件 > 镜像 → touch $3，返回
  │           │
  │           └─→ 遍历结束？
  │               │
  │               ├─→ 时间戳 > 镜像 → 删除镜像
  │               └─→ 时间戳 ≤ 镜像 → 退出
  │
结束
```

---

### 4. kernel_build.py

**文件路径**: `/Volumes/lexar/code/d/work/oh/kernel/linux/build/kernel_build.py`
**行数**: 388
**目的**: Python CI 构建脚本，支持多架构内核编译和编译器警告解析

#### 主要类和函数

**Reporter 类** (L36-158):

**用途**: 解析编译器警告，生成构建报告

**关键方法**:

| 方法 | 行号 | 功能 |
|------|------|------|
| `__init__(self, arch, path)` | L37-39 | 初始化 Reporter，设置架构和路径 |
| `normpath(self, filename)` | L42-45 | 规范化文件路径，处理 `./` 前缀 |
| `format_title(self, title)` | L48-52 | 格式化标题，替换特殊引号 |
| `report_build_warning(self, filename, regex, details)` | L55-78 | 报告单个构建警告 |
| `parse_build_warning(self, blocks, regex, title_regex)` | L81-123 | 解析构建日志中的警告 |
| `parse(self, content)` | L126-158 | 解析所有构建警告 |

**代码证据**:
```python
# kernel_build.py:130-149
patterns = (
    ('[-Wunused-but-set-variable]', 'warning: .* set but not used'),
    ('[-Wunused-but-set-parameter]', 'warning: .* set but not used'),
    ('[-Wunused-const-variable=]', 'warning: .* defined but not used'),
    ('[-Wold-style-definition]', 'warning: .* definition'),
    ('[-Wmaybe-uninitialized]', 'warning: .* uninitialized'),
    ('[-Wtype-limits]', 'warning: .* always (false|true)'),
    ('[-Wunused-function]', 'warning: .* defined but not used'),
    ('[-Wsequence-point]', 'warning: .* may be undefined'),
    ('[-Wformat=]', 'warning: format.*'),
    ('[-Wunused-variable]', 'warning: [^\[]*'),
    ('[-Wframe-larger-than=]', 'warning: frame size [^\[]*'),
    ('[-Wshift-count-overflow]', 'warning: left shift count >= width of type'),
)
```

**已知忽略警告** (L24-33):

**代码证据**:
```python
# kernel_build.py:24-33
ignores = [
    "include/trace/events/eas_sched.h: warning: format '%d' expects argument of type 'int', but argument 9 has type 'long unsigned int' [-Wformat=]",
    "drivers/block/zram/zram_drv.c: warning: left shift count >= width of type",
    "include/trace/events/eas_sched.h: note: in expansion of macro",
    "mm/vmscan.c: warning: suggest parentheses around assignment used as truth value [-Wparentheses]",
]
```

#### 构建函数

**exec_cmd(command_list, shell=False, show_output=False, cwd=None)** (L161-213):

**用途**: 执行命令并捕获输出

**功能**:
- 添加 `nice` 前缀降低进程优先级
- 使用 `subprocess.Popen` 执行命令
- 使用 `epoll` 异步捕获 stdout 和 stderr
- 打印命令执行时间

**代码证据**:
```python
# kernel_build.py:166-169
command_list = ['nice'] + [str(s) for s in command_list]

print(f"cwd: '{cwd}'")
print(f"cmd: '{command_list}'")
start = time.time()
```

**make_cmd(cmd, arch, cross_compile, knl_path)** (L216-223):

**用途**: 执行 make 命令

**代码证据**:
```python
# kernel_build.py:217-218
make = f"{cmd} ARCH={arch} CROSS_COMPILE={cross_compile}"
outmsg, errmsg, ret = exec_cmd(make, cwd=knl_path)
```

**make_config(arch, config, cross_compile, knl_path)** (L226-233):

**用途**: 应用内核配置

**代码证据**:
```python
# kernel_build.py:227-228
make = f"make {config} ARCH={arch} CROSS_COMPILE={cross_compile}"
outmsg, errmsg, ret = exec_cmd(make, cwd=knl_path)
```

**make_j(arch, cross_compile, knl_path)** (L236-267):

**用途**: 并行编译内核

**功能**:
- 使用 `make -j{os.cpu_count()}` 并行编译
- 解析 stderr 中的警告
- 区分已知警告和新警告
- 返回适当的退出码（0=成功，1=失败，2=警告但可接受）

**代码证据**:
```python
# kernel_build.py:237
make = f'make -j{os.cpu_count()} ARCH={arch} CROSS_COMPILE={cross_compile}'

# kernel_build.py:242-264
elif len(errmsg) > 0:
    print(f'"{make}" warnings --> \n {errmsg}')
    result = "success"
    reporter = Reporter(arch, knl_path)
    known_issue = "\nKnown issue:\n"
    for report in reporter.parse(errmsg):
        if ignores and [i for i in ignores if re.match(i, report['title'])]:
            known_issue = known_issue + report['title'] + "\n"
            continue
        result = 'failed'

    print(known_issue)
    new_issue = "\nNew Issue:\n"
    if result == "failed":
        for report in reporter.parse(errmsg):
            if ignores and [i for i in ignores if re.match(i, report['title'])]:
                continue
            new_issue = new_issue + report['title'] + "\n"
            new_issue = new_issue + report['report'] + "\n"
        print(new_issue)
        return 2, f'"{make}" warning --> \n{new_issue}'
```

**cp_config(arch, config, config_path, knl_path)** (L270-281):

**用途**: 复制配置文件到内核源码

**代码证据**:
```python
# kernel_build.py:272-275
cp = f'cp ' + config_path.format(arch, config) + ' ' + os.path.join(knl_path, 'arch', arch, 'configs', config)
outmsg, errmsg, ret = exec_cmd(cp)
```

**build(arch, config, config_path, cross_compile, knl_path, logger)** (L297-356):

**用途**: 执行完整的内核构建流程

**构建步骤**:
1. `make defconfig`
2. `make oldconfig`
3. `make clean`
4. `make -j{cpu_count}` (第一次编译)
5. `cp_config` 复制配置
6. `make {config}` 应用配置
7. `make clean`
8. `make -j{cpu_count}` (第二次编译)
9. `make allmodconfig`
10. `sed` 修改 `CONFIG_FRAME_WARN=2048`
11. `make clean`
12. `make -j{cpu_count}` (第三次编译)

**代码证据**:
```python
# kernel_build.py:298-336
ret, msg = make_cmd('make defconfig', arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg

ret, msg = make_cmd('make oldconfig', arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg

ret, msg = make_cmd('make clean', arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg

ret, msg = make_j(arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg

ret, msg = cp_config(arch, config, config_path, knl_path)
if ret:
    logger.error(msg)
    return ret, msg
else:
    ret, msg = make_config(arch, config, cross_compile, knl_path)
    if ret:
        logger.error(msg)
        return ret, msg

ret, msg = make_cmd('make clean', arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg

ret, msg = make_j(arch, cross_compile, knl_path)
if ret:
    logger.error(msg)
    return ret, msg
```

**main() 函数** (L359-387):

**用途**: CI 构建入口点

**功能**:
- 构建 ARM 和 ARM64 内核
- 使用不同的 defconfig 配置
- 记录构建日志
- 汇总构建结果

**代码证据**:
```python
# kernel_build.py:367-377
arch = 'arm'
config = 'hispark_taurus_standard_defconfig'
cross_compile = '../../../prebuilts/gcc/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin/arm-linux-gnueabi-'
arm_ret, arm_msg = build(arch, config, config_path, cross_compile, knl_path, logger)

arch = 'arm64'
config = 'rk3568_standard_defconfig'
cross_compile = '../../../prebuilts/gcc/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-'
arm64_ret, arm64_msg = build(arch, config, config_path, cross_compile, knl_path, logger)

print(f'arm_ret: {arm_ret}, arm64_ret: {arm64_ret}')
if any([arm_ret, arm64_ret]):
    print('kernel build test failed!')
    exit(arm_ret or arm64_ret)

print('kernel build test success.')
exit(0)
```

---

## Makefile 分析

### kernel.mk

**文件路径**: `/Volumes/lexar/code/d/work/oh/kernel/linux/build/kernel.mk`
**行数**: 122
**目的**: 定义内核构建的核心规则，包括工具链配置、补丁应用、多架构支持

#### 核心变量

**源码和输出路径** (L17-32):

```makefile
PRODUCT_NAME=$(TARGET_PRODUCT)
OHOS_BUILD_HOME := $(realpath $(shell pwd)/../../../)
KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/${KERNEL_VERSION}
KERNEL_OBJ_TMP_PATH := $(OUT_DIR)/kernel/OBJ/${KERNEL_VERSION}

ifeq ($(BUILD_TYPE), standard)
    BOOT_IMAGE_PATH = $(OHOS_BUILD_HOME)/device/board/hisilicon/hispark_taurus/uboot/prebuilts
    KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/src_tmp/${KERNEL_VERSION}
    export KERNEL_SRC_DIR=out/KERNEL_OBJ/kernel/src_tmp/${KERNEL_VERSION}
endif

KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc
CLANG_HOST_TOOLCHAIN := $(OHOS_BUILD_HOME)/prebuilts/clang/ohos/linux-x86_64/llvm/bin
KERNEL_HOSTCC := $(CLANG_HOST_TOOLCHAIN)/clang
KERNEL_PREBUILT_MAKE := make
CLANG_CC := $(CLANG_HOST_TOOLCHAIN)/clang
```

**工具链配置** (L36-68):

```makefile
KERNEL_CROSS_COMPILE :=
ifeq ($(KERNEL_ARCH), arm)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/arm/gcc-linaro-7.5.0-arm-linux-gnueabi/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/arm-linux-gnueabi-
else ifeq ($(KERNEL_ARCH), arm64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/aarch64-linux-gnu-
else ifeq ($(KERNEL_ARCH), riscv64)
    PATH := $(CLANG_HOST_TOOLCHAIN):$(PATH)
    KERNEL_ARCH := riscv
    KERNEL_TARGET_TOOLCHAIN_PREFIX :=
    KERNEL_CROSS_COMPILE += LLVM=1
    KERNEL_CROSS_COMPILE += LLVM_IAS=1
else ifeq ($(KERNEL_ARCH), x86_64)
    KERNEL_TARGET_TOOLCHAIN := gcc
    KERNEL_TARGET_TOOLCHAIN_PREFIX :=
else ifeq ($(KERNEL_ARCH), loongarch64)
    KERNEL_TARGET_TOOLCHAIN := $(PREBUILTS_GCC_DIR)/linux-x86/loongarch64/gcc-loongarch64-linux-gnu/bin
    KERNEL_TARGET_TOOLCHAIN_PREFIX := $(KERNEL_TARGET_TOOLCHAIN)/loongarch64-unknown-linux-gnu-
    KERNEL_ARCH := loongarch
endif

ifneq ($(KERNEL_ARCH), loongarch)
KERNEL_CROSS_COMPILE += CC="$(CLANG_CC)"
endif

ifneq ($(KERNEL_ARCH), x86_64)
KERNEL_CROSS_COMPILE += CROSS_COMPILE="$(KERNEL_TARGET_TOOLCHAIN_PREFIX)"
endif

KERNEL_MAKE := \
    PATH="$(BOOT_IMAGE_PATH):$$PATH" \
    $(KERNEL_PREBUILT_MAKE)
```

**补丁和配置路径** (L75-83):

```makefile
ifneq ($(findstring $(BUILD_TYPE), small standard),)
DEVICE_PATCH_DIR := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}/$(DEVICE_NAME)_patch
DEVICE_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME).patch
PRODUCT_PATCH_FILE := $(OHOS_BUILD_HOME)/vendor/hisilicon/watchos/patches/$(DEVICE_NAME).patch
SMALL_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME)_$(BUILD_TYPE).patch
KERNEL_IMAGE_FILE := $(KERNEL_SRC_TMP_PATH)/arch/$(KERNEL_ARCH)/boot/$(KERNEL_IMAGE)
DEFCONFIG_FILE := $(DEVICE_NAME)_$(BUILD_TYPE)_defconfig
UNIFIED_COLLECTION_PATCH_FILE := ${OHOS_BUILD_HOME}/kernel/linux/common_modules/ucollection/apply_ucollection.sh

export KBUILD_OUTPUT=$(KERNEL_OBJ_TMP_PATH)
```

#### 构建规则

**主构建规则** (L86-122):

```makefile
$(KERNEL_IMAGE_FILE):
	$(hide) echo "build kernel..."
ifeq ($(DEVICE_NAME), hispark_phoenix)
	$(hide) rm -rf $(KERNEL_SRC_TMP_PATH);mkdir -p $(KERNEL_SRC_TMP_PATH);cp -arfP $(KERNEL_SRC_PATH)/* $(KERNEL_SRC_TMP_PATH)/
	$(hide) cd $(KERNEL_SRC_TMP_PATH)/drivers && rm -rf common && ln -s $(SDK_SOURCE_DIR)/common/drv ./common && cd -
	$(hide) cd $(KERNEL_SRC_TMP_PATH)/drivers && rm -rf msp && ln -s $(SDK_SOURCE_DIR)/msp/drv ./msp && cd -
else
	$(hide) rm -rf $(KERNEL_SRC_TMP_PATH);mkdir -p $(KERNEL_SRC_TMP_PATH);cp -arfL $(KERNEL_SRC_PATH)/* $(KERNEL_SRC_TMP_PATH)/
endif
	$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(KERNEL_PATCH_PATH) $(DEVICE_NAME)

ifeq ($(PRODUCT_PATH), vendor/hisilicon/watchos)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(PRODUCT_PATCH_FILE)
else
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && test -f $(DEVICE_PATCH_FILE) && patch -p1 < $(DEVICE_PATCH_FILE) || true
endif

ifneq ($(findstring $(BUILD_TYPE), small),)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(SMALL_PATCH_FILE)
endif
ifeq ($(UNIFIED_COLLECTION_PATCH_FILE), $(wildcard $(UNIFIED_COLLECTION_PATCH_FILE)))
	$(hide) $(UNIFIED_COLLECTION_PATCH_FILE) $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(DEVICE_NAME) $(KERNEL_VERSION)
endif
	$(hide) cp -rf $(KERNEL_CONFIG_PATH)/. $(KERNEL_SRC_TMP_PATH)/
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) distclean
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) $(DEFCONFIG_FILE)
ifeq ($(KERNEL_VERSION), linux-5.10)
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) modules_prepare
endif
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) -j64 $(KERNEL_IMAGE)

ifeq ($(DEVICE_NAME), hispark_phoenix)
	$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) dtbs
endif

.PHONY: build-kernel
build-kernel: $(KERNEL_IMAGE_FILE)
```

#### 构建阶段

| 阶段 | 行号 | 操作 | 目的 |
|------|------|------|--------|
| 1 | L89-94 | 拷贝内核源码 | 准备构建环境 |
| 2 | L95 | 应用 HDF 补丁 | 集成 HDF 驱动框架 |
| 3 | L97-101 | 应用设备补丁 | 集成芯片平台驱动 |
| 4 | L103-105 | 应用 small 补丁 | 小型系统优化 |
| 5 | L107-108 | 应用统一集合补丁 | 公共模块集成 |
| 6 | L109-110 | 拷贝配置文件 | 应用内核配置 |
| 7 | L111 | 清理 | 清理旧的构建产物 |
| 8 | L112 | 应用 defconfig | 加载默认配置 |
| 9 | L113-114 | 准备模块 | (仅 linux-5.10) |
| 10 | L115 | 编译内核 | 生成内核镜像 |
| 11 | L117-119 | 编译设备树 | (仅 hispark_phoenix) |

---

## 脚本调用链

### 标准系统构建流程

```
OpenHarmony GN Build
        │
        ▼
┌─────────────────────────────────┐
│  BUILD.gn                     │
│  action("build_kernel")        │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  build_kernel.sh              │
│  - 设置工作目录                │
│  - 清理 kernel.timestamp       │
│  - 调用 kernel_module_build.sh │
│  - 复制内核镜像              │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  kernel_module_build.sh        │
│  - 导出环境变量               │
│  - 判断架构和镜像类型         │
│  - 调用 make -f kernel.mk  │
│  - 验证输出                  │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  kernel.mk                    │
│  1. 拷贝源码                │
│  2. 应用补丁（HDF+设备）     │
│  3. 应用配置                │
│  4. 清理 + defconfig         │
│  5. 并行编译 (-j64)          │
│  6. 生成镜像                │
└─────────────────────────────────┘
        │
        ▼
   内核镜像 (uImage/Image/...)
```

### 小型系统构建流程

```
OpenHarmony GN Build
        │
        ▼
┌─────────────────────────────────┐
│  BUILD.gn                     │
│  build_ext_component(          │
│    "linux_kernel")            │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  kernel_module_build.sh        │
│  - 直接执行                    │
│  - 不经过 build_kernel.sh     │
└─────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────┐
│  kernel.mk                    │
│  - 相同的构建流程             │
│  - 输出到不同目录            │
└─────────────────────────────────┘
```

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [GN Targets](04_GN_Targets.md) - GN 构建目标清单
- [内核配置](06_Kernel_Configuration.md) - defconfig 管理机制
- [补丁管理](07_Patch_Management.md) - 补丁应用机制详解
