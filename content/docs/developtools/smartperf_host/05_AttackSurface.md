# SmartPerf 攻击面分析

> **文档目的**: 帮助安全研究员快速识别 SmartPerf 的所有外部输入入口和敏感操作

## 1. 外部输入清单

### 1.1 网络输入

#### 1.1.1 Socket IPC (device_command ↔ device_ui)

**位置**: `smartperf_device/device_command/services/ipc/src/sp_server_socket.cpp`
**端口**: 50001 (TCP), 50002 (UDP), 50003 (UDP_EX)

**输入来源**:
- device_ui (ArkTS 应用) 通过 UDP Socket 发送命令
- Host IDE 通过 TCP Socket 发送命令
- 第三方工具通过 Socket 连接

**消息格式**:
```cpp
// common.h:21-59
enum class MessageType {
    APP_START_COLLECT, APP_STOP_COLLECT, APP_PAUSE_COLLECT, APP_RESUME_COLLECT,
    GET_CPU_NUM, GET_CPU_FREQ_LOAD, SET_PKG_NAME, SET_PROCESS_ID,
    GET_FPS_AND_JITTERS, GET_GPU_FREQ, GET_GPU_LOAD, GET_DDR_FREQ,
    GET_RAM_INFO, GET_MEMORY_INFO, GET_TEMPERATURE, GET_POWER, GET_CAPTURE,
    CATCH_ONE_TRACE, CATCH_TRACE_FINISH, SET_DUBAI_DB, START_DUBAI_DB,
    CATCH_NETWORK_TRAFFIC, GET_NETWORK_TRAFFIC, BACK_TO_DESKTOP, GET_CUR_FPS,
    SET_GAME_VIEW, GET_APP_TYPE, CHECK_UDP_STATUS, GET_LOG, GET_DAEMON_VERSION,
    GET_PROCESS_THREADS, GET_PROCESS_FDS, START_GPU_COUNTER, SAVE_GPU_COUNTER,
    APP_RECEIVE_DATA_ON, APP_RECEIVE_DATA_OFF, GET_INDEX_INFO
};
```

**风险关注点**:
- Token 校验绕过（HDC Shell 模式）
- 消息类型枚举越界
- 数据包长度验证

#### 1.1.2 Worker Socket (device_ui 内部)

**位置**: `smartperf_device/device_ui/entry/src/main/ets/workers/worker.js`
**端口**: 8283 (接收), 8284 (发送)

**输入来源**:
- SP_daemon 返回的数据
- Worker 线程间通信

**命令格式**:
```javascript
set_pkgName::${pkg}    // 设置目标包名
get_fps_and_jitters    // 获取 FPS
get_ram_info::${pkg}   // 获取 RAM
catch_trace_start/end  // 控制 trace 抓取
```

**风险关注点**:
- Socket 消息解析注入
- 包名字符串未严格校验

#### 1.1.3 HDC 通信 (Host IDE ↔ Device)

**位置**: `smartperf_host/ide/src/hdc/`

**输入来源**:
- Host IDE 通过 HDC 协议与设备通信
- USB/WebUSB 数据传输

**风险关注点**:
- HDC 协议中间人攻击
- 设备认证绕过

### 1.2 文件输入

#### 1.2.1 Trace 文件解析

**位置**: `smartperf_host/trace_streamer/src/main.cpp`

**输入来源**:
- 用户加载的本地 trace 文件
- HDC 拉取的设备 trace 文件

**支持格式**:
| 格式 | 扩展名 | 解析器位置 |
|------|--------|-----------|
| ftrace 文本 | .trace | parser/ptreader_parser/ |
| Hiperf Protobuf | .htrace | parser/pbreader_parser/ |
| HiSysEvent | .hisysevent | parser/ptreader_parser/ |
| XPower | .xpower | parser/pbreader_parser/ |
| Raw Trace | .raw | parser/rawtrace_parser/ |

**风险关注点**:
- 格式混淆攻击（文件头伪造）
- Protobuf 解析长度溢出
- 压缩炸弹（zip/gzip 炸弹）

#### 1.2.2 CSV 文件操作

**位置**: `smartperf_device/device_command/services/task_mgr/src/task_manager.cpp:339-378`

**输入来源**:
- 用户指定的 CSV 导出路径
- 采集数据写入

**风险关注点**:
- 路径遍历（`../etc/passwd`）
- 文件覆盖

#### 1.2.3 系统文件读取

**位置**: 各 collector 实现

**读取路径**:
| 文件路径 | 采集器 | 风险 |
|---------|--------|------|
| `/sys/devices/system/cpu/*/cpufreq/scaling_cur_freq` | CPU.ets | 符号链接劫持 |
| `/sys/class/power_supply/Battery/*` | Power.ets | 敏感信息 |
| `/proc/stat` | CPU.cpp | 信息泄露 |
| `/sys/class/devfreq/*/cur_freq` | GPU.ets/DDR.ets | 符号链接劫持 |
| `/sys/devices/virtual/thermal/thermal_zone*/temp` | Thermal.ets | 路径遍历 |

### 1.3 命令行输入

#### 1.3.1 SP_daemon 命令行参数

**位置**: `smartperf_device/device_command/smartperf_main.cpp:71-112`

**输入来源**:
- 用户通过 shell 执行 SP_daemon
- IDE 通过 HDC 发送命令

**参数处理**:
```cpp
static bool g_checkCmdParam(std::vector<std::string> &argv, std::string &errorInfo)
// 验证命令参数是否在白名单中
```

**风险关注点**:
- 命令注入（分号、管道符）
- 参数长度溢出

### 1.4 N-API 输入

#### 1.4.1 device_ui N-API 接口

**位置**: `smartperf_device/device_ui/entry/src/main/ets/common/profiler/`

**输入来源**:
- ArkTS/JS 层调用 Native 代码
- 用户界面交互

**关键接口**:
| 接口 | 输入 | 风险 |
|------|------|------|
| ProfilerTask.taskInit() | 包名字符串 | 字符串注入 |
| ProfilerTask.taskStart() | 配置参数 | 越界访问 |
| BaseProfiler.readData() | 设备节点路径 | 路径遍历 |

### 1.5 环境变量

**位置**: `smartperf_device/device_command/smartperf_main.cpp:227-232`

**检查的环境变量**:
```cpp
// VERSION_TYPE, const.security.developermode.state
if (OHOS::system::GetParameter(VERSION_TYPE, "Unknown") != "beta") {
    if (!OHOS::system::GetBoolParameter("const.security.developermode.state", true)) {
        std::cout << "Not a development mode state" << std::endl;
        return 0;
    }
}
```

**风险关注点**:
- 环境变量伪造
- 开发模式绕过

---

## 2. 敏感操作清单

### 2.1 系统调用

| 系统调用 | 位置 | 风险等级 | 说明 |
|---------|------|---------|------|
| `popen()` | `utils/src/sp_utils.cpp:115` | 🔴 高 | 执行系统命令 |
| `popen()` | `collector/src/RAM.cpp:205` | 🔴 高 | 执行 hidumper |
| `popen()` | `collector/src/FPS.cpp:512,630` | 🔴 高 | 执行 surface dump |
| `fork()` + `execvp()` | `utils/src/startup_delay.cpp:187-203` | 🔴 高 | 启动子进程 |
| `fork()` + `execvp()` | `collector/src/FPS.cpp:297-306` | 🔴 高 | 执行 hidumper |
| `daemon()` | `cmds/src/smartperf_command.cpp:110` | 🟡 中 | 转为守护进程 |

### 2.2 文件系统操作

| 操作 | 位置 | 风险等级 | 说明 |
|------|------|---------|------|
| `chmod 777` | `utils/src/sp_utils.cpp:919` | 🔴 高 | 修改目录权限 |
| `chmod` | `utils/src/sp_log.cpp:177` | 🟡 中 | 修改日志权限 |
| `realpath()` | `services/task_mgr/src/task_manager.cpp:339-378` | 🟢 低 | 路径规范化 |
| 写 `/sys/` | 各 collector | 🟡 中 | 写入系统节点 |
| 读 `/proc/` | 各 collector | 🟢 低 | 读取进程信息 |

### 2.3 网络操作

| 操作 | 位置 | 风险等级 | 说明 |
|------|------|---------|------|
| Socket bind(50001) | `services/ipc/src/sp_server_socket.cpp` | 🟡 中 | TCP 服务端口 |
| Socket bind(50002) | `services/ipc/src/sp_server_socket.cpp` | 🟡 中 | UDP 服务端口 |
| Socket bind(8283) | Worker | 🟡 中 | device_ui 端口 |
| HDC 连接 | `ide/src/hdc/` | 🟡 中 | USB 设备通信 |

### 2.4 IPC 操作

| 操作 | 位置 | 风险等级 | 说明 |
|------|------|---------|------|
| GameServicePlugin | `interface/GameServicePlugin.h` | 🔴 高 | 游戏服务 IPC |
| GameEventCallback | `interface/GameEventCallback.h` | 🟡 中 | 游戏事件回调 |
| GpuCounterCallback | `interface/GpuCounterCallback.h` | 🟡 中 | GPU 计数器回调 |
| ServicePlugin 加载 | `utils/src/service_plugin.cpp` | 🔴 高 | 动态库加载 |

### 2.5 权限操作

| 操作 | 位置 | 风险等级 | 说明 |
|------|------|---------|------|
| `SYSTEM_FLOAT_WINDOW` | `device_ui/module.json` | 🟡 中 | 系统悬浮窗 |
| `WRITE_USER_STORAGE` | `device_ui/module.json` | 🟡 中 | 写用户存储 |
| `GET_INSTALLED_BUNDLE_LIST` | `device_ui/module.json` | 🟢 低 | 获取应用列表 |

---

## 3. 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             SmartPerf 信任边界                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                           UNTRUSTED (不可信)                           │  │
│  │  ┌────────────────────────┐  ┌──────────────────────────────────────┐ │  │
│  │  │   用户输入文件          │  │   网络数据                           │ │  │
│  │  │   - trace 文件         │  │   - HDC 通信                         │ │  │
│  │  │   - CSV 文件           │  │   - Socket 数据                      │ │  │
│  │  │   - 命令行参数         │  │   - 第三方工具                       │ │  │
│  │  └────────────────────────┘  └──────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                           输入验证 + 格式解析                                 │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         SEMI-TRUSTED (半可信)                          │  │
│  │  ┌────────────────────────┐  ┌──────────────────────────────────────┐ │  │
│  │  │   device_ui (HAP)      │  │   trace_streamer (WASM)              │ │  │
│  │  │   - ArkTS/ETS          │  │   - 浏览器环境                       │ │  │
│  │  │   - 悬浮窗组件         │  │   - SQL 查询                         │ │  │
│  │  └────────────────────────┘  └──────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                           Token 校验 + 权限检查                               │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                           TRUSTED (可信)                               │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │   SP_daemon 内部                                                 │  │  │
│  │  │   - collector/ (采集器)                                         │  │  │
│  │  │   - services/task_mgr/ (任务管理)                                │  │  │
│  │  │   - utils/ (内部工具)                                           │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │   OpenHarmony 系统服务                                          │  │  │
│  │  │   - ipc / samgr                                                 │  │  │
│  │  │   - graphic_2d / window_manager                                 │  │  │
│  │  │   - ability_base / hiview                                       │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.1 边界跨越点

| 边界 | 验证机制 | 位置 |
|------|---------|------|
| UNTRUSTED → SEMI-TRUSTED | 文件格式校验 | `trace_streamer/src/parser/` |
| SEMI-TRUSTED → TRUSTED | Token 校验 | `sp_thread_socket.cpp:111-137` |
| TRUSTED → 系统服务 | IPC 鉴权 | `interface/GameServicePlugin.h` |

---

## 4. 攻击向量汇总

### 4.1 远程攻击向量

| 向量 | 前提条件 | 影响 | 代码位置 |
|------|---------|------|---------|
| Socket 命令注入 | 获取 Token 或 HDC Shell | RCE | `sp_thread_socket.cpp` |
| HDC 中间人 | 网络访问权限 | 数据篡改 | `ide/src/hdc/` |
| Trace 文件解析漏洞 | 用户加载恶意文件 | 信息泄露/RCE | `trace_streamer/src/parser/` |

### 4.2 本地攻击向量

| 向量 | 前提条件 | 影响 | 代码位置 |
|------|---------|------|---------|
| 命令行注入 | 本地 shell 访问 | RCE | `smartperf_main.cpp` |
| 路径遍历 | 控制输出文件名 | 任意文件写 | `sp_utils.cpp:621-671` |
| 动态库加载 | 控制插件路径 | 代码执行 | `service_plugin.cpp` |
| 符号链接劫持 | 控制 /sys 节点 | 信息泄露 | 各 collector |

### 4.3 权限提升向量

| 向量 | 前提条件 | 影响 | 代码位置 |
|------|---------|------|---------|
| SP_daemon 漏洞利用 | SP_daemon 以 root 运行 | root 权限 | 整体架构 |
| 开发模式绕过 | 修改系统属性 | 启动限制绕过 | `smartperf_main.cpp:227-232` |
| 悬浮窗权限滥用 | 获取 SYSTEM_FLOAT_WINDOW | UI 劫持 | `module.json` |

---

## 5. 安全测试建议

### 5.1 模糊测试目标

| 目标 | 输入类型 | 工具建议 |
|------|---------|---------|
| `trace_streamer` 解析器 | Trace 文件格式 | AFL++, libFuzzer |
| `sp_thread_socket` | Socket 消息 | Boofuzz |
| `argument_parser` | 命令行参数 | AFL++ |
| `sp_utils` 文件操作 | 文件路径 | 手工测试 |

### 5.2 渗透测试检查清单

- [ ] Socket Token 校验绕过测试
- [ ] 命令注入测试（`; | &`）
- [ ] 路径遍历测试（`../`）
- [ ] 动态库加载测试（恶意 `.so`）
- [ ] 符号链接劫持测试
- [ ] 环境变量伪造测试
- [ ] Protobuf 解析边界测试
- [ ] 压缩炸弹测试

---

## 相关文档

- [安全风险评估](06_Security.md) - 详细漏洞分析与修复建议
- [系统架构](01_Architecture.md) - 架构与信任边界
- [项目概览](00_Overview.md) - 项目定位与能力边界

---

*文档版本: v1.0*
*最后更新: 2026-02-07*
