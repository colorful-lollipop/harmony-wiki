# 目录结构

## 顶层目录

```
/drivers_liteos/
├── hievent/              # [核心] 事件日志管理驱动
│   ├── include/         # 对外头文件
│   ├── src/            # 源代码
│   ├── BUILD.gn        # GN 构建配置
│   ├── Kconfig         # 内核配置选项
│   └── Makefile        # 传统 Makefile
├── include/             # [空] 根目录对外头文件
├── tzdriver/            # [未开源] REE/TEE 通信驱动
├── figures/             # 文档资源图片
├── BUILD.gn             # 根构建入口
├── README.md            # 英文说明文档
├── README_zh.md         # 中文说明文档
├── LICENSE              # BSD-3-Clause
└── OAT.xml             # 静态代码检查配置
```

## hievent 模块详解

### 目录结构

```
hievent/
├── include/                    # 驱动层与事件处理层头文件
│   ├── hievent_driver.h        # 字符设备驱动接口
│   └── hiview_hievent.h        # 事件对象接口
├── src/                        # 实现源码
│   ├── hievent_driver.c        # 字符设备驱动实现
│   │   └── 功能: 环形缓冲区、设备注册、文件操作
│   └── hiview_hievent.c        # 事件处理实现
│       └── 功能: 事件构造、Payload 管理、上报
├── BUILD.gn                    # GN 构建配置
├── Kconfig                     # 内核配置菜单
└── Makefile                    # 传统 Makefile (兼容)
```

### 头文件职责

| 文件 | 导出符号 | 职责 |
|-----|---------|------|
| `hievent_driver.h` | `HieventInit`, `HieventWriteInternal` | 驱动层初始化与写入 |
| `hiview_hievent.h` | `HiviewHievent*` 系列函数 | 事件对象生命周期管理 |

### 源文件职责

| 文件 | 代码行数 | 主要功能 |
|-----|---------|---------|
| `hievent_driver.c` | ~390 行 | 字符设备驱动、环形缓冲区实现 |
| `hiview_hievent.c` | ~553 行 | 事件构造、Payload、格式化上报 |

## 文件清单 (不含测试)

### 头文件 (2)

| 文件路径 | 导出的宏 | 导出的类型 | 导出的函数 |
|---------|---------|-----------|-----------|
| `hievent/include/hievent_driver.h` | `CHECK_CODE` | `IdapHeader` | `HieventInit`, `HieventWriteInternal` |
| `hievent/include/hiview_hievent.h` | `MAX_PATH_NUMBER` | `HiviewHievent`, `HiviewHieventPayload` | 7 个事件管理函数 |

### 源文件 (2)

| 文件路径 | 主要全局变量 | 主要静态函数 |
|---------|-------------|-------------|
| `hievent/src/hievent_driver.c` | `g_hieventDev` | `HieventRead`, `HieventWrite`, `HieventPoll` |
| `hievent/src/hiview_hievent.c` | - | `HiviewHieventConvertString`, `LogBufToException` |

### 构建配置文件 (3)

| 文件 | 格式 | 作用 |
|-----|------|------|
| `BUILD.gn` (根) | GN | 根目录构建入口，导入 `kernel/liteos_a/liteos.gni` |
| `hievent/BUILD.gn` | GN | hievent 模块构建，定义 `kernel_module` |
| `Kconfig` | Kconfig | 内核配置菜单，`DRIVERS_HIEVENT` 选项 |

## 忽略的目录

根据项目规范，以下目录不计入源码分析：

| 目录模式 | 原因 |
|---------|------|
| `test/**` | 测试代码 |
| `*_test.*` | 单元测试 |
| `unittest/**` | 单元测试 |
| `fuzz/**` | 模糊测试 |

## 模块依赖关系

```mermaid
graph TD
    A[用户空间] -->|open/read/write| B[/dev/hwlog_exception]
    B --> C[hievent_driver.c]
    C --> D[环形缓冲区 1024B]
    C --> E[LosMux 互斥锁]
    C --> F[LOS_IsUserAddressRange]
    
    G[hiview_hievent.c] --> C
    G --> H[Payload 链表]
    G --> I[LogBufToException]
    
    C --> J[LiteOS 内核]
    J -->|los_memory| K[内存分配]
    J -->|los_mux| L[同步原语]
    J -->|los_task| M[任务管理]
```

---

*证据来源*:
- 目录结构: `ls -laR /Volumes/lexar/code/d/work/oh/drivers/liteos`
- 头文件: `hievent/include/*.h`
- 源文件: `hievent/src/*.c`
- 构建配置: `**/BUILD.gn`, `**/Kconfig`
