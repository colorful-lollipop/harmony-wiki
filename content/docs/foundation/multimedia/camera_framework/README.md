# OpenHarmony Camera Framework Wiki

> 为新人学习者和安全研究员提供的完整技术文档

**生成时间**: 2025-02-07  
**项目版本**: 3.1  
**文档版本**: v1.0  
**文档状态**: ✅ 已完成 (Phase 0-5)

---

## 📋 文档导航

**快速选择您的路线：**

### 📚 新人学习路线 (Newcomer Track)
适合刚接触项目的开发者：

1. **[项目概览](./01_Overview.md)** - 5分钟了解项目定位和能力边界
2. **[架构与数据流](./02_Architecture.md)** - 理解组件关系和调用流程
3. **[目录结构](./03_CodeMap.md)** - 快速找到代码位置
4. **[对外接口](./04_Interface.md)** - API 使用参考

### 🔒 安全研究路线 (Security Track)
适合安全审计和漏洞研究人员：

1. **[攻击面分析](./05_AttackSurface.md)** - 识别所有外部输入入口
2. **[安全风险评估](./06_SecurityReview.md)** - 详细风险分析（含代码证据）
3. **[架构与数据流](./02_Architecture.md)** - 信任边界分析

### 🔧 工程开发路线 (Developer Track)
适合进行功能开发或问题调试：

1. **[目录结构](./03_CodeMap.md)** - 代码导航指南
2. **[构建与产物](./07_Build.md)** - GN Targets 和编译配置
3. **[内部实现](./08_Internals.md)** - 核心类和设计模式
4. **[对外接口](./04_Interface.md)** - API 参考手册

---

## 📖 文档清单

| 文档 | 类型 | 内容 | 目标读者 |
|------|------|------|----------|
| [SUMMARY.md](./SUMMARY.md) | 导航 | 全站导航 + 双路线指南 | 所有人 |
| [01_Overview.md](./01_Overview.md) | 入门 | 项目定义、能力边界、快速开始 | 所有人 |
| [02_Architecture.md](./02_Architecture.md) | 架构 | 组件图、数据流、线程模型 | 所有人 |
| [03_CodeMap.md](./03_CodeMap.md) | 导航 | 目录结构、代码定位 | 开发者 |
| [04_Interface.md](./04_Interface.md) | 参考 | N-API/NDK/IPC 接口清单 | 开发者 |
| [05_AttackSurface.md](./05_AttackSurface.md) | 安全 | 攻击面识别、信任边界 | 安全研究员 |
| [06_SecurityReview.md](./06_SecurityReview.md) | 安全 | 风险分析、代码证据 | 安全研究员 |
| [07_Build.md](./07_Build.md) | 工程 | GN Targets、Feature 开关 | 开发者 |
| [08_Internals.md](./08_Internals.md) | 深度 | 核心类、设计模式、生命周期 | 深入研究者 |
| [appendix/FAQ.md](./appendix/FAQ.md) | 参考 | 常见问题解答 | 开发者 |

---

## 🎯 项目基本信息

| 属性 | 内容 |
|------|------|
| **项目名称** | @ohos/camera_framework |
| **所属子系统** | multimedia |
| **系统能力** | SystemCapability.Multimedia.Camera.Core |
| **版本** | 3.1 |
| **SAID** | 3008 (camera_service) |
| **License** | Apache License 2.0 |
| **仓库路径** | foundation/multimedia/camera_framework |

---

## ✅ 文档质量保证

### 证据完整性检查

- [x] 架构结论 → 有目录结构和代码调用链支撑
- [x] API 描述 → 有 N-API 注册点和函数定义定位
- [x] 安全风险 → 有具体代码路径 + 符号 + 行号
- [x] 构建配置 → 有 BUILD.gn 片段和 target 名称

### 受众适配检查

**新人视角**：
- [x] 能在 5 分钟内理解项目定位和用途
- [x] 能在 15 分钟内找到核心代码位置
- [x] 能在 30 分钟内理解基本架构

**安全研究员视角**：
- [x] 能快速识别所有外部输入入口
- [x] 能定位敏感操作和权限检查点
- [x] 每个风险都有可利用性评估和修复建议

---

## 📁 文档结构

```
wiki/
├── README.md              # 本文档 - 导航入口
├── SUMMARY.md             # 全站导航 + 双路线指南
├── 01_Overview.md         # 项目概览
├── 02_Architecture.md     # 架构与数据流
├── 03_CodeMap.md          # 目录结构与代码地图
├── 04_Interface.md        # 对外接口文档
├── 05_AttackSurface.md    # 攻击面分析
├── 06_SecurityReview.md   # 安全风险评估
├── 07_Build.md            # 构建与产物
├── 08_Internals.md        # 内部实现细节
├── _work/                 # 工作文件（开发维护用）
│   ├── ASSESSMENT.md      # 项目评估报告
│   ├── PLAN.md            # 任务计划
│   └── NOTES.md           # 调研笔记
└── appendix/              # 附录
    └── FAQ.md             # 常见问题
```

---

## 🔄 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-07 | v1.0 | 初始版本，完成核心文档（ASSESSMENT → 8篇核心文档 + FAQ）|

---

## 📚 相关资源

- **官方文档**: [OpenHarmony Camera 开发指南](https://docs.openharmony.cn/pages/v5.0/zh-cn/application-dev/media/camera/camera-preparation.md)
- **Gitee 仓库**: [multimedia_camera_framework](https://gitee.com/openharmony/multimedia_camera_framework)
- **TypeScript 定义**: `interfaces/kits/js/camera_napi/@ohos.multimedia.camera.d.ts`

---

## 📝 维护说明

### 更新方式

当代码发生重大变更时：

1. **API 变更** → 更新 [04_Interface.md](./04_Interface.md)
2. **模块重构** → 更新 [03_CodeMap.md](./03_CodeMap.md) 和 [08_Internals.md](./08_Internals.md)
3. **构建变更** → 更新 [07_Build.md](./07_Build.md)
4. **安全修复** → 更新 [06_SecurityReview.md](./06_SecurityReview.md)

### 贡献指南

如需补充或修正文档：

1. 确保所有结论有代码证据支撑（文件路径 + 行号）
2. 保持术语统一，使用中文 + 英文专业术语
3. 更新 SUMMARY.md 导航
4. 记录在 `_work/NOTES.md`

---

## 📄 许可证

本文档遵循 Apache License 2.0，与原项目许可证一致。

---

**最后更新**: 2025-02-07  
**文档版本**: v1.0  
**作者**: OpenHarmony Wiki Agent
