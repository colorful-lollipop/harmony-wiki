# 目录结构

> **更新时间**: 2026-02-06

---

## 文档目的

本文档描述 OpenHarmony Linux Kernel 构建项目的目录组织结构和各目录的职责，帮助开发者快速定位代码和配置。

---

## 顶层目录树

```
/Volumes/lexar/code/d/work/oh/kernel/linux/build/
├── build_kernel.sh              # [构建脚本] 主内核镜像构建脚本
├── kernel_build.py              # [构建脚本] Python CI 构建工具（含警告解析器）
├── kernel_module_build.sh       # [构建脚本] 模块编译包装脚本
├── check_build.sh              # [构建脚本] 基于时间戳的增量构建检查器
├── kernel.mk                  # [Makefile] 核心内核构建规则和补丁应用逻辑
├── BUILD.gn                   # [GN 配置] OpenHarmony GN 构建配置
├── bundle.json                 # [配置] OpenHarmony 组件清单
├── LICENSE                    # [文档] GPL 2.0 许可证
├── README.md                  # [文档] 英文文档
├── README_zh.md               # [文档] 中文文档
├── OAT.xml                    # [配置] OpenHarmony 合规性元数据
├── wiki/                      # [文档] 工程 Wiki 目录
│   ├── README.md
│   ├── SUMMARY.md
│   ├── _work/
│   │   ├── PLAN.md
│   │   └── NOTES.md
│   └── appendix/
│       ├── Callgraphs.md (待创建)
│       └── Config_Flags.md (待创建)
└── test/                      # [测试] 测试套件（文档不引用）
    ├── BUILD.gn
    ├── fuzztest/
    │   ├── accesstokenid/
    │   ├── hc_node/
    │   ├── memory/
    │   └── sched/
    ├── kernel_ltp/
    │   ├── syscalls/ (150+ 个系统调用测试)
    │   └── kernel_interface_template.gni
    ├── moduletest/
    │   └── runtest/
    ├── syzkaller/
    ├── tracepointtest/
    └── unittest/
```

---

## 目录职责分类

### 1. 构建脚本 (`scripts/`)

| 文件 | 行数 | 职责 | 关键功能 |
|------|------|--------|---------|
| `build_kernel.sh` | 52 | 内核镜像交付 | 复制不同架构的内核镜像到输出目录 |
| `kernel_module_build.sh` | 72 | 构建入口 | 设置环境变量，调用 kernel.mk |
| `check_build.sh` | 43 | 增量构建检查 | 通过时间戳比较判断是否需要重建 |
| `kernel_build.py` | 388 | CI 构建 | Python 构建，支持多架构和警告解析 |

**代码证据**:
```bash
# build_kernel.sh:22-26
#$1 - kernel build script work dir
#$2 - kernel build script stage dir
#$3 - GN target output dir

echo build_kernel
pushd ${1}
./kernel_module_build.sh ${2} ${4} ${5} ${6} ${7} ${8}
mkdir -p ${3}
rm -rf ${3}/../../../kernel.timestamp
```

---

### 2. 构建配置 (`configs/`)

| 文件 | 行数 | 职责 | 关键内容 |
|------|------|--------|---------|
| `kernel.mk` | 122 | 核心 Makefile | 工具链路径、补丁应用逻辑、多架构构建规则 |
| `BUILD.gn` | 83 | GN 构建规则 | 定义 linux_kernel target，集成到 OpenHarmony 构建 |
| `bundle.json` | 43 | 组件清单 | 组件元数据、依赖、构建入口点 |
| `OAT.xml` | ~120 | 合规性元数据 | OpenHarmony 合规性检查配置 |

**代码证据**:
```json
// bundle.json:2-42
{
    "name": "@openharmony/linux",
    "version": "3.1.0",
    "description": "Linux内核",
    "component": {
        "name": "linux",
        "subsystem": "kernel",
        "syscap": ["SystemCapability.Resourceschedule.QoS.Core"],
        "build": {
            "sub_component": ["//kernel/linux/build:linux_kernel"],
            "test": ["//kernel/linux/build/test:linuxkerneltest"]
        }
    }
}
```

---

### 3. 文档 (`docs/`)

| 文件 | 职责 | 目标受众 |
|------|--------|---------|
| `README.md` | 英文项目说明 | 国际开发者 |
| `README_zh.md` | 中文项目说明 | 中国开发者 |
| `LICENSE` | GPL 2.0 许可证 | 所有使用者 |
| `wiki/` | 工程 Wiki | 项目成员 |

---

### 4. 测试基础设施 (`test/` - 文档不引用)

**注意**: 测试目录内容不在本文档分析范围内。

**测试类型**:
- `fuzztest/` - 模糊测试（accesstokenid、hc_node、memory、sched）
- `kernel_ltp/` - Linux Test Project 系统调用测试（150+ 个）
- `moduletest/` - 内核模块功能测试
- `syzkaller/` - Syzkaller 模糊测试
- `tracepointtest/` - 追踪点测试
- `unittest/` - C++ 单元测试

---

## 外部依赖目录

本项目是构建包装器，依赖以下外部目录（不在本仓库中）：

| 路径 | 内容 | 用途 | 引用位置 |
|--------|------|--------|----------|
| `kernel/linux/linux-4.19` | Linux 4.19 内核源码 | 内核源码 | `kernel.mk:27` |
| `kernel/linux/linux-5.10` | Linux 5.10 内核源码 | 内核源码 | `kernel.mk:27` |
| `kernel/linux/patches/` | 驱动补丁 | 设备适配 | `kernel.mk:28` |
| `kernel/linux/config/` | 内核配置文件 | defconfig | `kernel.mk:29` |
| `drivers/hdf_core/adapter/khdf/linux/` | HDF 驱动框架 | HDF 补丁脚本 | `kernel.mk:95` |
| `prebuilts/gcc/` | GCC 交叉编译器 | 目标代码编译 | `kernel.mk:30` |
| `prebuilts/clang/` | Clang 工具链 | 主机编译/RISC-V | `kernel.mk:31-32` |
| `device/board/` | 板级配置 | Boot 镜像路径 | `kernel.mk:22` |
| `vendor/*/patches/` | 产品补丁 | 产品定制 | `kernel.mk:78` |

**代码证据**:
```makefile
# kernel.mk:27-32
KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc
CLANG_HOST_TOOLCHAIN := $(OHOS_BUILD_HOME)/prebuilts/clang/ohos/linux-x86_64/llvm/bin
```

---

## 关键文件路径规则

### 补丁文件路径

```
kernel/linux/patches/
├── linux-4.19/
│   ├── common_patch/
│   │   └── hdf.patch
│   ├── hispark_taurus_patch/
│   │   └── hispark_taurus.patch
│   └── rk3568_patch/
│       ├── kernel.patch
│       └── hdf.patch
└── linux-5.10/
    ├── common_patch/
    │   └── hdf.patch
    ├── hispark_taurus_patch/
    │   └── hispark_taurus.patch
    └── rk3568_patch/
        ├── kernel.patch
        └── hdf.patch
```

**代码证据**:
```makefile
# kernel.mk:76-77
DEVICE_PATCH_DIR := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}/$(DEVICE_NAME)_patch
DEVICE_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME).patch
```

### 配置文件路径

```
kernel/linux/config/
├── linux-4.19/
│   └── arch/
│       ├── arm/configs/
│       └── arm64/configs/
└── linux-5.10/
    └── arch/
        ├── arm/configs/
        │   ├── hispark_taurus_standard_defconfig
        │   └── hispark_taurus_small_defconfig
        ├── arm64/configs/
        │   └── rk3568_standard_defconfig
        └── riscv/configs/
```

**代码证据**:
```makefile
# kernel.mk:81
DEFCONFIG_FILE := $(DEVICE_NAME)_$(BUILD_TYPE)_defconfig
```

### 输出文件路径

```
out/
├── kernel/
│   ├── src_tmp/${KERNEL_VERSION}/    # Small/standard 构建源码临时目录
│   └── OBJ/${KERNEL_VERSION}/         # 编译对象目录
└── packages/phone/images/
    ├── uImage (arm)
    ├── Image (arm64/riscv64)
    ├── bzImage (x86_64)
    ├── vmlinuz.efi (loongarch64)
    ├── zImage-dtb (arm - hispark_taurus)
    └── dtbo.img (arm - hispark_phoenix)
```

**代码证据**:
```bash
# build_kernel.sh:29-30, 38-39
elif [ "$BUILD_TYPE" == "standard" ];then
    LINUX_KERNEL_OUT=${OUT_DIR}/kernel/src_tmp/${KERNEL_VERSION}
fi
LINUX_KERNEL_OBJ_OUT=${OUT_DIR}/kernel/OBJ/${KERNEL_VERSION}
```

---

## 文件组织原则

### 1. 职责分离

- **构建逻辑** 集中在 `kernel.mk` 和 `BUILD.gn`
- **脚本入口** 集中在 `kernel_module_build.sh`
- **镜像交付** 集中在 `build_kernel.sh`
- **测试基础设施** 独立在 `test/` 目录

### 2. 架构无关

- `kernel.mk` 通过变量实现多架构支持
- `BUILD.gn` 通过 `target_cpu` 自动选择镜像类型
- `build_kernel.sh` 通过条件判断处理不同架构输出

### 3. 外部引用

- 不包含内核源码、工具链和驱动源码
- 通过路径变量引用外部资源
- 支持灵活的目录结构配置

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [GN Targets](04_GN_Targets.md) - GN 构建目标清单
- [构建脚本](05_Build_Scripts.md) - 脚本详细分析
