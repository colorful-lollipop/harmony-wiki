# 构建系统 - 07_Build

## 概述

LiteOS-M 使用 **GN (Generate Ninja)** + **Ninja** 作为构建系统。

## 构建入口

| 文件 | 路径 | 说明 |
|------|------|------|
| 根 BUILD.gn | `kernel/liteos_m/BUILD.gn` | 主构建入口 |
| GN 模板 | `kernel/liteos_m/liteos.gni` | 自定义模板定义 |
| 架构配置 | `kernel/liteos_m/config.gni` | 编译器/架构配置 |
| menuconfig | `kernel/liteos_m/Kconfig` | 可配置选项 |

## 主要 Targets

### 根目录 BUILD.gn

```gn
# 静态库 - 整个内核
static_library("libkernel") {
  deps = [ ":modules" ]
}

# 可执行文件 (裸机镜像)
executable("liteos") {
  configs += [ ":public", ":los_config" ]
  deps = [ ":kernel" ]  # 或 "//build/lite:ohos"
}
```

> 参考: `kernel/liteos_m/BUILD.gn:178-189`

### 模块 group

```gn
group("modules") {
  deps = [
    "arch",
    "components",
    "kal",
    "kernel",
    "testsuites",
    "utils",
    HDFTOPDIR,  # "//drivers/hdf_core/adapter/khdf/liteos_m"
  ]
}
```

## GN 模板

### kernel_module 模板

定义位置: `kernel/liteos_m/liteos.gni:81`

```gn
template("kernel_module") {
  # 自动配置 public config
  # 自动创建 module group
  source_set(target_name) {
    # 源文件、依赖、配置
  }
}
```

### config 模板

```gn
config("public") {
  configs = [
    "arch:public",
    "kernel:public",
    "kal:public",
    "components:public",
    "utils:public",
  ]
}
```

### module_group 模板

用于批量声明模块依赖:

```gn
module_group("mygroup") {
  modules = [
    ":module1",
    ":module2",
  ]
}
```

## 各模块 BUILD.gn

### kernel/BUILD.gn

```
kernel/
├── BUILD.gn              # kernel_module 定义
├── include/              # 公共头文件
└── src/
    ├── los_task.c
    ├── los_queue.c
    ├── los_mux.c
    └── mm/
```

### components/BUILD.gn

```gn
# components/BUILD.gn
if (defined(LOSCFG_COMPONENTS_DYNLINK)) {
  deps += [ "dynlink:dynlink" ]
}
if (defined(LOSCFG_COMPONENTS_FS)) {
  deps += [ "fs:fs" ]
}
# ... 其他组件
```

## 编译产物

### 预期产物

| 产物 | 说明 | 路径 |
|------|------|------|
| `libkernel.a` | 静态库 | `out/xxx/libkernel.a` |
| `liteos` | 可执行镜像 | `out/xxx/unstripped/bin/liteos` |

### 安装路径

- 静态库: `//out/xxx/libs/`
- 可执行镜像: `//out/xxx/bin/`

## 构建命令

### 完整构建

```bash
# 使用 hb (HarmonyOS Build)
hb set
hb build
```

### 仅内核构建

```bash
# 设置内核配置
hb set --kernel

# 构建内核
hb build -T :liteos
```

### GN 直接构建

```bash
# 生成 ninja 文件
gn gen out/liteos_m

# 构建
ninja -C out/liteos_m liteos
```

## 配置流程

```
1. menuconfig 配置 (.config)
   └── kernel/liteos_m/Kconfig

2. 生成 config.gni
   └── litel os.gni:40-49
   └── exec_script 调用 genconfig

3. GN 读取 config.gni
   └── 设置 LOSCFG_* 变量

4. 条件编译
   └── if (defined(LOSCFG_COMPONENTS_XXX))
```

## 关键配置变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `LOSCFG_COMPILER_*` | 编译器选择 | GCC/Clang |
| `LOSCFG_ARCH_*` | 架构选择 | ARM/RISC-V |
| `LOSCFG_COMPONENTS_*` | 组件开关 | 视配置 |
| `LOSCFG_DEBUG_VERSION` | 调试版本 | - |

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [常见问题](09_FAQ.md)
