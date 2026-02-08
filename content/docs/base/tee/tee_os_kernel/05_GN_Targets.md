# GN Targets 与编译产物

> **文档版本**: 1.0
> **生成时间**: 2026-02-06
> **适用范围**: OpenHarmony TEE OS Kernel 构建系统

---

## 文档目的

本文档详细说明 tee_os_kernel 的 GN 构建系统、Targets 定义和编译产物。

**⚠️ 重要说明**：
- 本项目主要使用 **Makefile** 进行实际编译
- **GN 构建系统**仅作为 OpenHarmony 构建入口点存在
- GN 文件极少（仅 `build/BUILD.gn`），无复杂的 target 依赖链

---

## 目录

- [GN 构建配置](#gn-构建配置)
- [GN Targets](#gn-targets)
- [编译产物](#编译产物)
- [构建流程](#构建流程)
- [配置选项](#配置选项)

---

## GN 构建配置

### 1.1 GN 文件清单

**仅有一个 GN 文件**：
| 文件 | 路径 | 说明 |
|------|------|------|
| `BUILD.gn` | `build/BUILD.gn` | 定义 tee_os target 和构建 action |
| `bundle.json` | `/bundle.json` | OpenHarmony Bundle 配置，引用 GN target |

### 1.2 BUILD.gn 内容

**文件位置**：`build/BUILD.gn`

**GN Targets**：
```gn
# Target 1: action("build")
- Type: action（执行脚本）
- Script: oh_build_tee.sh
- Sources: //base/tee (整个 tee 源目录）
- Outputs: $root_build_dir/../tee/src_tmp/tee_os_kernel/kernel/bl32.bin

# Target 2: group("tee_os")
- Type: group（元目标）
- Deps: [":build"]
- 说明: OpenHarmony 构建入口点
```

### 1.3 Bundle 配置

**文件位置**：`bundle.json`

**关键配置**：
```json
{
  "name": "@openharmony/tee_os_kernel",
  "description": "tee_os_kernel",
  "version": "1.0.0",
  "license": "Mulan PSL v2",
  "component": {
    "name": "tee_os_kernel",
    "subsystem": "tee",
    "syscap": [],
    "adapted_system_type": ["standard"],
    "rom": "2048KB",
    "ram": "8M",
    "deps": {
      "components": [],
      "third_party": []
    },
    "build": {
      "sub_component": ["//base/tee/tee_os_kernel/build:tee_os"],
      "inner_kits": [],
      "test": []
    }
  }
}
```

---

## GN Targets

### 2.1 GN Target 清单

| Target 名称 | 类型 | 描述 | 输出 | 状态 |
|------------|------|------|------|------|
| `action("build")` | action | 执行 oh_build_tee.sh 脚本 | bl32.bin | ✅ 有效 |
| `group("tee_os")` | group | 元目标，依赖 build | N/A | ✅ 有效 |

**说明**：
- GN 构建系统是**最小集成**，仅作为 OpenHarmony 构建的入口点
- 实际编译由 `Makefile` 和 `build/build_tee.sh` 完成
- GN 的 `oh_build_tee.sh` 目前是 stub（大部分逻辑被注释）

---

## 编译产物

### 3.1 最终产物

| 产物名称 | 类型 | 位置 | 说明 |
|-----------|------|------|------|
| **bl32.bin** | Binary | `kernel/bl32.bin` | **TEE OS 镜像**，最终 Trusted Firmware |
| **procmgr** | Binary | `user/system-services/system-servers/procmgr/procmgr` | 进程管理器（嵌入内核）|
| **procmgr.bin** | Binary | `user/system-services/system-servers/procmgr/procmgr.bin` | 去除符号的 procmgr |
| **libc_shared.so** | Shared Library | `libc_shared.so` / `ramdisk-dir/libc_shared.so` | 共享 C 库 |
| **libohtee.so** | Shared Library | `user/chcore-libs/sys-libs/libohtee/libohtee.so` | TEE 用户态库 |
| **kernel.img** | ELF | `kernel/kernel.img` | 内核 ELF（stripped 成 bl32.bin）|
| **ramdisk.cpio** | CPIO Archive | `user/system-services/system-servers/tmpfs/ramdisk.cpio` | 初始 ramdisk |
| **tmpfs.srv** | Binary | `user/system-services/system-servers/tmpfs/tmpfs.srv` | 内存文件系统服务 |
| **fsm.srv** | Binary | `user/system-services/system-servers/fsm/fsm.srv` | 文件系统管理服务 |
| **chanmgr.srv** | Binary | `user/system-services/system-servers/chanmgr/chanmgr.srv` | Channel 管理服务 |

### 3.2 产物说明

#### bl32.bin（主产物）

**生成路径**：
- 输出位置：`kernel/bl32.bin`
- GN 声明输出：`$root_build_dir/../tee/src_tmp/tee_os_kernel/kernel/bl32.bin`

**包含内容**：
- 完整的 TEE 内核代码
- 嵌入的 procmgr 二进制
- 启动代码和初始化序列

**用途**：
- 作为 BL32（Trusted Bootloader Stage 3.2）运行
- 由 ATF (ARM Trusted Firmware) 加载到安全世界

#### procmgr（进程管理器）

**生成路径**：
- 源文件：`user/system-services/system-servers/procmgr/procmgr.c`
- 编译产物：`user/system-services/system-servers/procmgr/procmgr`

**特点**：
- 通过 ELF 工具嵌入到内核（`kernel/Makefile`）
- 作为 TEE 的第一个用户进程运行（badge = 1）
- 管理 TEE 内核的所有系统服务

#### 内核对象

| 对象类型 | 产物 |
|-----------|------|
| **PMO 对象** | 用户态内存映射 |
| **VMSpace 对象** | 进程地址空间 |
| **Thread 对象** | 线程 |
| **Connection 对象** | IPC 连接 |
| **Channel 对象** | TEE 消息通道 |

---

## 构建流程

### 4.1 完整构建流程

```
OpenHarmony 构建系统
    ↓
gn gen --platform=rk3568 --product-name=rk3568
    ↓
加载 bundle.json
    ↓
执行 //base/tee/tee_os_kernel/build:tee_os
    ↓
GN 执行 action("build")
    ↓
调用 oh_build_tee.sh <OH_ROOT>
    ↓ [当前为 stub，直接调用 build_tee.sh]
build_tee.sh（完整构建）
    ↓
1. 清理 framework 输出
    ↓
2. 在 tee_os_kernel 执行 make
    ↓
    make user（编译用户态）
        ↓
        - 编译 libchcore（musl libc）
        - 编译 libohtee.so
        - 编译 fs_base.a
        - 编译 tmpfs.srv
        - 编译 fsm.srv
        - 编译 chanmgr.srv
        - 编译 procmgr（嵌入上述服务）
    ↓
    make kernel（编译内核）
        ↓
        - 链接内核对象
        - 嵌入 procmgr 二进制
        - 生成 kernel.img
        - 生成 bl32.bin（stripped）
    ↓
3. 复制头文件和库到 framework
    ↓
4. 执行 framework 构建
    ↓
5. 重新 make user（集成 framework 应用）
    ↓
最终产物
    ↓
kernel/bl32.bin
libc_shared.so
libohtee.so
procmgr
```

### 4.2 Makefile 构建目标

**文件位置**：`Makefile`

**主要目标**：
```makefile
# 根目标
all: user kernel

# 用户态目标
user:
    # libc（musl libc 移植）
    $(MAKE) -C user/chcore-libc
    # libohtee（TEE 库）
    $(MAKE) -C user/chcore-libs/sys-libs/libohtee
    # fs_base（VFS 基础库）
    $(MAKE) -C user/system-services/system-servers/fs_base
    # tmpfs（内存文件系统）
    $(MAKE) -C user/system-services/system-servers/tmpfs
    # fsm（文件系统管理器）
    $(MAKE) -C user/system-services/system-servers/fsm
    # chanmgr（Channel 管理器）
    $(MAKE) -C user/system-services/system-servers/chanmgr
    # procmgr（进程管理器）
    $(MAKE) -C user/system-services/system-servers/procmgr

# 内核目标
kernel:
    # 生成 bl32.bin
    $(MAKE) -C kernel
```

---

## 配置选项

### 5.1 Makefile 配置（config.mk）

**字符串配置**：
| 变量 | 默认值 | 说明 |
|--------|---------|------|
| `CHCORE_COMPILER` | clang | 编译器工具链 |
| `CHCORE_CROSS_COMPILE` | aarch64-linux-ohos- | 交叉编译前缀 |
| `CHCORE_PLAT` | rk3568 | 目标平台（RK3568 或 RK3399）|
| `CHCORE_ARCH` | aarch64 | 目标架构 |
| `CHCORE_SPD` | opteed | Secure Payload Dispatcher |

**布尔配置**：
| 变量 | 默认值 | 说明 |
|--------|---------|------|
| `CHCORE_VERBOSE_BUILD` | OFF | 详细输出（关闭）|
| `CHCORE_ENABLE_FMAP` | ON | 功能映射（开启）|
| `CHCORE_USER_DEBUG` | OFF | 用户态调试（关闭）|
| `CHCORE_MINI` | ON | Mini 构建（减少功能）|
| `CHCORE_OH_TEE` | ON | **OpenHarmony TEE 模式** |
| `CHCORE_KERNEL_DEBUG` | OFF | 内核调试（关闭）|
| `CHCORE_KERNEL_TEST` | OFF | 内核测试（关闭）|
| `CHCORE_KERNEL_RT` | OFF | 实时内核（关闭）|

### 5.2 平台配置

| 平台 | 文本偏移 | 配置文件 |
|------|----------|----------|
| **RK3568** | 0x8400000 | `kernel/arch/aarch64/plat/rk3568/` |
| **RK3399** | 0x8408000 | `kernel/arch/aarch64/plat/rk3399/` |

**证据**：
```makefile
# Makefile 中的平台相关配置
ifeq ($(CHCORE_PLAT),rk3568)
    TEXT_OFFSET = 0x8400000
else ifeq ($(CHCORE_PLAT),rk3399)
    TEXT_OFFSET = 0x8408000
endif
```

---

## 构建命令

### 6.1 OpenHarmony 构建命令

```bash
# 完整构建命令（在 OpenHarmony 根目录）
./build.sh --product-name rk3568 --build-target tee --ccache

# 等价命令
hb build -f rk3568 --target tee
```

### 6.2 本地调试构建

```bash
# 直接使用 Makefile（跳过 GN）
cd /Volumes/lexar/code/d/work/oh/base/tee/tee_os_kernel
make clean
make all

# 仅构建用户态
make user

# 仅构建内核
make kernel
```

---

## 输出目录

### 7.1 构建输出

| 路径 | 说明 |
|------|------|
| `kernel/` | 内核编译产物（bl32.bin, kernel.img）|
| `user/chcore-libs/sys-libs/libohtee/` | TEE 库编译产物 |
| `user/system-services/system-servers/*/` | 系统服务编译产物 |
| `libc_shared.so` | 链接后的共享 C 库 |

### 7.2 OpenHarmony 输出路径

当通过 OpenHarmony 构建系统构建时：

```
${OH_ROOT}/out/tee/src_tmp/tee_os_kernel/
├── kernel/
│   └── bl32.bin  ← 最终 TEE 镜像
└── user/
    ├── libc_shared.so
    └── system-services/
        ├── procmgr/
        └── procmgr
```

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 详细的代码组织
- [架构设计](02_Architecture.md) - 内核组件与依赖关系
- [系统调用接口](03_Syscall_Interfaces.md) - 系统调用清单
- [编译产物详解](#编译产物详解) - 产物详细说明

---

**文档维护**: OpenHarmony TEE OS Kernel 开发团队
**最后更新**: 2026-02-06
