# 可选组件 - 06_Components

## 概述

`components/` 目录包含内核的可选组件，可根据需求通过 menuconfig 裁剪。

## 组件列表

| 组件 | 路径 | 功能 | 安全相关 |
|------|------|------|----------|
| **dynlink** | `components/dynlink/` | 动态加载 | **是** |
| **fs** | `components/fs/` | 文件系统 | - |
| **net** | `components/net/` | 网络功能 | - |
| **shell** | `components/shell/` | Shell 命令 | - |
| **backtrace** | `components/backtrace/` | 栈回溯 | - |
| **cppsupport** | `components/cppsupport/` | C++ 支持 | - |
| **cpup** | `components/cpup/` | CPU 统计 | - |
| **exchook** | `components/exchook/` | 异常钩子 | - |
| **lmk** | `components/lmk/` | 低内存杀手 | - |
| **lms** | `components/lms/` | 内存 sanitizer | - |
| **power** | `components/power/` | 电源管理 | - |
| **signal** | `components/signal/` | 信号处理 | - |
| **trace** | `components/trace/` | 跟踪工具 | - |

## 动态加载组件 (dynlink)

### 概述

`components/dynlink/` 提供动态加载和链接功能，允许运行时加载共享库。

### 关键文件

```
components/dynlink/
├── los_dynlink.c       # 动态加载实现 (26KB)
├── los_dynlink.h       # 对外接口
└── BUILD.gn
```

### 安全要求

> **重要**: 根据 README.md:73，动态加载的共享库必须进行签名验证或来源限制。

### 主要 API

| API | 说明 |
|-----|------|
| `LOS_FindModule()` | 查找模块 |
| `LOS_LoadModule()` | 加载模块 |
| `LOS_UnloadModule()` | 卸载模块 |
| `LOS_GetSym()` | 获取符号地址 |

## 文件系统组件 (fs)

### 概述

`components/fs/` 提供文件系统支持。

### 子组件

| 子组件 | 说明 |
|--------|------|
| FatFs | FAT 文件系统支持 |
| littlefs | 小型闪存文件系统 |

### 头文件

```
components/fs/
├── fs.h              # 通用文件系统接口
└── ...
```

## 网络组件 (net)

### 概述

`components/net/` 提供网络功能支持。

### 依赖

- 依赖 third_party/lwip (LwIP TCP/IP 栈)

## Shell 组件 (shell)

### 概述

`components/shell/` 提供命令行 Shell 支持。

### 功能

- 内置命令 (task, mem, queue 等)
- 用户自定义命令
- 命令行编辑

## 安全相关组件

### security

`components/security/` 提供安全功能模块。

### 潜在安全功能

- 权限检查
- 访问控制
- 加密/解密

### 启用方式

```
components/security/BUILD.gn
```

## 配置方式

### menuconfig 启用

```bash
make menuconfig
```

导航路径:
```
Kernel --->
    Enable Components --->
        [*] Enable dynlink
        [*] Enable fs
        [*] Enable net
        ...
```

### 关闭组件

```
Kernel --->
    Enable Components --->
        [ ] Enable shell
```

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [安全评审](08_Security.md)
