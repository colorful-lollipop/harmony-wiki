# 构建配置

## 构建系统

本项目支持两种构建系统：

| 构建系统 | 优先级 | 用途 |
|---------|-------|------|
| **GN** | 主要 | OpenHarmony 官方构建 |
| **Makefile** | 兼容 | 传统编译方式 |

## GN 构建配置

### 根目录 BUILD.gn

**文件路径**: `/Volumes/lexar/code/d/work/oh/drivers/liteos/BUILD.gn`

```gn
import("//kernel/liteos_a/liteos.gni")

group("liteos") {
  deps = [ "hievent" ]
}

config("public") {
  configs = [ "hievent:public" ]
}
```

**配置说明**:
| target | 类型 | 输出 | 说明 |
|-------|------|------|------|
| `liteos` | group | - | 根目录聚合 target |
| `public` | config | - | 导出 `hievent:public` 配置 |

### hievent/BUILD.gn

**文件路径**: `/Volumes/lexar/code/d/work/oh/drivers/liteos/hievent/BUILD.gn`

```gn
import("//kernel/liteos_a/liteos.gni")

module_switch = defined(LOSCFG_DRIVERS_HIEVENT)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  sources = [
    "src/hievent_driver.c",
    "src/hiview_hievent.c",
  ]

  public_configs = [ ":public" ]
}

config("public") {
  include_dirs = [ "include" ]
}
```

**target 配置详解**:

| 属性 | 值 | 说明 |
|-----|---|------|
| `module_switch` | `LOSCFG_DRIVERS_HIEVENT` | 构建开关，通过 Kconfig 定义 |
| `module_name` | `drivers/liteos/hievent` | 模块名 |
| `type` | `kernel_module` | 构建为内核模块 |
| `sources` | 2 个源文件 | 见下方源文件列表 |
| `public_configs` | `:public` | 导出 public 配置 |

**源文件列表**:

| 文件 | 路径 | 说明 |
|-----|------|------|
| `hievent_driver.c` | `src/hievent_driver.c` | 字符设备驱动 |
| `hiview_hievent.c` | `src/hiview_hievent.c` | 事件处理 |

**public_config 配置**:

| 属性 | 值 | 说明 |
|-----|---|------|
| `include_dirs` | `["include"]` | 导出头文件搜索路径 |

### GN 变量说明

| 变量名 | 来源 | 说明 |
|-------|------|------|
| `LOSCFG_DRIVERS_HIEVENT` | Kconfig | 布尔开关，默认关闭 |
| `OS_SYS_MEM_ADDR` | LiteOS 内核 | 系统内存起始地址 |

## Kconfig 配置

### DRIVERS_HIEVENT

**文件路径**: `/Volumes/lexar/code/d/work/oh/drivers/liteos/hievent/Kconfig`

```kconfig
config DRIVERS_HIEVENT
    bool "Enable hievent"
    default n
    depends on DRIVERS
    help
      Answer Y to enable LiteOS support hievent.
```

**配置项属性**:

| 属性 | 值 | 说明 |
|-----|---|------|
| 名称 | `DRIVERS_HIEVENT` | 配置符号名 |
| 类型 | bool | 布尔型 |
| 默认值 | n | 默认关闭 |
| 依赖 | `DRIVERS` | 必须先启用 DRIVERS |
| 提示 | "Enable hievent" | 菜单显示文本 |

**配置生效**:
```bash
# 在内核配置菜单中启用
Kernel → Device Drivers → Enable hievent
```

## Makefile 构建

### 兼容 Makefile

**文件路径**: `/Volumes/lexar/code/d/work/oh/drivers/liteos/hievent/Makefile`

```makefile
include $(LITEOSTOPDIR)/config.mk

MODULE_NAME := $(notdir $(shell pwd))

LOCAL_SRCS += $(wildcard ./src/*.c)
LOCAL_FLAGS += -I$(LITEOSTOPDIR)/../../drivers/liteos/hievent/include

include $(MODULE)
```

**Makefile 变量**:

| 变量 | 值 | 说明 |
|-----|---|------|
| `MODULE_NAME` | `hievent` | 模块名 |
| `LOCAL_SRCS` | `src/*.c` | 源文件通配 |
| `LOCAL_FLAGS` | `-I...` | 编译选项，包含头文件路径 |

## 编译产物

### 预期产物

| 产物类型 | 产物名称 | 位置 | 说明 |
|---------|---------|------|------|
| 内核模块 | `drivers_liteos_hievent.o` | out/ | 编译中间产物 |
| 内核模块 | `drivers_liteos_hievent.ko` | out/ | 可加载内核模块 |
| 符号表 | `.syms` | - | 调试符号 |

**说明**: 具体产物路径取决于 OpenHarmony 构建系统配置。

### 产物加载

```bash
# 加载模块
insmod drivers_liteos_hievent.ko

# 查看已加载模块
lsmod

# 查看模块信息
modinfo drivers_liteos_hievent.ko
```

## 依赖关系

### 编译时依赖

| 依赖项 | 来源 | 说明 |
|-------|------|------|
| LiteOS 内核头文件 | `kernel/liteos_a/liteos.gni` | 导入构建模板 |
| los_* 系列头文件 | LiteOS 内核 | 内存、互斥锁、任务等 |
| Linux 头文件 | 交叉编译工具链 | file_operations, poll 等 |

### 运行时依赖

| 依赖项 | 说明 |
|-------|------|
| LiteOS_A 内核 | 运行基础环境 |
| `/dev` 文件系统 | 设备节点挂载点 |
| 系统内存 | 环形缓冲区分配 |

## 构建命令

### GN 构建

```bash
# 设置编译目标
hb set

# 启用 hievent 配置
# 在 menuconfig 中启用: Drivers -> Enable hievent

# 构建
hb build -f
```

### Makefile 构建

```bash
cd hievent
make
```

## 配置验证

### 检查模块是否启用

```bash
# 在内核配置中搜索
grep -r "LOSCFG_DRIVERS_HIEVENT" ./

# 或在构建输出中检查
cat .config | grep HIEVENT
```

### 验证构建产物

```bash
# 检查目标文件存在
ls -la out/kernel/drivers_liteos_hievent.*

# 检查模块符号
nm drivers_liteos_hievent.ko | grep HiviewHievent
```

---

*证据来源*:
- BUILD.gn: `/Volumes/lexar/code/d/work/oh/drivers/liteos/BUILD.gn`, `hievent/BUILD.gn`
- Kconfig: `/Volumes/lexar/code/d/work/oh/drivers/liteos/hievent/Kconfig`
- Makefile: `/Volumes/lexar/code/d/work/oh/drivers/liteos/hievent/Makefile`
