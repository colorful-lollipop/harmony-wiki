# XDevice 配置参数参考

## 1. 命令行参数

### 1.1 run 命令参数

| 参数 | 完整形式 | 类型 | 说明 |
|------|---------|------|------|
| `-l` | `--testlist` | string | 测试模块列表，分号分隔 |
| `-tc` | `--testcase` | string | 特定测试用例 |
| `-tf` | `--testfile` | string | 测试文件路径 |
| `-c` | `--config` | string | 配置文件路径 |
| `-env` | `--environment` | string | XML 环境配置 |
| `-sn` | `--device_sn` | string | 设备序列号 |
| `-rp` | `--report_path` | string | 报告生成路径 |
| `-respath` | `--resource_path` | string | 资源路径 |
| `-tcpath` | `--testcases_path` | string | 测试用例路径 |
| `-ta` | `--testargs` | string | 测试参数 |
| `-pt` | `--pass_through` | flag | 透传参数 |
| `-e` | `--exectype` | string | 执行类型 |
| `-t` | `--testtype` | string | 测试类型 |
| `-td` | `--testdriver` | string | 测试驱动 |
| `-tl` | `--testlevel` | string | 测试级别 |
| `-bv` | `--build_variant` | string | 构建变体 |
| `-cov` | `--coverage` | string | 覆盖率 |
| `--retry` | - | string | 重试会话 ID |
| `--session` | - | string | 会话 ID |
| `--dryrun` | - | flag | 干运行 |
| `--reboot-per-module` | - | flag | 模块执行前重启 |
| `--check-device` | - | flag | 检查设备 |
| `--repeat` | - | int | 重复次数 |

---

## 2. 配置常量

### 2.1 测试类型 (TestType)

| 常量 | 值 | 说明 |
|------|-----|------|
| `TestType.unittest` | "unittest" | 单元测试 |
| `TestType.mst` | "moduletest" | 模块测试 |
| `TestType.systemtest` | "systemtest" | 系统测试 |
| `TestType.perf` | "performance" | 性能测试 |
| `TestType.sec` | "security" | 安全测试 |
| `TestType.reli` | "reliability" | 可靠性测试 |
| `TestType.dst` | "distributedtest" | 分布式测试 |

### 2.2 设备类型 (DeviceLabelType)

| 常量 | 值 | 说明 |
|------|-----|------|
| `DeviceLabelType.wifiiot` | "wifiiot" | WiFi IoT 设备 |
| `DeviceLabelType.ipcamera` | "ipcamera" | IP 摄像头 |
| `DeviceLabelType.watch_gt` | "watchGT" | 智能手表 |
| `DeviceLabelType.phone` | "phone" | 手机设备 |

### 2.3 设备测试类型 (DeviceTestType)

| 常量 | 值 | 说明 |
|------|-----|------|
| `DeviceTestType.cpp_test` | "CppTest" | C++ 测试 |
| `DeviceTestType.jsunit_test` | "JSUnitTest" | JS 单元测试 |
| `DeviceTestType.hap_test` | "HapTest" | Hap 测试 |
| `DeviceTestType.junit_test` | "JUnitTest" | JUnit 测试 |
| `DeviceTestType.oh_jsunit_test` | "OHJSUnitTest" | OH JS 测试 |
| `DeviceTestType.oh_kernel_test` | "OHKernelTest" | 内核测试 |

### 2.4 设备状态 (DeviceState)

| 常量 | 值 | 说明 |
|------|-----|------|
| `DeviceState.BOOTLOADER` | "bootloader" | 引导模式 |
| `DeviceState.OFFLINE` | "offline" | 离线 |
| `DeviceState.ONLINE` | "device" | 在线 |
| `DeviceState.CONNECTED` | "connected" | 已连接 |
| `DeviceState.RECOVERY` | "recovery" | 恢复模式 |
| `DeviceState.UNAUTHORIZED` | "Unauthorized" | 未授权 |

### 2.5 日志级别 (LogLevel)

| 值 | 说明 |
|---|------|
| `DEBUG` | 调试信息 |
| `INFO` | 一般信息 |
| `WARNING` | 警告 |
| `ERROR` | 错误 |

---

## 3. 环境变量

| 环境变量 | 说明 |
|---------|------|
| `XDEVICE_CONFIG` | 配置文件路径 |
| `XDEVICE_HOME` | XDevice 安装目录 |

---

## 4. 配置键名 (ConfigConst)

### 4.1 命令参数键

| 键 | 说明 |
|---|------|
| `ConfigConst.action` | 操作 |
| `ConfigConst.task` | 任务 |
| `ConfigConst.testlist` | 测试列表 |
| `ConfigConst.testcase` | 测试用例 |
| `ConfigConst.device_sn` | 设备 SN |
| `ConfigConst.report_path` | 报告路径 |
| `ConfigConst.resource_path` | 资源路径 |
| `ConfigConst.testcases_path` | 用例路径 |

### 4.2 设备日志标签

| 键 | 说明 |
|---|------|
| `ConfigConst.tag_dir` | 日志目录 |
| `ConfigConst.tag_enable` | 日志开关 |
| `ConfigConst.tag_clear` | 清理日志 |
| `ConfigConst.tag_loglevel` | 日志级别 |
| `ConfigConst.tag_suite_case_log` | 套件用例日志 |

---

## 相关文档

- [配置说明](../05_Configuration.md)
- [使用指南](../07_Usage.md)
