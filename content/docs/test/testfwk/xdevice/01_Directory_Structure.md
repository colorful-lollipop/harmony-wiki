# XDevice 目录结构

## 1. 根目录结构

```
xdevice/
├── config/                          # XDevice 配置目录
│     ├── user_config.xml             # XDevice 环境配置文件
│     ├── acts.json                  # ACTS 测试套件配置
│     └── ssts.json                  # SSTS 测试套件配置
│
├── src/                             # 组件源码目录
│     └── xdevice/                   # 主源码包
│           ├── __init__.py          # 主包入口，导出 173 个公共 API
│           ├── __main__.py          # CLI 入口点
│           └── _core/               # 核心实现模块
│
├── plugins/                         # XDevice 扩展插件
│     ├── ohos/                      # OpenHarmony 测试驱动插件
│     │     ├── setup.py            # ohos 插件打包配置
│     │     └── src/
│     │           └── ohos/
│     │                 ├── drivers/     # 测试驱动 (11+ 实现)
│     │                 ├── environment/ # 设备环境管理
│     │                 ├── managers/   # 设备管理器
│     │                 ├── parser/     # 结果解析器 (11+)
│     │                 └── testkit/    # 测试工具包
│     │
│     └── devicetest/                # DeviceTest 测试驱动插件
│           ├── setup.py             # devicetest 插件打包配置
│           ├── main.py              # 主入口类
│           └── core/                # 核心模块
│                 ├── driver/        # 设备测试驱动
│                 ├── runner/        # 测试运行器
│                 ├── controllers/   # 控制器
│                 ├── utils/         # 工具模块
│                 └── report/         # 报告生成
│
├── docs/                           # 文档目录
├── figures/                        # 图片资源
├── lite/                           # 轻量版构建配置
│     └── BUILD.gn
│
├── BUILD.gn                        # GN 构建入口
├── bundle.json                     # 组件配置
├── setup.py                        # Python 包打包脚本
├── run.sh / run.bat               # 启动脚本
├── README.md / README_zh.md        # 项目说明文档
└── OAT.xml                        # 开源合规配置
```

---

## 2. 核心模块结构 (`src/xdevice/_core/`)

```
_core/
├── __init__.py                    # 空初始化文件
│
├── interface.py                   # 核心接口定义 (ABC 抽象基类)
│       ├── IDeviceManager         # 设备管理器接口
│       ├── IDevice                # 设备接口
│       ├── IDriver                # 测试驱动接口
│       ├── IScheduler             # 调度器接口
│       ├── IListener              # 监听器接口
│       ├── IParser                # 结果解析器接口
│       ├── ITestKit               # 测试工具包接口
│       ├── IReporter              # 报告生成器接口
│       ├── IProxy                 # 设备代理扩展接口
│       └── IFilter                # 过滤器接口
│
├── plugin.py                      # 插件管理系统
│       └── Plugin                 # 插件类型常量定义
│
├── constants.py                   # 常量和枚举定义
│       ├── DeviceOsType           # 设备 OS 类型
│       ├── TestType               # 测试类型
│       ├── DeviceTestType         # 设备测试类型
│       ├── HostDrivenTestType    # 主机驱动测试类型
│       ├── ConfigConst            # 配置键名常量
│       └── ReportConst            # 报告相关常量
│
├── error.py                       # 错误码定义
│       └── Error / ErrorCategory  # 错误处理类
│
├── exception.py                   # 异常类定义
│       ├── ParamError             # 参数错误
│       ├── DeviceError            # 设备错误
│       └── HdcError              # HDC 相关错误
│
├── logger.py                      # 日志系统
│       ├── LogQueue               # 日志队列
│       └── EncryptFileHandler     # 加密日志处理器
│
├── variables.py                   # 全局变量管理
│       └── Variables              # 配置变量单例
│
├── utils.py                       # 通用工具函数
│       ├── exec_cmd()             # 执行外部命令
│       ├── get_device_log_file()  # 获取设备日志路径
│       └── ...
│
├── common.py                      # 通用定义
│
├── command/                       # 命令行交互模块
│       ├── __init__.py
│       └── console.py             # Console 类，主控制台入口
│
├── config/                        # 配置管理模块
│       ├── __init__.py
│       ├── config_manager.py      # UserConfigManager 类
│       └── resource_manager.py    # ResourceManager 类
│
├── context/                       # 上下文管理模块
│       ├── __init__.py
│       ├── center.py              # Context 单例类
│       ├── impl.py                # BaseScheduler 实现
│       ├── handler.py             # 结果处理函数
│       ├── life_stage.py          # 生命周期事件类
│       ├── proxy.py               # 代理和连接类
│       └── ...
│
├── driver/                        # 测试驱动模块
│       ├── __init__.py
│       └── parser_lite.py         # ShellHandler 类
│
├── environment/                   # 设备环境管理模块
│       ├── __init__.py
│       ├── manager_env.py         # EnvironmentManager 类
│       ├── env_pool.py           # EnvPool, DeviceSelector
│       ├── device_state.py       # 设备状态枚举
│       └── device_monitor.py      # DeviceStateMonitor 类
│
├── executor/                      # 测试执行器模块
│       ├── __init__.py
│       ├── scheduler.py           # Scheduler 主调度器
│       ├── concurrent.py         # 并发执行控制
│       ├── request.py            # Task, Request 定义
│       ├── source.py             # 测试源管理
│       ├── bean.py               # 结果 Bean
│       ├── abs.py                # 监听器抽象类
│       └── listener.py           # 具体监听器实现
│
├── report/                        # 测试报告模块
│       ├── __init__.py
│       ├── __main__.py           # 报告工具独立入口
│       ├── result_reporter.py    # ResultReporter 类
│       ├── suite_reporter.py     # SuiteReporter 类
│       ├── reporter_helper.py    # 报告辅助类
│       ├── repeater_helper.py   # RepeatHelper 类
│       └── encrypt.py            # RSA 加密相关
│
├── testkit/                       # 测试工具包模块
│       ├── __init__.py
│       ├── kit.py                # 测试工具函数集合
│       └── json_parser.py        # JsonParser 类
│
├── cluster/                       # 分布式集群模块
│       ├── __init__.py
│       ├── __main__.py           # 集群服务入口 (FastAPI)
│       ├── models.py             # SQLModel 数据模型
│       ├── runner.py             # Runner 类
│       ├── utils.py              # 工具类
│       │
│       ├── controller/           # 控制器 (主节点)
│       │       ├── __init__.py
│       │       ├── main.py       # 控制器主逻辑
│       │       ├── api.py        # FastAPI 路由
│       │       ├── db.py         # 数据库引擎
│       │       ├── crud.py       # 数据库 CRUD 操作
│       │       └── handler.py    # BlockHandler, TaskHandler
│       │
│       └── worker/                # 工作节点
│               ├── __init__.py
│               ├── main.py       # 工作节点启动
│               ├── api.py        # 工作节点 API
│               ├── task_manager.py  # TaskManager 类
│               └── task_runner.py   # WorkerRunner 类
│
└── resource/                      # 资源文件目录
        ├── config/user_config.xml  # 默认配置文件
        └── template/               # HTML 报告模板
                ├── report.html
                ├── summary_report.html
                └── static/
                        ├── css/
                        ├── js/
                        └── components/
```

---

## 3. 插件目录结构

### 3.1 ohos 插件

```
plugins/ohos/
├── setup.py                      # 打包配置，定义 30+ entry_points
│
├── src/
│     └── ohos/
│           ├── __init__.py        # 版本定义 (VERSION = '5.0.6.100')
│           ├── constants.py       # 常量定义
│           ├── error.py           # 错误消息
│           ├── exception.py       # 异常定义
│           ├── utils.py           # 工具函数
│           │
│           ├── config/
│           │       └── config_manager.py  # OHOS 专用配置管理
│           │
│           ├── drivers/           # 测试驱动 (11+ 实现)
│           │       ├── cpp_driver.py           # C++ 测试驱动
│           │       ├── cpp_driver_lite.py      # C++ Lite 测试驱动
│           │       ├── jsunit_driver.py        # JSUnit 测试驱动
│           │       ├── c_driver_lite.py       # C Lite 测试驱动
│           │       ├── oh_jsunit_driver.py    # OH JSUnit 驱动
│           │       ├── oh_kernel_driver.py    # 内核测试驱动
│           │       ├── oh_yara_driver.py      # YARA 扫描驱动
│           │       ├── ltp_posix_driver.py    # LTP POSIX 驱动
│           │       ├── vulkan_driver.py       # Vulkan 测试驱动
│           │       ├── opensource_driver_lite.py
│           │       └── build_only_driver_lite.py
│           │
│           ├── environment/      # 设备环境管理
│           │       ├── device.py           # 标准设备实现
│           │       ├── device_lite.py      # Lite 设备实现
│           │       ├── native_device.py
│           │       ├── emulator.py
│           │       ├── dmlib.py            # HDC 设备管理库
│           │       └── dmlib_lite.py
│           │
│           ├── managers/          # 设备管理器
│           │       ├── manager_device.py
│           │       └── manager_lite.py
│           │
│           ├── executor/         # 执行器
│           │       ├── listener.py
│           │       └── bean.py
│           │
│           ├── parser/           # 结果解析器 (11+)
│           │       ├── cpp_parser.py
│           │       ├── cpp_parser_lite.py
│           │       ├── jsunit_parser.py
│           │       ├── jsunit_parser_lite.py
│           │       ├── junit_parser.py
│           │       ├── oh_jsunit_parser.py
│           │       ├── oh_kernel_parser.py
│           │       ├── oh_rust_parser.py
│           │       ├── oh_yara_parser.py
│           │       ├── vulkan_parser.py
│           │       ├── c_parser_lite.py
│           │       ├── opensource_parser_lite.py
│           │       ├── build_only_parser_lite.py
│           │       └── constants.py
│           │
│           └── testkit/          # 测试工具包
│                   ├── kit.py              # 标准设备工具包
│                   └── kit_lite.py         # Lite 设备工具包
│
└── setup.py                      # 插件打包脚本
```

### 3.2 devicetest 插件

```
plugins/devicetest/
├── setup.py                      # 打包配置
│
├── __init__.py                   # 版本定义 (VERSION = '5.0.6.100')
│
├── main.py                       # DeviceTest, DeviceTestSuite 主类
│
├── constants.py                  # 常量定义
├── error.py                     # 错误定义
│
├── core/                         # 核心模块
│       ├── constants.py
│       ├── exception.py
│       ├── error_message.py
│       ├── record.py
│       ├── report.py
│       ├── result_upload.py
│       ├── test_case.py
│       ├── variables.py
│       │
│       ├── suite/                # 测试套件
│       │       └── test_suite.py
│       │
│       └── variables.py
│
├── driver/                       # 设备测试驱动
│       ├── device_test.py         # 设备测试驱动实现
│       └── windows.py            # Windows 设备驱动
│
├── runner/                       # 测试运行器
│       ├── test_runner.py
│       └── prepare.py
│
├── controllers/                  # 控制器
│       └── tools/
│               └── screen_agent.py
│
├── utils/                        # 工具模块
│       ├── util.py
│       ├── file_util.py
│       ├── time_util.py
│       ├── type_utils.py
│       └── img_util.py
│
├── log/                          # 日志模块
│       ├── logger.py
│       └── variables.py
│
├── report/                       # 报告生成
│       └── generation.py
│
└── res/                          # 资源文件
        └── template/
                └── case.html      # 报告模板
```

---

## 4. 模块职责速查表

| 模块 | 路径 | 主要类 | 职责 |
|------|------|--------|------|
| command | `_core/command/` | Console | 命令行解析与交互 |
| config | `_core/config/` | UserConfigManager | XML 配置解析 |
| driver | `_core/driver/` | ShellHandler | 驱动解析辅助 |
| environment | `_core/environment/` | EnvironmentManager, EnvPool | 设备发现与管理 |
| executor | `_core/executor/` | Scheduler | 任务调度与并发 |
| report | `_core/report/` | ResultReporter | 报告生成 |
| testkit | `_core/testkit/` | JsonParser, Kit | 测试工具 |
| context | `_core/context/` | Context | 全局状态管理 |
| cluster | `_core/cluster/` | Controller, Worker | 分布式测试 |
| resource | `_core/resource/` | ResourceManager | 资源文件 |

---

## 5. 入口文件清单

| 文件 | 用途 | 启动方式 |
|------|------|---------|
| `src/xdevice/__main__.py` | CLI 主入口 | `python -m xdevice` |
| `src/xdevice/_core/report/__main__.py` | 报告工具入口 | `python -m xdevice._core.report` |
| `src/xdevice/_core/cluster/__main__.py` | 集群服务入口 | `python -m xdevice._core.cluster` |
| `plugins/ohos/setup.py` | ohos 插件打包 | `python setup.py sdist` |
| `plugins/devicetest/setup.py` | devicetest 插件打包 | `python setup.py sdist` |

---

## 相关文档

- [架构说明](02_Architecture.md)
- [模块详情](03_Modules.md)
- [配置说明](05_Configuration.md)
- [使用指南](07_Usage.md)
