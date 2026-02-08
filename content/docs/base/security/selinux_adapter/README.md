# SELinux Adapter 工程 Wiki

## 文档说明

本 Wiki 提供 OpenHarmony SELinux Adapter 组件的完整工程文档，覆盖架构、API、编译、安全等关键维度。

### 覆盖范围

| 维度 | 状态 | 说明 |
|------|------|------|
| 项目概述 | ✅ | 定位、能力、依赖 |
| 架构设计 | ✅ | 组件图、数据流、线程模型 |
| Inner API | ✅ | 5 个核心库的 C 接口 |
| GN 构建 | ✅ | Targets、依赖、产物 |
| 安全评审 | ✅ | 攻击面、风险分析 |
| 故障排查 | ✅ | 常见问题与定位 |

### 文档更新

- **最后更新**: 2026-02-06
- **更新方式**: 随代码变更手动同步
- **证据来源**: 代码直接证据（路径+符号）

### 阅读路线

#### 新人学习路线
```
SUMMARY.md → 01_Overview.md → 02_Architecture.md → 03_API.md
```

#### 安全研究路线
```
SUMMARY.md → 01_Overview.md → 02_Architecture.md → 05_Security.md
```

#### 开发者路线
```
01_Overview.md → 03_API.md → 04_Build.md → 06_Troubleshooting.md
```

### 文档索引

| 场景 | 推荐文档 |
|------|---------|
| 快速了解项目 | `01_Overview.md` |
| 理解架构设计 | `02_Architecture.md` |
| 查看 API 接口 | `03_API.md` |
| 编译构建问题 | `04_Build.md` |
| 安全风险评估 | `05_Security.md` |
| 调试排查问题 | `06_Troubleshooting.md` |
| 查看项目评估 | `_work/ASSESSMENT.md` |

### 相关链接

- [OpenHarmony SELinux 开发指南](docs/en/device-dev/subsystems/subsys-security-selinux-develop-intro.md)
- [SELinux 策略编写](docs/en/device-dev/subsystems/subsys-security-selinux-sample-file.md)
- [代码仓库](https://gitee.com/openharmony/security_selinux_adapter.git)
