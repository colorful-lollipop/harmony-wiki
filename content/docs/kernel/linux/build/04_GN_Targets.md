# GN Targets

> **更新时间**: 2026-02-06

---

## 文档目的

本文档详细列出 OpenHarmony Linux Kernel 构建系统中的所有 GN targets，包括类型、依赖关系、输出产品和关键配置。

---

## 主构建 Targets

### 1. linux_kernel (GROUP)

**类型**: `group` (标准系统) / `build_ext_component` (小型系统)

**目的**: 内核构建的入口点，供 OpenHarmony 构建系统调用

**代码证据**:
```gn
// BUILD.gn:36-44
if (os_level == "mini" || os_level == "small") {
  build_ext_component("linux_kernel") {
    no_default_deps = true
    exec_path = rebase_path(".", root_build_dir)
    outdir = rebase_path("$root_out_dir")
    build_type = "small"
    product_path_rebase = rebase_path(product_path, ohos_root_path)
    command = "./kernel_module_build.sh ${outdir} ${build_type} ${target_cpu} ${product_path_rebase} ${board_name} ${linux_kernel_version}"
  }
} else {
  group("linux_kernel") {
    deps = [ ":build_kernel" ]
  }
}
```

**依赖**:
- 标准系统: `:build_kernel`
- 小型系统: 直接执行 `kernel_module_build.sh`

**输出**: 无直接输出，依赖链传递到其他 targets

---

### 2. build_kernel (ACTION)

**类型**: `action`

**目的**: 执行内核构建脚本并生成最终的内核镜像

**代码证据**:
```gn
// BUILD.gn:59-77
action("build_kernel") {
  script = "build_kernel.sh"
  sources = [ kernel_source_dir ]

  deps = [ ":check_build" ]
  product_path = "vendor/$product_company/$product_name"
  build_type = "standard"
  outputs = [ "$root_build_dir/packages/phone/images/$kernel_image" ]
  args = [
    rebase_path(kernel_build_script_dir, root_build_dir),
    rebase_path("$root_out_dir/../KERNEL_OBJ"),
    rebase_path("$root_build_dir/packages/phone/images"),
    build_type,
    target_cpu,
    product_path,
    device_name,
    linux_kernel_version,
  ]
}
```

**参数说明**:

| 参数 | 值来源 | 用途 |
|------|--------|--------|
| `kernel_build_script_dir` | `"//kernel/linux/build"` | 构建脚本位置 |
| `kernel_source_dir` | `"//kernel/linux/$linux_kernel_version"` | 内核源码位置 |
| `build_type` | `"standard"` | 构建变体 |
| `target_cpu` | `arm/arm64/riscv64/x86_64/loongarch64` | 目标架构 |
| `product_path` | `"vendor/$product_company/$product_name"` | 产品配置路径 |
| `device_name` | 设备名称 (如 `hispark_taurus`) | 设备标识 |
| `linux_kernel_version` | `"linux-5.10"` 或 `"linux-4.19"` | 内核版本 |

**依赖**: `:check_build`

**输出**:
```
$root_build_dir/packages/phone/images/$kernel_image
```

**镜像类型** (由 `target_cpu` 决定):

| target_cpu | kernel_image | 输出文件 |
|-----------|--------------|-----------|
| arm | "uImage" | `packages/phone/images/uImage` |
| arm64 | "Image" | `packages/phone/images/Image` |
| riscv64 | "Image" | `packages/phone/images/Image` |
| x86_64 | "bzImage" | `packages/phone/images/bzImage` |
| loongarch64 | "vmlinuz.efi" | `packages/phone/images/vmlinuz.efi` |

**代码证据**:
```gn
// BUILD.gn:22-34
kernel_image = ""
if (target_cpu == "arm") {
  kernel_image = "uImage"
} else if (target_cpu == "arm64") {
  kernel_image = "Image"
} else if (target_cpu == "riscv64") {
  kernel_image = "Image"
} else if (target_cpu == "x86_64") {
  kernel_image = "bzImage"
} else if (target_cpu == "loongarch64") {
  kernel_image = "vmlinuz.efi"
}
```

---

### 3. check_build (ACTION)

**类型**: `action`

**目的**: 通过时间戳检查确定是否需要重建内核，实现增量构建

**代码证据**:
```gn
// BUILD.gn:48-57
action("check_build") {
  script = "check_build.sh"
  sources = [ kernel_source_dir ]
  outputs = [ "$root_build_dir/kernel.timestamp" ]
  args = [
    rebase_path(kernel_source_dir, root_build_dir),
    rebase_path("$root_build_dir/packages/phone/images/$kernel_image"),
    rebase_path("$root_build_dir/kernel.timestamp"),
  ]
}
```

**参数说明**:

| 参数 | 用途 |
|------|--------|
| `kernel_source_dir` (路径) | 内核源码目录，用于检查文件更新时间 |
| `packages/phone/images/$kernel_image` (路径) | 输出镜像路径，用于比较时间戳 |
| `kernel.timestamp` (路径) | 时间戳文件，触发重建的标志 |

**依赖**: 无

**输出**:
```
$root_build_dir/kernel.timestamp
```

**工作原理**:
1. 遍历 `kernel_source_dir` 中的所有文件
2. 检查是否有文件比输出镜像新 (`$file -nt $output`)
3. 如果有新文件，删除旧镜像，触发重建
4. 生成/更新 `kernel.timestamp` 文件

**代码证据**:
```bash
# check_build.sh:18-41
function readfile () {
    for file in $1/*
    do
        if [ -d "$file" ];then
	    readfile $file $2 $3
        elif [ "$file" -nt "$2" ]; then
            echo $file is update
            touch $3;
            return
        fi
    done
}

if [ -e "$2" ]; then
    readfile $1 $2 $3
    if [ "$3" -nt "$2" ]; then
        echo "need update $2"
        rm -rf $2;
    fi
fi
```

---

## 依赖关系图

### 主构建依赖链

```
linux_kernel (group)
    │
    └──> build_kernel (action)
              │
              ├──> check_build (action)
              │
              └──> kernel_module_build.sh (shell)
                        │
                        └──> kernel.mk (makefile)
                                  │
                                  ├──> 内核源码拷贝
                                  ├──> HDF 补丁应用
                                  ├──> 设备补丁应用
                                  ├──> 配置文件应用
                                  └──> 内核编译 (make)
```

**代码证据**:
- `BUILD.gn:79-81` - `linux_kernel` 依赖 `:build_kernel`
- `BUILD.gn:63` - `build_kernel` 依赖 `:check_build`
- `build_kernel.sh:24` - 调用 `kernel_module_build.sh`
- `kernel_module_build.sh:58` - 调用 `make -f kernel.mk`

---

## 配置变量

### GN 变量

| 变量名 | 定义位置 | 用途 | 默认值/来源 |
|---------|---------|--------|--------------|
| `linux_kernel_version` | GN args | 内核版本选择 | `"linux-5.10"` 或 `"linux-4.19"` |
| `target_cpu` | GN args | 目标架构 | `"arm"`/`"arm64"`/`"riscv64"`/`"x86_64"`/`"loongarch64"` |
| `os_level` | GN args | 系统类型 | `"mini"`/`"small"`/`"standard"`/`"foundation"` |
| `product_company` | GN args | 厂商名称 | 来自产品配置 |
| `product_name` | GN args | 产品名称 | 来自产品配置 |
| `device_name` | GN args | 设备名称 | 来自产品配置 |
| `product_path` | BUILD.gn:64 | 产品配置路径 | `"vendor/$product_company/$product_name"` |
| `build_type` | BUILD.gn:65 | 构建变体 | `"standard"` |
| `root_build_dir` | GN 全局变量 | 构建输出根目录 | `"out/xxx"` |
| `root_out_dir` | GN 全局变量 | 构建输出目录 | `"out/xxx/..."` |
| `kernel_image` | BUILD.gn:22-34 | 内核镜像名称 | 由 `target_cpu` 决定 |
| `kernel_source_dir` | BUILD.gn:47 | 内核源码目录 | `"//kernel/linux/$linux_kernel_version"` |
| `kernel_build_script_dir` | BUILD.gn:46 | 构建脚本目录 | `"//kernel/linux/build"` |

**代码证据**:
```gn
// BUILD.gn:14-19
import("//build/ohos/kernel/kernel.gni")

// BUILD.gn:22-34
kernel_image = ""
if (target_cpu == "arm") {
  kernel_image = "uImage"
} else if (target_cpu == "arm64") {
  kernel_image = "Image"
} ...
```

### kernel.mk 变量

| 变量名 | 定义位置 | 用途 | 示例值 |
|---------|---------|--------|--------|
| `KERNEL_VERSION` | Make 参数 | 内核版本 | `linux-5.10` |
| `BUILD_TYPE` | Make 参数 | 构建变体 | `standard`/`small` |
| `KERNEL_ARCH` | Make 参数 | 目标架构 | `arm`/`arm64`/`riscv64`/`x86_64`/`loongarch64` |
| `DEVICE_NAME` | Make 参数 | 设备名称 | `hispark_taurus`/`rk3568` |
| `PRODUCT_NAME` | kernel.mk:17 | 产品名称 | `Hi3516DV300` |
| `OHOS_BUILD_HOME` | kernel.mk:18 | OpenHarmony 根目录 | `/path/to/openharmony` |
| `KERNEL_SRC_TMP_PATH` | kernel.mk:19-24 | 内核源码临时目录 | `out/kernel/src_tmp/linux-5.10` |
| `KERNEL_OBJ_TMP_PATH` | kernel.mk:20 | 编译对象目录 | `out/kernel/OBJ/linux-5.10` |
| `KERNEL_SRC_PATH` | kernel.mk:27 | 内核源码路径 | `kernel/linux/linux-5.10` |
| `KERNEL_PATCH_PATH` | kernel.mk:28 | 补丁目录路径 | `kernel/linux/patches/linux-5.10` |
| `KERNEL_CONFIG_PATH` | kernel.mk:29 | 配置文件路径 | `kernel/linux/config/linux-5.10` |
| `KERNEL_IMAGE_FILE` | kernel.mk:80 | 内核镜像文件路径 | `${KERNEL_OBJ_TMP_PATH}/arch/arm/boot/uImage` |
| `DEFCONFIG_FILE` | kernel.mk:81 | defconfig 文件名 | `hispark_taurus_standard_defconfig` |
| `KERNEL_TARGET_TOOLCHAIN` | kernel.mk:38-54 | 目标工具链路径 | GCC 工具链路径 |
| `KERNEL_CROSS_COMPILE` | kernel.mk:36-68 | 交叉编译参数 | `CROSS_COMPILE=...` |
| `KERNEL_MAKE` | kernel.mk:70-72 | make 命令 | 带环境变量的 make |

**代码证据**:
```makefile
# kernel.mk:17-32
PRODUCT_NAME=$(TARGET_PRODUCT)
OHOS_BUILD_HOME := $(realpath $(shell pwd)/../../../)
KERNEL_SRC_TMP_PATH := $(OUT_DIR)/kernel/${KERNEL_VERSION}
KERNEL_OBJ_TMP_PATH := $(OUT_DIR)/kernel/OBJ/${KERNEL_VERSION}
KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
```

---

## Target 与产物映射

### 最终内核镜像产物

| Target | 输出路径 | 内核镜像 | 架构 |
|--------|----------|----------|--------|
| `:build_kernel` | `out/.../packages/phone/images/uImage` | uImage | arm |
| `:build_kernel` | `out/.../packages/phone/images/Image` | Image | arm64 |
| `:build_kernel` | `out/.../packages/phone/images/Image` | Image | riscv64 |
| `:build_kernel` | `out/.../packages/phone/images/bzImage` | bzImage | x86_64 |
| `:build_kernel` | `out/.../packages/phone/images/vmlinuz.efi` | vmlinuz.efi | loongarch64 |

**代码证据**:
```bash
# build_kernel.sh:29-49
if [ "$5" == "arm" ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/uImage ${3}/uImage
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

### 特殊输出 (hispark_phoenix)

| Target | 输出文件 | 用途 |
|--------|----------|--------|
| `:build_kernel` | `dtbo.img` | 设备树镜像 |
| `:build_kernel` | `zImage-dtb` | 内核+设备树合并镜像 |

**代码证据**:
```bash
# build_kernel.sh:31-36
if [ "$7" == "hispark_phoenix"  ];then
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/dts/hi3751v350.dtb ${3}/dtbo.img
    cat ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage ${3}/dtbo.img > ${3}/zImage-dtb
else
    cp ${2}/kernel/OBJ/${8}/arch/arm/boot/zImage-dtb ${3}/zImage-dtb
fi
```

---

## GN 构建命令示例

### 构建 Linux 5.10 内核 (标准系统，ARM64)

```bash
./build.sh --product-name rk3568 \
          --build-target build_kernel \
          --gn-args 'target_cpu="arm64" linux_kernel_version="linux-5.10"'
```

**GN 层面**:
```bash
gn gen out/rk3568 \
   --args='target_cpu="arm64" linux_kernel_version="linux-5.10"'
```

**执行链**:
1. GN 解析 `BUILD.gn`
2. Ninja 执行 `:linux_kernel` target
3. 调用 `:check_build` action (时间戳检查)
4. 调用 `:build_kernel` action
5. `build_kernel.sh` → `kernel_module_build.sh` → `kernel.mk`
6. 生成 `out/rk3568/packages/phone/images/Image`

### 构建 Linux 4.19 内核 (小型系统，ARM)

```bash
./build.sh --product-name Hi3516DV300 \
          --build-target build_kernel \
          --gn-args 'os_level="small" target_cpu="arm" linux_kernel_version="linux-4.19"'
```

**执行链**:
1. GN 解析 `BUILD.gn`
2. 执行 `build_ext_component("linux_kernel")`
3. 直接调用 `kernel_module_build.sh`
4. 生成 `out/.../kernel/linux-4.19/arch/arm/boot/uImage`

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [构建脚本](05_Build_Scripts.md) - Shell 脚本详细分析
- [编译产物](08_Build_Artifacts.md) - 内核镜像类型与输出路径
