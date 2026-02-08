# Wiki 维护说明

**最后更新**: 2026-02-07

---

## 文档结构（标准化后）

| 文件 | 主题 | 状态 | 受众 |
|------|------|------|--------|
| README.md | 项目概述与导航 | ✅ | 新人 |
| SUMMARY.md | 全站导航 + 双路线 | ✅ | 新人/安全研究员 |
| 01_Overview.md | 项目概览 | ✅ | 新人 |
| 02_Architecture.md | 架构设计 | ✅ | 新人/安全研究员 |
| 03_CodeMap.md | 目录结构与代码地图 | ✅ | 新人 |
| 04_Interface.md | 对外接口文档 | ✅ | 新人/安全研究员 |
| 05_AttackSurface.md | 攻击面分析 | ✅ | 安全研究员 |
| 06_SecurityReview.md | 安全风险评估 | ✅ | 安全研究员 |
| 07_Build.md | 构建与产物 | ✅ | 工程师 |
| 08_Internals.md | 内部实现细节 | ✅ | 开发者 |

## 文件重命名记录（2026-02-07）

| 旧文件名 | 新文件名 | 原因 |
|----------|----------|------|
| 03_NAPI_Reference.md | 04_Interface.md | 更符合标准的"对外接口"命名 |
| 04_Inner_API.md | 08_Internals.md | 更符合标准的"内部实现"命名 |
| 05_Build_System.md | 07_Build.md | 更符合标准的"构建"命名 |

**说明**: 重命名是为了匹配标准的文档结构，不影响内容质量。

---

## 更新规范

当代码变更时，需同步更新对应 Wiki 章节：

### 1. 接口变更
**触发条件**: 新增、修改、删除 API

**需要更新的文档**:
- [ ] `04_Interface.md` - 更新 API 清单表
- [ ] `05_AttackSurface.md` - 更新外部输入清单
- [ ] `06_SecurityReview.md` - 更新风险分析

**更新步骤**:
1. 在 `NOTES.md` 中记录变更的 API
2. 更新 `04_Interface.md` 的对应章节
3. 如果涉及安全风险，同步更新 `05_AttackSurface.md`
4. 运行证据完整性检查（每个技术点都有代码证据）

### 2. 模块增删
**触发条件**: 新增、删除、重构模块

**需要更新的文档**:
- [ ] `02_Architecture.md` - 更新模块列表与依赖关系
- [ ] `03_CodeMap.md` - 更新目录结构与文件定位
- [ ] `07_Build.md` - 更新 GN targets

**更新步骤**:
1. 更新 `ASSESSMENT.md` 的项目类型判定
2. 更新 `02_Architecture.md` 的组件图
3. 更新 `03_CodeMap.md` 的目录树
4. 更新 `07_Build.md` 的构建目标清单

### 3. 构建配置变更
**触发条件**: GN 配置、编译产物、依赖修改

**需要更新的文档**:
- [ ] `07_Build.md` - 更新 GN targets 和依赖
- [ ] `06_SecurityReview.md` - 更新依赖安全风险

**更新步骤**:
1. 更新 `NOTES.md` 的构建目标清单
2. 更新 `07_Build.md` 的构建配置片段
3. 如果涉及新依赖，更新 `06_SecurityReview.md`

### 4. 安全相关
**触发条件**: 安全漏洞修复、输入验证增强、FFI 边界加固

**需要更新的文档**:
- [ ] `05_AttackSurface.md` - 更新攻击面
- [ ] `06_SecurityReview.md` - 更新风险评估

**更新步骤**:
1. 在 `NOTES.md` 中记录安全变更
2. 更新 `05_AttackSurface.md` 的对应章节
3. 深度更新 `06_SecurityReview.md` 的风险分析和修复建议
4. 更新 `04_Interface.md` 的异常说明

---

## 质量标准

### 证据完整性
- [ ] 每个技术结论都有代码证据支持
- [ ] 所有关键 API 都有行号引用
- [ ] 所有错误码都有对应位置

### 链接有效性
- [ ] SUMMARY.md 所有链接可访问
- [ ] 文档间交叉引用有效
- [ ] 外部链接（如官方文档）可访问

### 受众适配
- [ ] 新人学习路线完整（Overview → Interface → Architecture）
- [ ] 安全研究路线完整（AttackSurface → SecurityReview）
- [ ] API 文档适合快速参考

### 术语一致性
- [ ] "API" vs "接口" - 统一使用"API"
- [ ] "FFI" vs "Foreign Function Interface" - 统一使用"FFI"
- [ ] "日历" vs "Calendar" - 代码用英文，中文解释可混合

---

## 贡献指南

### 添加新 API 文档

1. 在对应模块的 `.cj` 文件中找到 API 定义
2. 记录到 `NOTES.md` 的"详细 API 清单"章节
3. 更新 `04_Interface.md` 的对应 API 表
4. 如果是路径/资源 ID 输入，更新 `05_AttackSurface.md`

### 添加安全风险

1. 识别新的输入验证缺陷
2. 分析潜在的攻击向量
3. 更新 `05_AttackSurface.md` 的对应章节
4. 在 `06_SecurityReview.md` 深度分析风险

### 添加架构说明

1. 识别新的模块或依赖关系
2. 更新 `02_Architecture.md` 的组件图
3. 更新 `03_CodeMap.md` 的目录结构

---

## 版本控制

- **当前版本**: v1.0
- **最后更新**: 2026-02-07
- **更新人**: Sisyphus AI Agent

---

**相关文档**:
- [ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估
- [NOTES.md](_work/NOTES.md) - 代码证据库
- [PLAN.md](_work/PLAN.md) - 任务进度追踪
