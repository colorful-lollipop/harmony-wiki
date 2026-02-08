# UniProton 构建系统

## 概述

UniProton 使用 **GN + Ninja** 作为主要构建系统，同时保留 CMake 支持。

| 构建系统 | 用途 | 优先级 |
|----------|------|--------|
| **GN + Ninja** | 主要构建方式 | 推荐 |
| **CMake** | 辅助/兼容 | 可选 |

**证据**: `BUILD.gn` 第 13-14 行 - `import("//build/lite/config/component/lite_component.gni")`, `import("//build/ohos.gni")`

---

## GN 构建文件

### 根构建文件 (BUILD.gn)

路径: `/Volumes/lexar/code/d/work/oh/kernel/uniproton/BUILD.gn`

#### 主要 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `kernel` | group | - | 内核组，包含 libkernel 和 notice |
| `libkernel` | static_library | `libkernel.a` | ★ 核心静态库 |
| `uniproton` | executable | `uniproton` | 可执行固件 |
| `copy_image` | copy | `OHOS_Image` | 镜像拷贝 |
| `build_kernel_image` | build_ext_component | `.bin/.asm/.sym` | 镜像生成 |

#### 关键 Config

| Config | 用途 | 主要 Flags |
|--------|------|-----------|
| `arch_config` | 架构配置 | 架构特定编译选项 |
| `kernel_config` | 内核配置 | 头文件路径 |
| `warn_config` | 警告配置 | `-Wall`, `-Werror` |
| `strong_waring_flag` | 强警告 | 更多严格检查 |
| `ssp_config` | 栈保护 | `-fstack-protector-*` |
| `misc_config` | 杂项配置 | `-fno-exceptions`, `-fno-pic` |
| `os_config` | 系统配置 | 汇总上述配置 |

**证据**: `BUILD.gn` 第 24-105 行定义各类 config

---

### 配置文件 (uniproton.gni)

路径: `/Volumes/lexar/code/d/work/oh/kernel/uniproton/uniproton.gni`

#### 模板定义

| 模板 | 用途 |
|------|------|
| `kernel_module` | 内核模块构建模板 |
| `config` | 配置构建模板 |
| `module_group` | 模块组模板 |

#### 核心变量

```gn
# 路径定义
OSTOPDIR = "//kernel/uniproton/src"
OSTHIRDPARTY = "//third_party"
HDFTOPDIR = "//drivers/hdf_core/adapter/khdf/uniproton"

# 基础头文件路径
KERNEL_BASE_INCLUDE_DIRS = [
  "$OSTOPDIR/arch/include",
  "$OSTOPDIR/core/kernel/include",
  "$OSTOPDIR/mem/include",
  "$OSTOPDIR/om/include",
  "$OSTOPDIR/utility/lib/include",
]
```

**证据**: `uniproton.gni` 第 34-38 行及第 131-139 行

---

## 源文件分组

### 基础源文件 (KERNEL_BASE_SOURCES)

```gn
KERNEL_BASE_SOURCES = [
  "$OSTOPDIR/config/prt_config.c",           # 配置
  "$OSTOPDIR/core/kernel/irq/prt_irq.c",     # 中断
  "$OSTOPDIR/core/kernel/kexc/prt_kexc.c",   # 异常
  "$OSTOPDIR/core/kernel/sys/prt_sys.c",     # 系统
  "$OSTOPDIR/core/kernel/sys/prt_sys_init.c",# 初始化
  "$OSTOPDIR/core/kernel/sys/prt_sys_time.c",# 时间
  # ... 任务相关 ...
  "$OSTOPDIR/core/kernel/task/prt_task.c",
  # ... Tick/定时器 ...
  "$OSTOPDIR/core/kernel/tick/prt_tick.c",
  "$OSTOPDIR/core/kernel/timer/prt_timer.c",
]
```

**证据**: `uniproton.gni` 第 141-166 行

---

### 可选 IPC 源文件

| Feature | 变量 | 路径 |
|---------|------|------|
| 事件 | `KERNEL_IPC_EVENT_SOURCES` | `$OSTOPDIR/core/ipc/event/prt_event.c` |
| 队列 | `KERNEL_IPC_QUEUE_SOURCES` | `$OSTOPDIR/core/ipc/queue/prt_*.c` |
| 软件定时器 | `KERNEL_SWTMR_SOURCES` | `$OSTOPDIR/core/kernel/timer/swtmr/prt_swtmr.c` |
| 信号量 | `KERNEL_IPC_SEM_SOURCES` | `$OSTOPDIR/core/ipc/sem/prt_sem.c` |

**证据**: `BUILD.gn` 第 189-198 行

```gn
if (defined(OS_OPTION_EVENT)) {
  sources += KERNEL_IPC_EVENT_SOURCES
}
if (defined(OS_OPTION_QUEUE)) {
  sources += KERNEL_IPC_QUEUE_SOURCES
}
```

---

### 可选组件源文件

| 组件 | 变量 | 路径 |
|------|------|------|
| 内存 | `KERNEL_MEM_SOURCES` | `src/mem/` |
| 运维 | `KERNEL_OM_SOURCES` | `src/om/` |
| CPU 占用率 | `KERNEL_OM_CPUP_SOURCES` | `src/om/cpup/` |
| 安全 | `KERNEL_SECURITY_SOURCES` | `src/security/rnd/prt_rnd_set.c` |
| 工具库 | `KERNEL_UTILITY_SOURCES` | `src/utility/lib/` |

**证据**: `BUILD.gn` 第 189-205 行

---

### 文件系统 (可选)

```gn
if (defined(OS_SUPPORT_FS)) {
  sources += KERNEL_FS_SOURCES + LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE
  include_dirs += KERNEL_FS_INCLUDE_DIRS + LITTLEFS_INCLUDE_DIRS
}
```

**证据**: `BUILD.gn` 第 207-210 行

---

### 网络栈 (可选)

```gn
if (defined(OS_SUPPORT_NET)) {
  sources += KERNEL_LWIP_SOURCES + LWIPNOAPPSFILES
  include_dirs += KERNEL_LWIP_INCLUDE_DIRS + LWIP_INCLUDE_DIRS
}
```

**证据**: `BUILD.gn` 第 212-215 行

---

## 架构特定源文件

### ARMv7-M

```gn
if (defined(OS_ARCH_ARMV7_M)) {
  sources += ARCH_ARMVM7_M_SOURCES
  if ("$board_cpu" == "cortex-m4") {
    sources += ARCH_CORTEX_M4_SOURCES
    include_dirs += ARCH_CORTEX_M4_INCLUDE_DIRS
  }
}
```

**ARMv7-M 源文件**:
- `src/arch/cpu/armv7-m/common/boot/prt_hw_boot.c`
- `src/arch/cpu/armv7-m/common/exc/prt_exc.c`
- `src/arch/cpu/armv7-m/common/hwi/prt_hwi.c`
- `src/arch/cpu/armv7-m/common/tick/prt_hw_tick.c`
- `src/arch/cpu/armv7-m/common/prt_port.c`

**Cortex-M4 特定**:
- `src/arch/cpu/armv7-m/cortex-m4/prt_dispatch.S`
- `src/arch/cpu/armv7-m/cortex-m4/prt_hw.S`
- `src/arch/cpu/armv7-m/cortex-m4/prt_vector.S`

**证据**: `uniproton.gni` 第 244-264 行

---

## 依赖关系

### 内部依赖

```
uniproton (executable)
├── libkernel (static_library)
│   ├── KERNEL_BASE_SOURCES (基础内核)
│   ├── KERNEL_IPC_*_SOURCES (IPC 机制)
│   ├── KERNEL_MEM_SOURCES (内存)
│   ├── KERNEL_OM_SOURCES (运维)
│   ├── ARCH_ARMVM7_M_SOURCES (ARMv7-M 架构)
│   ├── ARCH_CORTEX_M4_SOURCES (Cortex-M4 特定)
│   ├── KERNEL_FS_SOURCES + LITTLEFS (文件系统, 可选)
│   └── KERNEL_LWIP_SOURCES (网络, 可选)
└── 外部依赖:
    ├── //third_party/bounds_checking_function:libsec_static
    ├── //third_party/musl/porting/uniproton/kernel:kernel
    └── HDFTOPDIR (HDF 驱动, 可选)
```

**证据**: `BUILD.gn` 第 227-231 行

```gn
deps = [ "//third_party/bounds_checking_function:libsec_static" ]
deps += [ "//third_party/musl/porting/uniproton/kernel:kernel" ]
if (defined(DRIVERS_HDF)) {
  deps += [ HDFTOPDIR ]
}
```

---

## 编译产物

### 静态库

| 产物 | 路径 | 说明 |
|------|------|------|
| `libkernel.a` | `out/uniproton/lib/` | 内核静态库 |

### 可执行文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `uniproton` | `out/uniproton/unstripped/bin/` | 未 strip 的固件 |
| `OHOS_Image` | `out/uniproton/` | 复制后的镜像 |
| `OHOS_Image.bin` | `out/uniproton/` | 二进制镜像 |
| `OHOS_Image.sym` | `out/uniproton/` | 符号表 (排序后) |
| `OHOS_Image.asm` | `out/uniproton/` | 反汇编 |

**证据**: `BUILD.gn` 第 241-278 行

```gn
executable("uniproton") {
  ldflags = [
    "-static",
    "-Wl,--gc-sections",
    "-Wl,-Map=$uniproton_name.map",
  ]
  output_dir = target_out_dir
}

copy("copy_image") {
  sources = [ "$target_out_dir/unstripped/bin/uniproton" ]
  outputs = [ "$root_out_dir/$uniproton_name" ]
}

build_ext_component("build_kernel_image") {
  command = "$objcopy -O binary $uniproton_name $uniproton_name.bin"
  command += " && sh -c '$objdump -t $uniproton_name | sort >$uniproton_name.sym.sorted'"
  command += " && sh -c '$objdump -d $uniproton_name >$uniproton_name.asm'"
}
```

---

## 编译配置

### 配置开关

| 开关 | 类型 | 说明 |
|------|------|------|
| `OS_OPTION_EVENT` | bool | 启用事件机制 |
| `OS_OPTION_QUEUE` | bool | 启用消息队列 |
| `OS_OPTION_CPUP` | bool | 启用 CPU 占用率统计 |
| `OS_SUPPORT_FS` | bool | 启用文件系统 |
| `OS_SUPPORT_NET` | bool | 启用网络栈 |
| `OS_ARCH_ARMV7_M` | bool | ARMv7-M 架构 |
| `OS_ARCH_ARMV8` | bool | ARMv8 架构 |
| `CC_STACKPROTECTOR_*` | string | 栈保护级别 |

### 配置来源

配置文件: `${product_path}/kernel_configs/*.config` (通过 Kconfig 生成 `config.gni`)

**证据**: `uniproton.gni` 第 15-32 行

```gn
product_config_file = "${ohos_build_type}.config"
exec_script(
    "//build/lite/run_shell_cmd.py",
    [ "env CONFIG_= KCONFIG_CONFIG_HEADER='y=true' ..." ],
    "",
    [ product_config_file ])
import("$root_out_dir/config.gni")
```

---

## 编译命令

### 标准编译

```bash
# 使用 hb (OpenHarmony 构建工具)
hb set -kernel uniproton
hb build

# 或直接使用 GN
gn gen out/uniproton --args="target_board='stm32f407zg'"
ninja -C out/uniproton
```

### 输出产物

```
out/uniproton/
├── OHOS_Image           # ELF 镜像
├── OHOS_Image.bin       # 二进制镜像
├── OHOS_Image.asm       # 反汇编
├── OHOS_Image.map       # 链接 map
├── OHOS_Image.sym       # 符号表
└── unstripped/bin/
    └── uniproton        # 未 strip 的可执行文件
```

---

## 组件清单 (bundle.json)

```json
{
  "name": "kernel_uniproton",
  "description": "UniProton 实时操作系统内核",
  "version": "4.0",
  "license": "Mulan-PSL-2.0",
  "publish-type": "component",
  "segment": {
    "dest": {
      "code": [
        "kernel/uniproton/src"
      ],
      "header": []
    }
  }
}
```

---

## 相关文档

- [概览](./01_Overview.md) - 项目定位
- [目录结构](./02_Directory_Structure.md) - 代码组织
- [API 参考](./03_API_Reference.md) - 接口说明
- [架构设计](./04_Architecture.md) - 组件交互
