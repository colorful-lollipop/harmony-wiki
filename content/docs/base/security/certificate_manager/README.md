# Certificate Manager Wiki

> OpenHarmony 证书管理模块 - 完整技术文档

## 文档说明

本 Wiki 文档为 OpenHarmony **certificate_manager**（证书管理）模块提供全面的技术文档，面向以下受众：

- **新人学习者**：快速理解项目、掌握 API 使用、熟悉架构
- **安全研究员**：识别攻击面、评估安全风险、定位漏洞点
- **开发者**：集成证书管理功能、调试问题

### 文档范围

✅ **已覆盖**：
- 项目概览与架构
- 目录结构与代码地图
- N-API（JavaScript Native API）接口文档
- 内部 C API 参考
- IPC 服务接口
- 攻击面分析
- 安全风险评估
- 构建系统与产物
- 内部实现细节

❌ **未覆盖**：
- 测试代码（test/ 目录）
- 第三方依赖内部实现（HUKS、OpenSSL 算法细节）
- UI 层实现（Dialog UIAbility）

### 使用指南

#### 新人入门路线
1. **[项目概览](01_Overview.md)** - 了解项目定位和能力（5 分钟）
2. **[架构与数据流](02_Architecture.md)** - 理解三层架构和证书生命周期（15 分钟）
3. **[代码地图](03_CodeMap.md)** - 快速定位核心代码位置（15 分钟）
4. **[接口文档](04_Interface.md)** - 学习 API 使用方式（按需查阅）
5. **[安全研究]**（以下按需）

#### 安全研究员路线
1. **[项目概览](01_Overview.md)** - 理解信任边界（5 分钟）
2. **[攻击面分析](05_AttackSurface.md)** - 识别所有外部输入入口（10 分钟）
3. **[安全风险评估](06_SecurityReview.md)** - 分析潜在漏洞和利用路径（30 分钟）
4. **[接口文档](04_Interface.md)** - 检查输入验证机制（按需查阅）
5. **[内部实现细节](08_Internals.md)** - 深入理解权限控制（按需查阅）

---

## 项目信息

| 属性 | 值 |
|------|-----|
| **组件名** | certificate_manager |
| **子系统** | security |
| **版本** | 4.0 |
| **许可证** | Apache License 2.0 |
| **SA ID** | 3512 |
| **ROM 占用** | 5000KB |
| **RAM 占用** | 500KB |

---

## 文档目录

```
wiki/
├── README.md                      # 本文档
├── SUMMARY.md                     # 全站导航 + 双路线推荐
├── 01_Overview.md                 # 项目概览（P0）
├── 02_Architecture.md             # 架构与数据流（P0）
├── 03_CodeMap.md                 # 目录结构与代码地图（P0）
├── 04_Interface.md               # 对外接口文档（P0）
├── 05_AttackSurface.md          # 攻击面分析（P0）
├── 06_SecurityReview.md          # 安全风险评估（P0）
├── 07_Build.md                   # 构建与产物（P1）
├── 08_Internals.md               # 内部实现细节（P1）
└── _work/
    ├── ASSESSMENT.md             # 项目评估结果
    ├── NOTES.md                  # 代码证据汇总
    └── PLAN.md                  # 任务进度追踪
```

---

## 更新日志

| 日期 | 版本 | 更新内容 | 作者 |
|------|------|---------|------|
| 2026-02-07 | 1.0.0 | 初始版本，创建完整 Wiki 框架 | AI Wiki Generator |

---

## 贡献指南

### 如何更新文档

1. **触发条件**：
   - 新增 N-API 接口
   - 修改内部 API 签名
   - 调整构建配置或产物
   - 修复安全漏洞或变更安全策略

2. **更新流程**：
   - 修改相关代码后，检查对应的 Wiki 文档
   - 更新文件路径、函数名、常量等证据引用
   - 运行测试验证文档准确性
   - 在文档末尾添加更新日志

3. **验证清单**：
   - [ ] 代码路径与文档描述一致
   - [ ] API 参数/返回值与实际代码匹配
   - [ ] GN targets 与构建产物对应正确
   - [ ] 安全风险点已更新或标注已修复

### 质量标准

**证据优先原则**：
- 每个技术结论必须有代码证据支撑
- 证据格式：文件路径:行号 或 函数签名
- 无法确认处必须标注 `TODO(证据不足)`

**受众适配**：
- 新人能在 5 分钟内理解项目定位和用途
- 新人能在 15 分钟内找到核心代码位置
- 新人能在 30 分钟内理解基本架构
- 安全研究员能快速识别所有外部输入入口
- 安全研究员能定位敏感操作和权限检查点
- 每个风险都有可利用性评估和修复建议

---

## 相关资源

### 官方文档
- **OpenHarmony 文档**: http://docs.openharmony.cn/
- **Gitee 仓库**: https://gitee.com/openharmony/security_certificate_manager
- **Device Certificate Kit**: @kit.DeviceCertificateKit
- **安全子系统**: security_huks, security_asset

### 依赖项目
- **HUKS**: https://gitee.com/openharmony/security_huks
- **OpenSSL**: 加密库依赖

---

*本文档基于代码仓库：base/security/certificate_manager*
*最后更新：2026-02-07*
