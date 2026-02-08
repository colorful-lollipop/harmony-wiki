# DSoftBus Wiki 导航

## 快速入口

- **[首页](../README.md)** - 项目主 README
- **[代码仓库](https://gitee.com/openharmony/communication_dsoftbus)**

## 文档目录

### 入门指南

1. **[01_Overview.md](./01_Overview.md)** - 项目概览
   - 项目定位与核心能力
   - 运行环境与依赖
   - 关键概念说明

2. **[02_Architecture.md](./02_Architecture.md)** - 架构说明
   - 整体架构图
   - 模块职责划分
   - 数据流与线程模型

### 接口文档

3. **[03_NAPI.md](./03_NAPI.md)** - N-API 接口
   - BR Proxy 模块
   - Link Enhance 模块
   - API 清单与调用链

4. **[04_InnerAPI.md](./04_InnerAPI.md)** - 内部 API
   - 核心模块接口
   - 模块依赖关系
   - 稳定性标注

### 工程配置

5. **[05_Build.md](./05_Build.md)** - 编译配置
   - GN Targets 清单
   - 编译产物说明
   - Feature 开关

### 安全与运维

6. **[06_Security.md](./06_Security.md)** - 安全风险评审
   - 攻击面分析
   - 信任边界
   - 风险与修复建议

7. **[07_Troubleshooting.md](./07_Troubleshooting.md)** - 问题定位
   - 常见构建问题
   - 运行调试指南
   - 日志与 trace

## 附录

- **[appendix/Callgraphs.md](./appendix/Callgraphs.md)** - 关键调用链
  - 入口 → 核心逻辑
  - JS → N-API → 核心模块

## 阅读路线图

```
新人入门:
    README → 01_Overview → 02_Architecture → 03_NAPI → 07_Troubleshooting

应用开发:
    01_Overview → 03_NAPI → 07_Troubleshooting

系统集成:
    01_Overview → 02_Architecture → 04_InnerAPI → 05_Build

安全审计:
    06_Security (独立阅读)

贡献代码:
    全部文档
```

## 术语表

| 术语 | 说明 |
|------|------|
| BR | Basic Rate, 蓝牙基础速率 |
| BLE | Bluetooth Low Energy, 低功耗蓝牙 |
| LNN | Local Name Negotiation, 本地名称协商 |
| N-API | Native API, 原生 API 绑定 |
| SA | System Ability, 系统能力 |
| IPC | Inter-Process Communication, 进程间通信 |

## 版本兼容性

| 文档版本 | OpenHarmony 版本 |
|----------|------------------|
| v1.0 | 4.0 / 5.0 |

---

*Generated: 2026-02-06*
