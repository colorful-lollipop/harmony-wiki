# Battery Manager Wiki 导航

> 本 Wiki 提供双路线导航，分别针对「新人学习者」和「安全研究员」设计。

---

## 🎯 阅读路线选择

### 🟢 新人学习路线（推荐）

**目标受众**: 首次接触 battery_manager 项目的开发者
**学习目标**: 快速理解项目定位、掌握核心 API、能够进行日常开发

**预计时间**: 2-3 小时

```
Step 1: 快速入门（15 分钟）
├─ 00_Overview（项目概览）
│   ├─ 5 分钟理解项目定位
│   ├─ 核心功能清单
│   └─ 运行环境说明
│
Step 2: 建立全局认知（45 分钟）
├─ 01_Project_Position（项目边界与核心能力）
│   ├─ 功能边界
│   ├─ 与上下游组件的关系
│   └─ 技术选型说明
│
├─ 02_Directory_Structure（目录结构与模块职责）
│   ├─ 代码组织结构
│   └─ 各模块职责划分
│
└─ 03_Architecture（系统架构说明）
    ├─ 分层架构
    ├─ 组件交互关系
    ├─ 数据流向
    └─ 线程模型
│
Step 3: API 接口学习（45 分钟）
├─ 04_NAPI_API（对外 N-API 接口）
│   ├─ JS API 清单
│   ├─ 参数说明
│   └─ 调用示例
│
├─ 05_Inner_API（内部 API 接口）
│   ├─ 模块间调用接口
│   └─ 接口稳定性划分
│
└─ 09_Troubleshooting（常见问题与定位）
    ├─ 常见问题
    ├─ 调试技巧
    └─ 问题排查方法
│
Step 4: 深入学习（可选，1 小时）
├─ 06_GN_Targets（GN 构建目标）
├─ 07_Build_Artifacts（编译产物说明）
└─ 附录文档
    ├─ appendix/Callgraphs.md（调用链）
    └─ appendix/Config_Flags.md（配置开关）
```

---

### 🔴 安全研究路线

**目标受众**: 安全研究员、安全审计工程师
**研究目标**: 快速识别攻击面、理解信任边界、挖掘潜在漏洞

**预计时间**: 3-4 小时

```
Step 1: 攻击面识别（1 小时）
├─ 05_AttackSurface（攻击面分析）⭐ 新增
│   ├─ 威胁模型
│   ├─ 外部输入入口清单
│   │   ├─ N-API 输入点（Set/GetBatteryConfig）
│   │   ├─ IPC 消息处理
│   │   ├─ 配置文件输入
│   │   └─ HDI 事件输入
│   ├─ 敏感操作清单
│   │   ├─ 文件操作
│   │   ├─ 系统服务调用
│   │   ├─ HDI 调用
│   │   └─ 权限提升点
│   └─ 信任边界图
│       ├─ 用户空间 → N-API 层
│       ├─ N-API 层 → IPC 通信
│       ├─ IPC 通信 → 服务端
│       └─ 服务端 → HDI 驱动
│
Step 2: 深度安全分析（2 小时）
└─ 08_Security_Review（安全风险评审）⭐ 重点
    ├─ 攻击面分析（可利用点清单）
    │   ├─ 配置注入攻击
    │   ├─ 配置信息泄露
    │   ├─ 权限提升风险
    │   ├─ 竞态条件风险
    │   ├─ 内存安全风险
    │   ├─ 信息泄露风险
    │   └─ 逻辑漏洞风险
    ├─ 每个风险的详细信息
    │   ├─ 位置（文件路径 + 行号）
    │   ├─ 证据（代码片段）
    │   ├─ 触发路径（输入 → 漏洞点）
    │   ├─ 影响评估（可利用性、权限提升可能）
    │   └─ 修复建议（具体代码级建议）
    └─ 利用路径可视化（Mermaid 图）
│
Step 3: 架构理解（补充，45 分钟）
├─ 03_Architecture（关注数据流和边界）
│   ├─ 数据流关键节点
│   ├─ 线程模型（并发控制点）
│   └─ 组件交互（信任边界）
│
└─ 04_NAPI_API（关注参数校验）
    ├─ 参数解析代码
    ├─ 输入验证逻辑
    └─ 错误处理机制
│
Step 4: 代码导航（补充，30 分钟）
├─ 02_Directory_Structure（快速定位代码）
└─ 05_Inner_API（内部接口调用链）
```

---

## 📋 文档快速索引

### 按主题分类

| 主题 | 文档 | 适合人群 |
|------|------|---------|
| **项目概览** | [00_Overview](00_Overview.md) | 🟢 新人 |
| **项目定位** | [01_Project_Position](01_Project_Position.md) | 🟢 新人 |
| **目录结构** | [02_Directory_Structure](02_Directory_Structure.md) | 🟢 新人 |
| **系统架构** | [03_Architecture](03_Architecture.md) | 🟢 新人 / 🔴 安全 |
| **N-API 接口** | [04_NAPI_API](04_NAPI_API.md) | 🟢 新人 / 🔴 安全 |
| **内部 API** | [05_Inner_API](05_Inner_API.md) | 🟢 新人 |
| **攻击面分析** | [05_AttackSurface](05_AttackSurface.md) | 🔴 安全 ⭐ |
| **GN 构建目标** | [06_GN_Targets](06_GN_Targets.md) | 🟢 新人 |
| **编译产物** | [07_Build_Artifacts](07_Build_Artifacts.md) | 🟢 新人 |
| **安全风险评审** | [08_Security_Review](08_Security_Review.md) | 🔴 安全 |
| **故障排查** | [09_Troubleshooting](09_Troubleshooting.md) | 🟢 新人 |

### 附录文档

| 附录 | 文档 | 说明 |
|------|------|------|
| **调用链分析** | [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链路 |
| **配置开关** | [appendix/Config_Flags.md](appendix/Config_Flags.md) | Feature 开关说明 |

---

## 🎓 学习建议

### 对于新人开发者

1. **快速入门**: 先阅读 [00_Overview](00_Overview.md) 的"5 分钟理解"章节
2. **理解架构**: 按顺序阅读 [01_Project_Position] → [02_Directory_Structure] → [03_Architecture]
3. **学习 API**: 详细阅读 [04_NAPI_API]，掌握 JS 调用方法
4. **实践调试**: 参考 [09_Troubleshooting] 进行问题排查
5. **深入构建**: 如需修改构建配置，阅读 [06_GN_Targets] 和 [07_Build_Artifacts]

### 对于安全研究员

1. **快速定位攻击面**: 先阅读 [05_AttackSurface]（新增），掌握所有外部输入入口
2. **深度分析**: 详细阅读 [08_Security_Review]，理解每个可利用点的触发路径
3. **理解边界**: 阅读 [03_Architecture] 的数据流和线程模型，识别竞态条件
4. **验证输入校验**: 阅读 [04_NAPI_API] 的参数解析代码，确认输入验证是否充分
5. **追踪调用链**: 使用附录文档 [appendix/Callgraphs.md] 追踪完整调用链路

---

## 📊 文档覆盖范围

### 已覆盖内容

- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 系统架构与数据流
- ✅ N-API 接口（含 API 清单表）
- ✅ 内部 API 接口
- ✅ GN 构建目标与依赖关系
- ✅ 编译产物说明
- ✅ 攻击面分析（新增）
- ✅ 安全风险评审（含 7 个可利用点）
- ✅ 常见问题与排查

### 未覆盖内容

- ⚠️ 测试相关代码（test/ 目录）
- ⚠️ 第三方依赖实现细节
- ⚠️ OpenHarmony 基础库内部实现

---

## 🔗 相关资源

### 官方文档

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [电源管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/电源管理子系统.md)
- [N-API 开发指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/napi)

### 相关仓库

- [powermgr_power_manager](https://gitee.com/openharmony/powermgr_power_manager)
- [powermgr_battery_statistics](https://gitee.com/openharmony/powermgr_battery_statistics)
- [powermgr_battery_lite](https://gitee.com/openharmony/powermgr_battery_lite)

### 安全规范

- [OpenHarmony 安全子系统](https://www.bookstack.cn/read/openharmony-1.0-zh-cn/readme-%E5%AE%89%E5%85%A8%E5%AD%90%E7%B3%BB%E7%BB%9F)
- [设备安全指南](https://opendeep.wiki/openharmony/docs/security)

---

## 📝 文档维护

### 代码变更后

1. 更新对应模块的 Wiki 文档
2. 更新 `wiki/_work/NOTES.md` 中的发现
3. 如有新增接口，更新 [04_NAPI_API](04_NAPI_API.md)
4. 如有架构变更，更新 [03_Architecture](03_Architecture.md)
5. 如有安全相关变更，更新 [05_AttackSurface](05_AttackSurface.md) 和 [08_Security_Review](08_Security_Review.md)

### 建议更新时机

- 每次 API 变更后
- 每次架构调整后
- 每次安全评审后
- 每次 GN 配置调整后

---

## 📄 文档约定

- 所有代码引用使用 **代码证据** 标注（文件路径 + 行号）
- 不引用测试代码作为业务证据
- 使用 `TODO(需确认)` 标注需要进一步验证的内容
- 术语统一使用 OpenHarmony 官方术语

---

**最后更新**: 2026-02-07 10:15:00
**返回**: [README](README.md)
