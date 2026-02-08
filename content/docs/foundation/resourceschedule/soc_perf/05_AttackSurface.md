# 05_AttackSurface - 攻击面分析

本文档从安全研究员视角分析 SOC 统一调频部件的攻击面，包括外部输入入口、敏感操作和信任边界。

## 外部输入清单

### 1. IPC 接口参数

所有 IPC 接口均接收外部输入，是主要攻击面。

| 接口 | 输入参数 | 信任级别 | 验证逻辑 |
|------|----------|----------|----------|
| `PerfRequest` | `cmdId`, `msg` | 低 | `cmdId` 范围检查（TODO） |
| `PerfRequestEx` | `cmdId`, `onOffTag`, `msg` | 低 | 字符串长度限制 |
| `PowerLimitBoost` | `onOffTag`, `msg` | 低 | 无类型验证 |
| `ThermalLimitBoost` | `onOffTag`, `msg` | 低 | 无类型验证 |
| `LimitRequest` | `clientId`, `tags`, `configs`, `msg` | 低 | 数组越界风险 |
| `SetRequestStatus` | `status`, `msg` | 低 | 无布尔验证 |
| `SetThermalLevel` | `level` | 低 | 无范围检查 |
| `RequestDeviceMode` | `mode`, `status` | 低 | `mode.length() <= 64` |
| `RequestCmdIdCount` | `msg` | 低 | 无验证 |

**证据来源**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp`

### 2. 配置文件

| 文件 | 输入类型 | 解析位置 | 风险等级 |
|------|----------|----------|----------|
| `socperf_resource_config.xml` | XML 文件 | `socperf_config.cpp:141-181` | 中 |
| `socperf_boost_config.xml` | XML 文件 | `socperf_config.cpp:214-220` | 中 |

**潜在风险**：

- XML 实体注入（XXE）
- 路径遍历（`../` 注入）
- 恶意配置项注入
- 整数溢出（resId、cmdId）

**证据来源**：`services/core/src/socperf_config.cpp`

### 3. 服务死亡回调

| 输入类型 | 处理位置 | 风险等级 |
|----------|----------|----------|
| 服务端异常死亡 | `socperf_client.cpp:80-97` | 低 |

**证据来源**：`interfaces/inner_api/socperf_client/src/socperf_client.cpp:80-97`

```cpp
void SocPerfDeathRecipient::OnRemoteDied(const wptr<IRemoteObject> &object)
{
    std::lock_guard<std::mutex> lock(socPerfClient_.mutex_);
    socPerfClient_.client_ = nullptr;
    socPerfClient_.recipient_ = nullptr;
}
```

## 敏感操作清单

### 1. 系统调用

| 操作 | 文件:行号 | 风险等级 |
|------|-----------|----------|
| 写 CPU 频率节点 | `socperf_thread_wrap.cpp:156-178` | 高 |
| 写 GPU 频率节点 | `socperf_thread_wrap.cpp:156-178` | 高 |
| 写 DDR 频率节点 | `socperf_thread_wrap.cpp:156-178` | 高 |
| 写内核 governor | `socperf_thread_wrap.cpp:156-178` | 高 |

**触发路径**：

```
IPC 接口 → SocPerf::DoFreqActions() → SocPerfThreadWrap::Execute() → write()/ioctl()
```

### 2. 服务间通信

| 操作 | 目标服务 | 风险等级 |
|------|----------|----------|
| 上报性能事件 | 资源调度框架 | 中 |
| 权限验证 | AccessToken 服务 | 中 |

### 3. 调试接口

| 接口 | 权限要求 | 风险等级 |
|------|----------|----------|
| `Dump(fd, args)` | `ohos.permission.DUMP` + ENG_MODE | 低 |

**证据来源**：`services/server/src/socperf_server.cpp:76`

## 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           不可信域（Untrusted）                              │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  HAP 应用 (用户空间)                                                    │ │
│  │  ├── IPC 参数 (cmdId, msg, tags, configs)                            │ │
│  │  ├── XML 配置文件 (socperf_boost_config.xml)                          │ │
│  │  └── 网络输入 (N/A - 本模块不支持)                                     │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 信任边界 1: IPC 权限检查
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                          半可信域（Semi-Trusted）                            │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  SocPerfServer (SA 主线程)                                            │ │
│  │  ├── HasPerfPermission() → VerifyAccessToken()                       │ │
│  │  ├── permissionCache_ (LRU 缓存)                                     │ │
│  │  └── IPC 参数解析                                                      │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 信任边界 2: 配置验证
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                           可信域（Trusted）                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  SocPerf (核心逻辑)                                                    │ │
│  │  ├── 配置解析 (SocPerfConfig)                                         │ │
│  │  ├── 仲裁处理                                                          │ │
│  │  └── FFRT 线程池                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Linux Kernel (内核空间)                                               │ │
│  │  ├── cpufreq 驱动                                                      │ │
│  │  └── 设备节点 (/sys/devices/...)                                      │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 跨域数据流

### 1. 应用 → soc_perf

```
HAP 应用
    ↓ IPC 参数 (cmdId, msg)
IPCSkeleton::GetCallingTokenID()
    ↓
HasPerfPermission()
    ↓ 验证失败 → 返回错误
    ↓ 验证成功 → 继续处理
SocPerf::PerfRequest()
```

### 2. 配置加载

```
配置文件 (XML)
    ↓ 文件读取
xmlReadFile()
    ↓
ParseBoostXmlFile()
    ↓
LoadCmdInfo()
    ↓
内存数据结构 (Actions, Action)
```

### 3. 调频生效

```
SocPerf::DoFreqActions()
    ↓
SocPerfThreadWrap::Execute()
    ↓
write()/ioctl() → /sys/devices/system/cpu/cpu*/cpufreq/*
```

## 攻击面评估总结

| 攻击面 | 可控程度 | 影响范围 | 风险等级 |
|--------|----------|----------|----------|
| IPC cmdId 参数 | 高 | 任意提频场景 | 中 |
| IPC msg 字符串 | 高 | 日志信息泄露 | 低 |
| XML 配置解析 | 中（需系统权限） | 全局调频策略 | 中 |
| 设备节点写入 | 高（通过配置） | CPU/GPU 频率 | 高 |
| 服务死亡回调 | 低 | 服务可用性 | 低 |

## 建议缓解措施

### 1. 输入验证增强

| 输入点 | 当前状态 | 建议 |
|--------|----------|------|
| `cmdId` | 无范围检查 | 添加 1000-65535 范围限制 |
| `level` | 无范围检查 | 添加 0-10 范围限制 |
| `mode` | 长度限制 | 保留，添加字符白名单 |
| `tags`/`configs` | 无验证 | 添加数组长度和值范围检查 |

### 2. 配置安全

| 风险 | 建议 |
|------|------|
| XXE 攻击 | 禁用 XML 外部实体解析 |
| 路径遍历 | 验证 `path` 参数不以 `../` 开头 |
| 整数溢出 | 使用 `uint32_t` 替代 `int32_t` |

### 3. 运行时保护

| 风险 | 建议 |
|------|------|
| 竞态条件 | FFRT 任务队列添加互斥保护 |
| 缓存投毒 | permissionCache_ 添加 TTL |

---

## 相关文档

- 架构设计：[02_Architecture](02_Architecture.md)
- 安全风险评估：[06_SecurityReview](06_SecurityReview.md)
- 接口文档：[04_Interface](04_Interface.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
