# Curl - OpenHarmony 集成文档

> **OpenHarmony 第三方库 Wiki 文档**
>
> curl 8.8.0 在 OpenHarmony 中的集成与适配文档
>
> 重点关注：Patch、特殊适配、构建集成、依赖关系

---

## 📚 文档导航（Documentation Navigation）

| 文档 | 描述 | 阅读时间 | 适用读者 |
|-------|-------|----------|----------|
| **SUMMARY.md** | 快速查找指南 | 5 分钟 | 所有人 |
| **01_Overview.md** | 库简介与定位 | 10 分钟 | 新手入门 |
| **02_Patches.md** ⭐ | Patch 详细分析（核心）| 60 分钟 | 开发者、维护者 |
| **03_Build_Integration.md** | 构建集成与配置 | 30 分钟 | 构建工程师 |
| **04_Usage_in_OH.md** | 依赖关系与使用方式 | 20 分钟 | 应用开发者 |
| **05_API_Differences.md** | API 差异与使用 | 40 分钟 | 应用开发者 |
| **06_Security.md** | 安全风险分析 | 20 分钟 | 安全工程师 |
| **_work/** | 工作文件（ASSESSMENT、NOTES、PLAN）| - | 深度研究者 |

---

## 🎯 快速查找（Quick Lookup）

### 我想了解什么？

| 问题 | 推荐文档 | 关键词 |
|------|----------|--------|
| curl 在 OH 中是什么？ | 01_Overview.md | 定位、作用 |
| OH 对 curl 做了哪些 Patch？ | 02_Patches.md | 诊断、性能、安全 |
| 如何在 OH 中编译 curl？ | 03_Build_Integration.md | BUILD.gn、配置头 |
| 哪些模块在使用 curl？ | 04_Usage_in_OH.md | ace_engine、media_foundation |
| 有哪些新增的 API？ | 05_API_Differences.md | CURLINFO、CURLOPT |
| 有哪些安全风险？ | 06_Security.md | CVE、Patch 攻击面 |

### 常见任务（Common Tasks）

| 任务 | 相关文档 | 估计时间 |
|------|----------|----------|
| 使用 curl 发起 HTTP 请求 | 01_Overview、05_API | 15 分钟 |
| 诊断网络连接问题 | 02_Patches、05_API | 20 分钟 |
| 集成新的 TLS 后端 | 03_Build | 30 分钟 |
| 升级 curl 版本 | 02_Patches、06_Security | 90 分钟 |
| 分析性能瓶颈 | 02_Patches | 30 分钟 |

---

## 🔍 深度分析路径（Deep Analysis Paths）

### Patch 工作流（2 小时）
```
1. 阅读 02_Patches.md → 理解所有 Patch 的目的和影响
2. 查看代码片段 → 理解实现细节
3. 参考 05_API_Differences.md → 了解 API 使用方式
4. 检查 06_Security.md → 评估安全影响
```

### 构建集成工作流（1 小时）
```
1. 阅读 03_Build_Integration.md → 理解构建选项
2. 查看 _work/ASSESSMENT.md → 了解架构
3. 参考 BUILD.gn → 理解依赖关系
```

### 依赖分析工作流（30 分钟）
```
1. 阅读 04_Usage_in_OH.md → 了解使用场景
2. 查看依赖图 → 理解模块关系
3. 分析使用方式 → 确定集成策略
```

---

## 📊 文档统计（Document Statistics）

### 总计（Total）
- **Markdown 文件数**: 7 个
- **总估计字数**: ~15,000 字
- **预估阅读时间**: 3.5 小时（完整阅读）

### 核心内容比例
| 类别 | 文档数 | 占比 |
|-------|--------|------|
| OH 特定内容 | 2 | 28.6% |
| 技术参考 | 4 | 57.1% |
| 工作文件 | 1 | 14.3% |

---

## 💡 使用提示（Usage Tips）

### 新手入门
1. **从 01_Overview.md 开始**：了解 curl 的基本概念
2. **关注 02_Patches.md**：这是最重要的文档，包含所有 OH 定制化内容
3. **按需阅读**：根据任务选择相关文档，不必全部阅读

### 开发者参考
1. **修改 Patch 前**：阅读 02_Patches.md 和 06_Security.md
2. **集成新功能**：参考 05_API_Differences.md 的使用示例
3. **构建问题**：查看 03_Build_Integration.md 的配置说明

### 维护者参考
1. **版本升级**：参考 02_Patches.md 的升级建议
2. **安全更新**：查看 06_Security.md 的 CVE 状态
3. **架构变更**：通过 _work/ASSESSMENT.md 了解整体结构

---

## 🔄 更新记录（Update Log）

### v1.0 - 2026-02-07
- 初始化文档结构
- 创建 7 个核心文档
- 完成 Phase 0 信息收集
- 添加导航和快速查找

### 未来更新计划
- 补充 API 使用示例
- 添加更多 OH 集成案例
- 更新 CVE 修复状态

---

## 📮 反馈与贡献（Feedback & Contributing）

### 文档问题？
- 查看 **_work/PLAN.md** 了解任务进度
- 检查 **_work/NOTES.md** 查看已知限制

### 想要贡献？
1. Fork 本仓库
2. 创建分支进行修改
3. 提交 Pull Request
4. 参考贡献指南

### 联系方式
- **Issue Tracker**: 通过 OpenHarmony 项目提 Issue
- **技术讨论**: 在相应技术社区讨论

---

## 📖 相关资源（Related Resources）

### 官方文档
- [curl 官方文档](https://curl.se/libcurl/c/)
- [Everything curl](https://everything.curl.dev/)

### OpenHarmony 资源
- [OpenHarmony 第三方库](https://gitee.com/openharmony/third_party_curl)
- [OpenHarmony 文档](https://docs.openharmony.cn/)

### 上游资源
- [curl GitHub](https://github.com/curl/curl)
- [PR #17743 - OpenHarmony SOVERSION 支持](https://github.com/curl/curl/pull/17743)

---

**文档维护**: OpenHarmony 第三方库 Wiki 维护团队
