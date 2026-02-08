# SecurityGuard Wiki 文档

**项目**: OpenHarmony SecurityGuard (设备风险管理平台)  
**版本**: 3.1.0  
**License**: Apache-2.0  
**最后更新**: 2026-02-06  
**文档覆盖**: 代码可验证的 N-API、架构、构建、安全分析

---

## 文档覆盖范围

| 分类 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ 完成 | 核心能力、目录结构、运行环境 |
| N-API 接口 | ✅ 完成 | 8 个 JS API 详细文档 |
| 内部架构 | ✅ 完成 | 模块职责、依赖关系、线程模型 |
| GN 构建 | ✅ 完成 | targets、依赖、编译产物 |
| 安全风险 | ✅ 完成 | 攻击面、风险点分析 |
| SA 服务 | ✅ 完成 | 服务接口、IPC 通信、权限管理 |

---

## 快速开始

### 新人阅读顺序（推荐）

**路线 1：应用开发者（使用 JS API）**
```
README.md → 01_Overview.md → 02_NAPI_Reference.md
```

**路线 2：系统集成（构建与部署）**
```
01_Overview.md → 04_Build.md → 03_Architecture.md → 05_Services.md
```

**路线 3：安全评审**
```
01_Overview.md → 06_Security_Review.md → 02_NAPI_Reference.md
```

**完整技术深入**
```
README.md → 01_Overview.md → 02_NAPI_Reference.md → 03_Architecture.md → 
04_Build.md → 05_Services.md → 06_Security_Review.md
```

---

## 核心能力

SecurityGuard 提供三大安全检测能力：

| 能力 | 模型 ID | 功能描述 |
|------|---------|----------|
| 越狱检测 | 3001000000 | 检测系统是否处于越狱状态 |
| 设备完整性检测 | 3001000001 | 检测 boot 状态、设备是否被解锁 |
| 物理机检测 | 3001000002 | 区分物理机与模拟器 |

---

## 目录结构

```
security_guard/
├── frameworks/          # 框架代码，被 interfaces 和 services 使用
│   ├── common/         # 公共框架
│   │   ├── collect/    # 数据采集 SDK
│   │   ├── classify/   # 风险分类 SDK
│   │   ├── collector/  # 采集器 SDK
│   │   ├── constants/  # 常量定义
│   │   ├── utils/      # 工具类
│   │   ├── json/       # JSON 处理
│   │   └── log/        # 日志
│   └── js/             # JS 框架
│       └── napi/       # N-API 接口实现
├── interfaces/         # 接口 API 代码
│   └── inner_api/      # Inner API（SDK）
├── services/           # 服务框架代码
│   ├── config_manager/  # 配置管理
│   ├── data_collect/    # 数据采集服务
│   ├── risk_classify/   # 风险分类服务
│   ├── security_collector/ # 安全采集器
│   ├── collector_manager/ # 采集器管理
│   └── bigdata/        # 大数据处理
├── resource/           # 资源文件
├── sa_profile/         # SA 配置文件
├── oem_property/       # 厂商配置
└── wiki/              # 本文档目录
```

---

## 系统架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              应用层（JS/ArkUI）                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           N-API 层（securityguard_napi.z.so）                 │
│   事件上报 | 事件查询 | 事件订阅 | 采集器控制 | 模型获取 | 策略更新           │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Inner API 层（平台 SDK）                             │
│   libsg_collect_sdk.so | libsg_classify_sdk.so | libsg_collector_sdk.so     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            服务层（SA Services）                               │
│   SA 3523 RiskAnalysisManager | SA 3524 DataCollectManager | SA 3525 SecurityCollectorManager │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            数据存储层                                         │
│   SQLite | 文件系统 | RDB                                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 核心文档

| 文档 | 说明 | 上次更新 |
|------|------|----------|
| [SUMMARY.md](./SUMMARY.md) | 全站导航与阅读路线 | 2026-02-06 |
| [01_Overview.md](./01_Overview.md) | 项目定位、核心能力、架构图 | 2026-02-06 |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API 参考（8 个 API） | 2026-02-06 |
| [03_Architecture.md](./03_Architecture.md) | 模块职责、线程模型、数据流 | 2026-02-06 |
| [04_Build.md](./04_Build.md) | GN 构建配置、targets 列表、编译产物 | 2026-02-06 |
| [05_Services.md](./05_Services.md) | SA 服务架构、IPC 接口、权限管理 | 2026-02-06 |
| [06_Security_Review.md](./06_Security_Review.md) | 安全风险评审、攻击面分析 | 2026-02-06 |
| [07_QuickStart.md](./07_QuickStart.md) | 5 分钟快速开始指南 | 2026-02-06 |
| [08_Debugging.md](./08_Debugging.md) | 调试指南、日志分析、问题排查 | 2026-02-06 |

---

## 如何更新本文档

### 文档更新规则

1. **代码变更时**：同步更新相关 API 文档
2. **新增 API**：在 `02_NAPI_Reference.md` 添加条目
3. **修改构建**：更新 `04_Build.md` 的 targets 列表
4. **安全修复**：在 `06_Security_Review.md` 添加风险记录

### 验证要求

所有关键结论必须包含代码证据：

```
证据格式:
- 文件路径: line
- 关键符号名
- 必要代码片段
```

### 证据标准

- **N-API 文档**：必须包含注册位置、API 清单表、参数校验、错误码
- **GN 文档**：必须包含 targets 列表、依赖关系、产物映射
- **安全文档**：必须包含攻击面清单、风险点、修复建议

---

## 相关链接

### 核心文档

- [OpenHarmony SecurityGuard 仓库](https://gitee.com/openharmony/security_security_guard)
- [项目概览](./01_Overview.md)
- [API 文档](./02_NAPI_Reference.md)
- [架构详解](./03_Architecture.md)
- [构建配置](./04_Build.md)
- [服务详解](./05_Services.md)
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
