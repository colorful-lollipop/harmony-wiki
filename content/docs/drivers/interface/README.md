# OpenHarmony drivers_interface Wiki

**文档版本**: 2.0  
**最后更新**: 2026-02-07  
**分析范围**: 533 IDL 文件 / 47 模块 / 6 安全敏感模块

---

## 文档概述

本文档是 OpenHarmony **drivers_interface** 仓库的完整工程 Wiki，提供硬件设备接口（HDI）定义的技术参考和安全分析。

### 仓库统计

| 指标 | 数值 |
|-----|------|
| 硬件模块 | 47+ 个 |
| IDL 文件 | 533 个 |
| 回调接口 | 126 个 |
| 构建配置 | 97+ BUILD.gn |
| 安全风险 | 10+ 条详细分析 |

---

## 文档导航

### 🎓 新人学习路线
适合刚接触 OpenHarmony 驱动开发的工程师：

1. **[项目概览](./01_Overview.md)** - 5分钟了解仓库定位
2. **[目录结构与模块地图](./02_Directory_Structure.md)** - 15分钟找到目标模块  
3. **[HDI 接口定义规范](./03_HDI_IDL_Specification.md)** - 30分钟掌握 IDL 语法
4. **[构建系统与编译产物](./04_Build_System.md)** - 了解 GN 构建
5. **[内部架构与实现原理](./06_Inner_Architecture.md)** - 深入理解机制

### 🔒 安全研究路线
适合进行安全审计和漏洞挖掘的研究员：

1. **[攻击面分析](./05_AttackSurface.md)** - 快速识别所有 IPC 入口和攻击面
2. **[安全机制与风险评估](./07_Security.md)** - 深度安全风险分析
3. **[构建系统安全配置](./04_Build_System.md)** - 编译选项安全分析

**快速索引**: [SUMMARY.md](./SUMMARY.md) 提供完整的双路线导航

---

## 文档列表

### 核心文档

| 文档 | 说明 | 目标读者 |
|-----|------|---------|
| [01_Overview](./01_Overview.md) | 项目定位、核心能力、运行环境 | 新人 |
| [02_Directory_Structure](./02_Directory_Structure.md) | 模块分类与职责说明 | 开发者 |
| [03_HDI_IDL_Specification](./03_HDI_IDL_Specification.md) | IDL 语法、版本管理、接口定义 | 开发者 |
| [04_Build_System](./04_Build_System.md) | GN 构建配置、模板参数、产物清单 | 开发者 |
| [05_AttackSurface](./05_AttackSurface.md) | 完整攻击面清单与数据流分析 | 安全研究员 |
| [06_Inner_Architecture](./06_Inner_Architecture.md) | 模块依赖、线程模型、生命周期 | 架构师 |
| [07_Security](./07_Security.md) | 安全架构、风险评估、修复建议 | 安全工程师 |

### 附录文档

| 文档 | 说明 |
|-----|------|
| [appendix/Callgraphs](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags](./appendix/Config_Flags.md) | 关键宏与 feature flags |

### 工作文档

| 文档 | 说明 |
|-----|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | Phase 0 项目评估结果 |
| [_work/NOTES.md](./_work/NOTES.md) | 代码证据汇总库 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度跟踪 |

---

## 关键安全发现

### 🔴 严重风险 (P0)

| 风险 | 位置 | 影响 |
|-----|------|------|
| HUKS Passthrough 模式 | `huks/v1_1/BUILD.gn:25` | 密钥完全暴露 |
| 明文密钥导入 | `huks/v1_1/IHuks.idl:45` | 密钥泄露 |
| Root Secret 返回 | `user_auth/v4_0/UserAuthTypes.idl:52` | 文件可被解密 |
| APDU 任意传输 | `secure_element/v1_0/ISecureElementInterface.idl:35` | SE 被绕过 |

### 🟡 高危风险 (P1)

| 风险 | 位置 | 影响 |
|-----|------|------|
| 生物认证缺少 PAC | `user_auth/v4_1/BUILD.gn` | 代码执行 |
| PIN 明文传输 | `pin_auth/v3_0/IAllInOneExecutor.idl:31` | PIN 泄露 |

**详见**: [07_Security.md](./07_Security.md) 包含完整的风险分析、触发路径和修复建议

---

## 文档更新方式

本 Wiki 随代码仓库同步维护。当以下文件发生变更时，需同步更新对应文档：

- HDI 接口定义 (`.idl` 文件)
- 构建配置 (`.gni/.gn` 文件)
- 安全配置 (`bundle.json`)

### 更新检查清单

- [ ] 新增模块是否添加到模块列表
- [ ] 接口变更是否更新 API 参考
- [ ] 构建配置变更是否更新编译产物文档
- [ ] 安全相关变更是否更新安全文档
- [ ] 代码证据路径和行号是否准确

---

## 相关链接

- **本仓库**: https://gitee.com/openharmony/drivers_interface
- **驱动实现**: https://gitee.com/openharmony/drivers_peripheral
- **HDF 框架**: https://gitee.com/openharmony/drivers_framework
- **HDF 适配器**: https://gitee.com/openharmony/drivers_adapter
- **驱动子系统文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/驱动子系统.md

---

## 贡献指南

1. 使用 Markdown 格式编写文档
2. **关键结论必须附带代码证据**（文件路径 + 行号）
3. 术语使用保持一致性
4. 避免引用测试代码作为业务证据
5. 安全风险分析必须包含：位置、证据、触发路径、影响评估、修复建议

---

*最后更新: 2026-02-07*  
*分析范围: 533 IDL 文件 / 47 模块 / 6 安全敏感模块*
