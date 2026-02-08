# 目录结构

## 3.1 顶层目录

```
developer_test/
├── src/                           # [核心框架源码] 主要业务逻辑
├── aw/                            # [静态库] C++ 测试库
├── libs/                          # [测试库] 特殊测试类型支持
├── config/                        # [配置文件] XML 配置模板
├── examples/                      # [示例] 测试用例示例
├── third_party/                   # [第三方适配] 依赖配置
├── BUILD.gn                       # [构建入口] GN 构建文件
├── bundle.json                    # [组件配置] 部件配置
├── start.sh                       # [启动脚本] Linux 入口
├── start.bat                      # [启动脚本] Windows 入口
├── install.sh                     # [安装脚本] 环境配置
└── wiki/                          # [本文档] 工程文档
```

## 3.2 源码目录 (src/)

### 3.2.1 入口模块 (src/main)

| 文件 | 说明 | 关键代码 |
|------|------|---------|
| `__main__.py` | 程序入口 | `Console().console(sys.argv)` |
| `_init_global_config.py` | 全局配置初始化 | - |

### 3.2.2 核心模块 (src/core/)

```
src/core/
├── command/                      # [命令处理] 用户交互
│   ├── console.py               # 控制台入口，参数解析
│   ├── run.py                   # 测试执行命令
│   ├── gen.py                   # 代码生成命令
│   ├── display.py               # 显示帮助/信息
│   ├── parameter.py             # 命令参数定义
│   ├── distribute_execute.py    # 分布式执行
│   └── distribute_utils.py      # 分布式工具
│
├── driver/                       # [设备驱动] 测试执行
│   ├── drivers.py              # C++/JS 测试驱动
│   ├── openharmony.py          # 标准设备驱动
│   ├── lite_driver.py          # Lite 设备驱动
│   └── parser.py               # 结果解析器
│
├── build/                        # [构建管理] 编译控制
│   ├── build_manager.py         # 构建入口
│   ├── build_testcases.py      # 用例编译
│   ├── build_lite_manager.py   # Lite 编译
│   ├── select_targets.py       # 目标选择
│   └── pretreat_targets.py     # 目标预处理
│
├── config/                       # [配置管理]
│   ├── config_manager.py       # 配置管理器
│   ├── resource_manager.py     # 资源配置
│   └── parse_parts_config.py   # 配置解析
│
├── testcase/                    # [用例管理]
│   └── testcase_manager.py    # 用例管理
│
├── testkit/                     # [工具包]
│   └── kit_lite.py            # Lite 工具
│
├── arkts_tdd/                   # [ArkTS 支持]
│   ├── arkts_tdd_execute/      # ArkTS 执行
│   └── artts_tdd_report/      # ArkTS 报告
│
├── common.py                    # [公共函数]
├── utils.py                     # [工具函数]
├── constants.py                 # [常量定义]
└── exception.py                 # [异常定义]
```

## 3.3 静态库目录 (aw/)

### 3.3.1 C++ 静态库 (aw/cxx/)

```
aw/cxx/
├── distributed/                 # [分布式测试库]
│   ├── BUILD.gn               # 构建配置
│   ├── distributed.h          # 主头文件
│   ├── distributed_agent.cpp  # 代理实现
│   ├── distributed_agent.h   # 代理头文件
│   ├── distributed_cfg.cpp   # 配置实现
│   ├── distributed_cfg.h    # 配置头文件
│   ├── distributed_major.cpp # 主逻辑
│   ├── distributed_major.h  # 主逻辑头文件
│   └── utils/
│       └── csv_transform_xml.h # CSV转XML工具
│
└── hwext/                      # [性能测试库]
    ├── BUILD.gn               # 构建配置
    ├── perf.h                # 性能测试头文件
    └── perf.cpp              # 性能测试实现
```

### 3.3.2 Python 静态库 (aw/python/)

**证据**: `src/` 目录结构显示使用 Python 实现核心逻辑

## 3.4 测试库目录 (libs/)

```
libs/
├── arkts1.2/                   # [ArkTS 1.2 测试支持]
├── benchmark/                   # [Benchmark 测试库]
│   └── README_zh.md           # 使用文档
├── fuzzlib/                    # [Fuzz 测试库]
│   └── README_zh.md           # 使用文档
└── js_template/                # [JS 测试模板]
```

## 3.5 配置文件目录 (config/)

| 文件 | 说明 | 关键配置 |
|------|------|---------|
| `user_config.xml` | 用户配置 | 设备、NFS、覆盖率 |
| `framework_config.xml` | 框架配置 | 产品形态、测试类型 |
| `build_config.xml` | 构建配置 | 构建模板 |
| `filter_config.xml` | 过滤配置 | 用例过滤规则 |
| `fuzz_config.xml` | Fuzz 配置 | Fuzz 参数 |

## 3.6 示例目录 (examples/)

### 3.6.1 示例列表

| 目录 | 说明 | 测试类型 |
|------|------|---------|
| `calculator/` | 计算器示例 | UT, FUZZ, BENCHMARK |
| `app_info/` | 应用信息示例 | UT (JS) |
| `detector/` | 探测器示例 | UT |
| `sleep/` | 睡眠测试示例 | PERF |
| `distributedb/` | 分布式数据库示例 | DST |
| `lite/` | Lite 设备示例 | - |
| `stagetest/` | Stage 模型示例 | ACTS |

### 3.6.2 calculator 示例结构

```
examples/calculator/
├── BUILD.gn                   # 构建配置
├── include/
│   └── calculator.h           # 头文件
├── src/
│   └── calculator.cpp         # 源文件
└── test/
    ├── BUILD.gn              # 测试构建
    ├── unittest/             # 单元测试
    │   ├── common/           # 公共用例
    │   └── phone/            # phone 形态用例
    ├── fuzztest/             # Fuzz 测试
    └── benchmarktest/        # 性能测试
```

## 3.7 第三方依赖 (third_party/)

```
third_party/
└── lib/
    └── [编译配置]
```

## 3.8 模块职责速查

| 模块 | 职责 | 稳定性 |
|------|------|--------|
| `src/core/command/*` | 用户交互 | 稳定 |
| `src/core/build/*` | 编译控制 | 稳定 |
| `src/core/driver/*` | 设备驱动 | 稳定 |
| `src/core/config/*` | 配置管理 | 稳定 |
| `aw/cxx/*` | 测试库 | 稳定 |
| `libs/*` | 特殊测试支持 | 稳定 |

## 3.9 稳定性说明

### 稳定接口

- `src/core/command/` - 命令层 API 稳定
- `src/core/config/config_manager.py` - 配置管理 API 稳定

### 内部接口

- `src/core/build/*` - 构建层为内部使用
- `src/core/driver/*` - 驱动层为内部使用

## 3.10 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 系统架构
- [04_Configuration.md](04_Configuration.md) - 配置说明
- [05_Build_System.md](05_Build_System.md) - 构建系统
- [07_Examples.md](07_Examples.md) - 示例说明
