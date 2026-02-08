# XDevice 模块详解

## 1. command 模块 - 命令行交互

### 1.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/command/` |
| 关键文件 | `console.py` (45KB) |
| 主要类 | `Console` |
| 职责 | 命令行参数解析、命令分发、用户交互 |

### 1.2 核心类：`Console`

```python
class Console:
    """单例模式，主控制台类"""
    
    def argument_parser(self):
        """使用 argparse 解析命令行参数"""
        
    def command_parser(self):
        """命令分发处理"""
        
    def main_loop(self):
        """主循环，持续接收用户输入"""
```

**证据来源**：`src/xdevice/_core/command/console.py`

### 1.3 支持的命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `help` | 显示帮助信息 | `help run` |
| `list` | 列出设备和任务 | `list devices` |
| `run` | 执行测试 | `run acts -l module1` |
| `quit` | 退出框架 | `quit` |
| `tool` | 工具命令 | `tool` |

### 1.4 参数组设计

```python
# 互斥参数组 1：测试指定方式
group1 = parser.add_mutually_exclusive_group()
group1.add_argument('-l', '--testlist', action=SplicingAction)
group1.add_argument('-tc', '--testcase', nargs='+')
group1.add_argument('-tf', '--testfile')

# 互斥参数组 2：配置来源
group2 = parser.add_mutually_exclusive_group()
group2.add_argument('-c', '--config')
group2.add_argument('-env', '--environment')

# 互斥参数组 3：重试机制
group3 = parser.add_mutually_exclusive_group()
group3.add_argument('--repeat', type=int)
group3.add_argument('--retry')
group3.add_argument('--auto_retry')
```

### 1.5 关键方法

| 方法 | 功能 |
|------|------|
| `_process_command_run()` | 处理 run 命令 |
| `_process_command_list()` | 处理 list 命令 |
| `_verify_input_args()` | 验证输入参数 |

---

## 2. config 模块 - 配置管理

### 2.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/config/` |
| 关键文件 | `config_manager.py` (467行) |
| 主要类 | `UserConfigManager` |
| 职责 | XML 配置文件解析、配置验证 |

### 2.2 核心类：`UserConfigManager`

```python
class UserConfigManager:
    """用户配置管理器，负责解析 user_config.xml"""
    
    def __init__(self, config_file="", env=""):
        """初始化，加载配置文件"""
        
    def get_element_cfg(self, tag):
        """获取指定标签的配置"""
        
    def get_devices(self, target_name):
        """获取设备配置列表"""
        
    def _verify_duplicate(self, items):
        """验证重复项"""
```

**证据来源**：`src/xdevice/_core/config/config_manager.py`

### 2.3 配置解析方法

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `environment` | `list[dict]` | 设备环境配置 |
| `testcases` | `dict` | 测试用例目录配置 |
| `resource` | `dict` | 资源路径配置 |
| `devicelog` | `dict` | 设备日志配置 |
| `cluster` | `dict` | 集群配置 |

---

## 3. driver 模块 - 测试驱动

### 3.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/driver/` |
| 关键文件 | `parser_lite.py` |
| 主要类 | `ShellHandler` |
| 职责 | 测试驱动解析、Shell 命令处理 |

### 3.2 核心类：`ShellHandler`

```python
class ShellHandler:
    """Shell 命令处理器"""
    
    def __init__(self, device):
        """初始化，关联设备"""
        
    def execute_command(self, command, timeout=None):
        """执行 Shell 命令"""
```

### 3.3 驱动接口定义

```python
# 证据来源：src/xdevice/_core/interface.py

class IDriver(ABC):
    """测试驱动接口"""
    
    @abstractmethod
    def __check_environment__(self, device_options):
        """检查运行环境"""
        
    @abstractmethod
    def __check_config__(self, config):
        """检查配置"""
        
    @abstractmethod
    def __execute__(self, request):
        """执行测试"""
        
    @abstractmethod
    def __result__(self):
        """获取结果"""
```

---

## 4. environment 模块 - 设备环境管理

### 4.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/environment/` |
| 关键文件 | `env_pool.py` (441行), `manager_env.py` (373行) |
| 主要类 | `EnvironmentManager`, `EnvPool`, `DeviceSelector` |
| 职责 | 设备发现、设备分配、状态监控 |

### 4.2 核心类

| 类 | 职责 |
|---|------|
| `EnvironmentManager` | 环境管理器，协调设备初始化 |
| `EnvPool` | 设备连接池，管理设备实例 |
| `DeviceSelector` | 设备选择器，支持灵活的设备过滤 |
| `DeviceStateMonitor` | 设备状态监控器 |

### 4.3 设备状态枚举

```python
# 证据来源：src/xdevice/_core/environment/device_state.py

class DeviceState(Enum):
    """设备状态枚举"""
    BOOTLOADER = "bootloader"      # 引导模式
    OFFLINE = "offline"            # 离线
    ONLINE = "device"              # 在线
    CONNECTED = "connected"        # 已连接
    RECOVERY = "recovery"          # 恢复模式
    UNAUTHORIZED = "Unauthorized"  # 未授权
```

### 4.4 设备连接方式

| 方式 | 协议 | 使用场景 |
|------|------|---------|
| HDC | USB/TCP | 标准系统设备 |
| 串口 | pySerial | LiteOS 设备 |
| SSH | Paramiko | NFS 文件传输 |
| Telnet | - | 远程设备 |

---

## 5. executor 模块 - 测试执行器

### 5.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/executor/` |
| 关键文件 | `scheduler.py` (32KB), `concurrent.py` (36KB) |
| 主要类 | `Scheduler`, `Concurrent`, `DriversThread` |
| 职责 | 任务调度、并发控制、结果收集 |

### 5.2 核心类

| 类 | 职责 |
|---|------|
| `Scheduler` | 主调度器，任务分发 |
| `Concurrent` | 并发控制器 |
| `DriversThread` | 设备测试执行线程 |
| `ModuleThread` | 模块级执行线程 |
| `QueueMonitorThread` | 结果队列监控线程 |

### 5.3 调度器类型

```python
# 证据来源：src/xdevice/_core/constants.py

class SchedulerType:
    """调度器类型枚举"""
    scheduler = "Scheduler"       # 默认并行调度
    module = "module"             # 模块级调度
    synchronize = "synchronize"   # 同步调度
```

### 5.4 任务结构

```python
class Task:
    """任务描述符"""
    root: Descriptor              # 测试根描述符
    test_drivers: List           # 测试驱动列表
    config: Config               # 任务配置
```

---

## 6. report 模块 - 测试报告

### 6.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/report/` |
| 关键文件 | `result_reporter.py` (37KB), `reporter_helper.py` (70KB) |
| 主要类 | `ResultReporter`, `SuiteReporter`, `DataHelper` |
| 职责 | 测试结果解析、报告生成、报告加密 |

### 6.2 核心类

| 类 | 职责 |
|---|------|
| `ResultReporter` | 结果报告生成器 |
| `SuiteReporter` | 套件报告生成器 |
| `DataHelper` | 数据处理辅助类 |
| `EncryptFileHandler` | 加密日志处理器 |

### 6.3 报告监听器层次

```
IListener (ABC)
    └── AbsReportListener
          └── ReportEventListener
                └── UniversalReportListener
                      └── PlusReportListener
                            └── ReportListener
```

### 6.4 报告文件结构

```
reports/
├── result/                    # XML 结果文件
│   └── module_name.xml
├── log/                       # 日志文件
│   ├── task_log.log
│   └── module_run.log
├── summary_report.html        # 汇总报告
├── summary_data_report.xml    # 数据报告
├── task_info.record           # 任务记录
└── summary.ini               # 摘要信息
```

---

## 7. testkit 模块 - 测试工具包

### 7.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/testkit/` |
| 关键文件 | `kit.py`, `json_parser.py` (138行) |
| 主要类 | `JsonParser` |
| 职责 | JSON 配置解析、测试工具函数 |

### 7.2 核心类：`JsonParser`

```python
class JsonParser:
    """JSON 配置文件解析器"""
    
    def __init__(self, path_or_content):
        """初始化，解析 JSON"""
        self._do_parse(path_or_content)
        
    def _check_config(self, json_content):
        """验证 JSON 配置结构"""
```

### 7.3 工具函数

| 函数 | 用途 |
|------|------|
| `junit_para_parse()` | JUnit 参数解析 |
| `gtest_para_parse()` | GTest 参数解析 |
| `get_app_name_by_tool()` | 通过工具获取应用名 |
| `remount()` | 重新挂载设备 |
| `get_kit_instances()` | 获取 Kit 实例列表 |

---

## 8. context 模块 - 上下文管理

### 8.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/context/` |
| 关键文件 | `center.py`, `impl.py`, `life_stage.py` |
| 主要类 | `Context`, `BaseScheduler`, `LifeCycle` |
| 职责 | 全局状态管理、生命周期事件 |

### 8.2 核心类

| 类 | 职责 |
|---|------|
| `Context` | 单例上下文管理器 |
| `BaseScheduler` | 调度器基类实现 |
| `LifeCycle` | 生命周期事件枚举 |

### 8.3 生命周期事件

```python
class LifeStage:
    """生命周期枚举"""
    task_start = "TaskStart"      # 任务开始
    task_end = "TaskEnd"          # 任务结束
    case_start = "CaseStart"      # 用例开始
    case_end = "CaseEnd"          # 用例结束
```

---

## 9. cluster 模块 - 分布式测试

### 9.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/cluster/` |
| 关键文件 | `__main__.py` (FastAPI), `controller/handler.py` |
| 主要类 | `TaskHandler`, `BlockHandler`, `TaskManager` |
| 职责 | 分布式任务分发、集群管理 |

### 9.2 控制器组件

| 类 | 职责 |
|---|------|
| `TaskHandler` | 任务处理器，拆分测试块 |
| `BlockHandler` | 任务块处理器，设备匹配 |
| `DeviceMatcher` | 设备匹配器 |

### 9.3 工作节点组件

| 类 | 职责 |
|---|------|
| `TaskManager` | 任务管理器 |
| `WorkerRunner` | 工作节点运行器 |
| `SubProcess` | 子进程管理 |

---

## 10. resource 模块 - 资源管理

### 10.1 模块概述

| 属性 | 值 |
|------|-----|
| 路径 | `src/xdevice/_core/resource/` |
| 关键文件 | `config/user_config.xml`, `template/` |
| 主要类 | `ResourceManager` |
| 职责 | 资源文件管理、报告模板 |

### 10.2 资源结构

```
resource/
├── config/
│   └── user_config.xml           # 默认配置文件
└── template/
    ├── report.html              # 报告模板
    ├── summary_report.html     # 汇总报告模板
    └── static/                  # 静态资源
        ├── css/
        ├── js/
        └── components/
```

---

## 相关文档

- [架构说明](02_Architecture.md)
- [配置说明](05_Configuration.md)
- [使用指南](07_Usage.md)
