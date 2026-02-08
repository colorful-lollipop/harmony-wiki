# XDevice 项目概览

## 1. 项目简介

### 1.1 项目定位

**XDevice** 是 OpenHarmony 测试框架的核心组件，提供用例执行所依赖的相关服务。

| 属性 | 值 |
|------|-----|
| 项目名称 | xdevice |
| 子系统 | testfwk |
| 版本 | 5.0.6.100 |
| bundle.json 版本 | 2.30.0 |
| 许可协议 | Apache License 2.0 |
| 适配系统 | mini, small, standard |

### 1.2 核心能力

XDevice 测试框架提供以下核心能力：

1. **命令行交互** (`command` 模块)
   - 支持 `help`、`list`、`run` 等命令
   - 灵活的参数解析和命令分发

2. **测试执行** (`executor` 模块)
   - 支持多种测试类型（单元测试、系统测试、性能测试等）
   - 并发执行和调度管理

3. **设备管理** (`environment` 模块)
   - 支持多种设备连接方式（USB-HDC、串口、TCP/IP）
   - 设备发现、分配、状态监控

4. **报告生成** (`report` 模块)
   - HTML 格式可视化报告
   - XML/JSON 格式机器可读报告
   - 支持报告加密

5. **分布式测试** (`cluster` 模块)
   - 控制器-工作节点架构
   - 任务分配和结果收集

---

## 2. 主要模块

XDevice 由 10 个核心模块组成：

| 模块 | 路径 | 职责 |
|------|------|------|
| **command** | `src/xdevice/_core/command/` | 命令行交互，Console 类 |
| **config** | `src/xdevice/_core/config/` | 配置管理，UserConfigManager |
| **driver** | `src/xdevice/_core/driver/` | 测试驱动解析 |
| **environment** | `src/xdevice/_core/environment/` | 设备环境管理，EnvPool |
| **executor** | `src/xdevice/_core/executor/` | 测试执行器，Scheduler |
| **report** | `src/xdevice/_core/report/` | 报告生成，ResultReporter |
| **testkit** | `src/xdevice/_core/testkit/` | 测试工具包 |
| **context** | `src/xdevice/_core/context/` | 上下文管理，单例模式 |
| **cluster** | `src/xdevice/_core/cluster/` | 分布式测试，FastAPI |
| **resource** | `src/xdevice/_core/resource/` | 资源文件管理 |

**证据来源**：
- `src/xdevice/__init__.py`：主包入口，导出 173 个公共 API
- `src/xdevice/_core/constants.py`：模块定义和常量

---

## 3. 技术栈

### 3.1 运行环境

| 依赖 | 版本要求 | 用途 |
|------|---------|------|
| Python | >= 3.7.5 | 运行环境 |
| pySerial | >= 3.3 | 串口通信 |
| Paramiko | >= 2.7.1 | SSH/SFTP 通信 |
| RSA | >= 4.0 | 加密模块 |

**证据来源**：`README.md` 运行环境要求

### 3.2 可选依赖

| 依赖 | Python 版本 | 用途 |
|------|------------|------|
| cryptography | 3.10+ | 加密增强 |
| psutil | 3.10+ | 系统监控 |
| fastapi | 3.10+ | Web 服务 |
| uvicorn | 3.10+ | ASGI 服务器 |
| jinja2 | - | 模板渲染 |
| numpy, pillow, opencv-python | - | 图像处理 |

**证据来源**：`setup.py` extras_require 定义

### 3.3 构建工具

- **GN**：构建配置
- **setuptools**：Python 包打包

---

## 4. 插件系统

### 4.1 插件类型

XDevice 采用插件化架构，支持 9 种插件类型：

| 插件类型 | 标识符 | 说明 |
|---------|--------|------|
| SCHEDULER | "scheduler" | 调度器插件 |
| DRIVER | "driver" | 测试驱动插件 |
| DEVICE | "device" | 设备类型插件 |
| MANAGER | "manager" | 设备管理器插件 |
| PARSER | "parser" | 结果解析器插件 |
| LISTENER | "listener" | 事件监听器插件 |
| TEST_KIT | "testkit" | 测试工具包插件 |
| REPORTER | "reporter" | 报告生成器插件 |
| LOG | "log" | 日志处理器插件 |

**证据来源**：`src/xdevice/_core/plugin.py` 第 19-27 行

### 4.2 官方插件

#### ohos 插件

| 组件 | 数量 | 说明 |
|------|------|------|
| 设备驱动 | 11 | cpp_driver, jsunit_driver, oh_jsunit_driver 等 |
| 设备类型 | 2 | device（标准设备）、device_lite（轻量设备） |
| 设备管理器 | 2 | manager_device、manager_lite |
| 结果解析器 | 11 | cpp_parser、jsunit_parser 等 |
| 测试工具包 | 2 | kit、kit_lite |

**证据来源**：`plugins/ohos/setup.py` entry_points 定义

#### devicetest 插件

- 黑盒自动化测试驱动
- Windows 设备支持
- 图像识别和 UI 测试

---

## 5. 设备连接支持

### 5.1 连接方式

| 连接类型 | 模块 | 协议 | 用途 |
|---------|------|------|------|
| USB-HDC | `ohos.environment.dmlib.py` | HDC 协议 | 标准设备连接 |
| 串口 | `ohos.environment.device_lite.py` | pySerial | LiteOS 设备 |
| SSH/SFTP | `ohos.drivers.cpp_driver_lite.py` | Paramiko | NFS 文件传输 |
| Telnet | `ohos.environment.device_lite.py` | Telnet | 远程设备 |

### 5.2 支持的设备类型

| 设备标签 | 系统类型 | 说明 |
|---------|---------|------|
| ohos | standard | 标准系统设备 |
| wifiiot | lite | WiFi IoT 设备 |
| ipcamera | lite | IP 摄像头设备 |
| watchGT | lite | 智能手表设备 |

---

## 6. 测试类型支持

### 6.1 测试执行类型

| 类型 | 标识符 | 说明 |
|------|--------|------|
| device_test | "device" | 设备上运行的测试 |
| host_test | "host" | 主机上运行的测试 |
| host_driven_test | "hostdriven" | 主机驱动设备测试 |

### 6.2 测试类型

| 类型 | 标识符 | 说明 |
|------|--------|------|
| unittest | "unittest" | 单元测试 |
| moduletest | "mst" | 模块测试 |
| systemtest | "systemtest" | 系统测试 |
| perf | "performance" | 性能测试 |
| sec | "security" | 安全测试 |
| reli | "reliability" | 可靠性测试 |

---

## 7. 项目定位总结

XDevice 是 OpenHarmony 测试框架的核心基础设施：

```
┌─────────────────────────────────────────────────────────┐
│                    XDevice 测试框架                       │
├─────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ 命令行  │  │ 配置管理 │  │ 设备管理 │  │ 调度执行 │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│                                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ 报告生成 │  │ 分布式  │  │ 插件扩展 │  │ 工具包  │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │    OpenHarmony 设备测试        │
        │  • 标准系统设备                 │
        │  • 轻量系统设备                 │
        │  • 小型系统设备                 │
        └────────────────────────────────┘
```

---

## 相关文档

- [目录结构](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [模块详情](03_Modules.md)
- [配置说明](05_Configuration.md)
- [使用指南](07_Usage.md)
