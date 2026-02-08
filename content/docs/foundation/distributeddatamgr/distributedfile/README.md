# Distributed File Wiki

> OpenHarmony 分布式文件管理子系统文档

## 覆盖范围

本文档覆盖 **Distributed File（分布式文件）子系统**的以下方面：

| 类别 | 覆盖 | 说明 |
|------|------|------|
| 项目定位 | ✅ | 架构、能力边界、运行环境 |
| 目录结构 | ⚠️ | 基于 README 描述的标准结构 |
| N-API | ✅ | @OHOS.distributedfile.fileio, @system.file |
| 构建系统 | ⚠️ | GN 需代码仓库完整 |
| 安全评审 | ✅ | 基于架构分析 |
| 运行时 | ⚠️ | 需代码验证 |

**符号说明**:
- ✅ 完全覆盖
- ⚠️ 部分覆盖（基于推断或需代码补充）
- ❌ 未覆盖（代码缺失）

## 更新方式

1. **代码变更时**: 同步更新对应章节
2. **N-API 变更**: 更新 `01_APIs.md`
3. **架构调整**: 更新 `00_Overview.md` 和 `03_Build_System.md`

### 贡献指南

```bash
# 1. 克隆仓库
git clone https://gitee.com/openharmony/distributeddatamgr_file.git

# 2. 创建分支
git checkout -b wiki-update

# 3. 编辑 wiki/*.md 文件

# 4. 提交（仅限 wiki 目录变更）
git commit -a -m "docs: update wiki/xx.md"

# 5. PR 到主分支
```

## 文档版本

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本（基于 README） |

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [Distributed File 源码](https://gitee.com/openharmony/distributeddatamgr_file)
- [相关仓库列表](./00_Overview.md#相关仓库)
