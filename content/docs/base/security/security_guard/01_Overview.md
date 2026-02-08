# 项目概览

## 1. 项目定位

**SecurityGuard（设备风险管理平台，简称 SG）** 是 OpenHarmony 安全子系统的核心组件，向应用提供设备风险分析能力。

**证据来源**: `README_zh.md:10-11`

```
设备风险管理平台（SecurityGuard，简称SG）向应用提供风险分析能力，
包括root检测，设备完整性检测，物理真机检测等功能。
```

---

## 2. 核心能力

### 2.1 三大安全检测模型

| 模型 | 模型 ID | 功能描述 |
|------|---------|----------|
| 越狱检测模型 | `3001000000` | 检测系统是否处于越狱状态（分析系统调用表、内核代码段） |
| 设备完整性检测模型 | `3001000001` | 检测 boot 状态是否有异常、设备是否被解锁 |
| 物理机检测模型 | `3001000002` | 检测当前设备是物理机或模拟器 |

**证据来源**: `security_guard_napi.h:125-132`

```cpp
enum ModelIdType {
    ROOT_SCAN_MODEL_ID = 3001000000,           // JailbreakCheck
    DEVICE_COMPLETENESS_MODEL_ID = 3001000001, // IntegrityCheck
    PHYSICAL_MACHINE_DETECTION_MODEL_ID = 3001000002, // SimulatorCheck
    SECURITY_RISK_FACTOR_MODEL_ID = 3001000009, // RiskFactorCheck
    WLAN_RISK_DETECTION_MODEL_ID = 3001000011,  // WifiCheck
};
```

### 2.2 附加能力

| 能力 | 功能描述 |
|------|----------|
| 安全事件采集 | 启动/停止安全事件采集器 |
| 安全事件查询 | 按条件查询历史安全事件 |
| 安全事件上报 | 向系统上报安全事件 |
| 订阅/通知机制 | 订阅安全事件实时通知 |
| 策略更新 | 更新安全策略配置文件 |

---

## 3. 架构概览

### 3.1 三层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (JS API)                           │
│  security.securityGuard (N-API 模块)                        │
│  - getModelResult()  - querySecurityEvent()                │
│  - reportSecurityEvent() - start/stopSecurityEventCollector│
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    SDK 层 (Inner API)                       │
│  libsg_collect_sdk.so    - 数据采集 SDK                     │
│  libsg_classify_sdk.so    - 风险分类 SDK                     │
│  libsg_collector_sdk.so   - 采集器 SDK                      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   服务层 (SA Services)                       │
│  sg_collect_service (SA 3523/3524)   - 数据采集服务         │
│  sg_classify_service (SA 3523/3524)  - 风险分类服务         │
│  security_collector_service (SA 3525) - 安全采集器服务      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   安全模型层                                 │
│  - 越狱检测模型 (root detection)                            │
│  - 设备完整性模型 (device integrity)                        │
│  - 物理机检测模型 (physical machine detection)              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 目录结构

**证据来源**: `README_zh.md:26-38`

```
security_guard/
├── frameworks/          # 框架代码, 被 interfaces 和 services 使用
│   ├── common/         # 公共框架
│   │   ├── collect/    # 数据采集 SDK
│   │   ├── classify/   # 风险分类 SDK
│   │   ├── collector/  # 采集器 SDK
│   │   ├── utils/      # 工具类
│   │   └── log/        # 日志
│   └── js/             # JS 框架
│       └── napi/       # N-API 接口
├── interfaces/         # 接口 API 代码
│   └── inner_api/      # inner api 接口
├── services/           # 服务框架代码
│   ├── config_manager/  # SG 配置管理
│   ├── data_collect/    # SG 数据管理
│   ├── risk_classify/   # SG 模型管理
│   └── security_collector # SG 采集器管理
└── ...
```

---

## 4. 运行环境

### 4.1 系统要求

| 要求 | 说明 |
|------|------|
| 系统类型 | OpenHarmony Standard System |
| 系统能力 | SystemCapability.Security.SecurityGuard |
| 最低版本 | OpenHarmony 3.1+ |

**证据来源**: `bundle.json:15-17, 36-38`

```json
"syscap": [
  "SystemCapability.Security.SecurityGuard"
],
"adapted_system_type": [
  "standard"
]
```

### 4.2 权限要求

| 权限 | 用途 |
|------|------|
| `ohos.permission.COLLECT_SECURITY_EVENT` | 采集安全事件 |
| `ohos.permission.QUERY_SECURITY_EVENT` | 查询安全事件 |

**证据来源**: `sa_profile/security_guard.cfg:29-31`

```json
"permission": [
  "ohos.permission.COLLECT_SECURITY_EVENT",
  "ohos.permission.QUERY_SECURITY_EVENT"
]
```

---

## 5. 关键配置

### 5.1 Feature Flags

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `security_guard_enable` | true | 主开关 |
| `security_guard_enable_ext` | false | 扩展功能 |
| `security_guard_trim_model_analysis` | false | 精简模型分析 |
| `security_guard_enable_device_id` | false | 设备 ID 功能 |

**证据来源**: `security_guard.gni:17-20`

### 5.2 配置文件

| 配置文件 | 用途 |
|----------|------|
| `security_guard_event.json` | 事件配置 |
| `security_guard_model.cfg` | 模型配置 |
| `security_guard_event_group.json` | 事件组配置 |
| `security_audit.cfg` | 采集器配置 |

---

## 6. 组件矩阵

| 组件 | 类型 | 产物 | 说明 |
|------|------|------|------|
| `security_guard_napi` | N-API | libsecurityguard_napi.so | JS 接口绑定 |
| `sg_classify_service` | SA | libsg_classify_service.so | 风险分类服务 |
| `sg_collect_service` | SA | libsg_collect_service.so | 数据采集服务 |
| `security_collector_service` | SA | libsecurity_collector_service.so | 安全采集器服务 |
| `sg_config_manager` | Library | libsg_config_manager.so | 配置管理 |

**证据来源**: `BUILD.gn:18-75`

---

## 相关文档

### 核心文档

- [N-API 参考](./02_NAPI_Reference.md)
- [架构详解](./03_Architecture.md)
- [构建配置](./04_Build.md)
- [安全评审](./06_Security_Review.md)

### 开发指南

- [快速开始](./07_QuickStart.md)
- [调试指南](./08_Debugging.md)

### 示例代码

- [设备安全检查示例](./samples/01_device_check.md)
- [事件监控示例](./samples/02_event_monitoring.md)
- [风险控制集成示例](./samples/03_risk_app.md)

### 附录

- [术语表](./appendix/99_Glossary.md)
- [错误码速查](./appendix/99_ErrorCodeQuickRef.md)
