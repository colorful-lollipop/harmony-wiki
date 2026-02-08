# napi-generator Wiki 使用指南

> 本 Wiki 由代码自动分析生成，目标是帮助新人快速完整理解 napi-generator 项目。

## 覆盖范围

### ✅ 已覆盖

| 章节 | 内容 | 证据来源 |
|------|------|----------|
| 项目定位 | N-API 代码生成工具集 | `README.md`, `src/cli/*/` |
| 核心工具 | dts2cpp, h2sa, h2dtscpp 等 6 个工具 | 代码探索 |
| 目录结构 | src/cli/*, examples/* 完整结构 | 目录遍历 |
| N-API 架构 | 模块注册、类型转换、异步支持 | dts2cpp/gen/* |
| SA 框架生成 | proxy/stub, MessageParcel | h2sa/src/gen/* |
| GN 构建 | BUILD.gn 配置分析 | 构建文件分析 |
| 安全评审 | 威胁模型与风险点 | 代码安全分析 |
| 攻击面分析 | 输入/文件系统/命令执行攻击面 | `05_AttackSurface.md` |
| 代码导航 | 功能到文件路径快速索引 | `03_CodeMap.md` |

### ⏳ 待完善

| 章节 | 预计内容 | 依赖 |
|------|----------|------|
| 详细 API 参考 | 每个工具的完整 API 参数表 | 继续深入代码 |
| 编译产物路径 | 各工具的输出路径验证 | 构建测试 |

## 更新方式

### 何时需要更新 Wiki

1. 新增核心工具模块 (`src/cli/` 新增目录)
2. 修改现有工具的入口或核心生成逻辑
3. 添加新的 N-API 函数类型支持
4. 更新依赖的 OpenHarmony 仓库版本
5. 修改构建系统 (BUILD.gn, package.json)

### 如何更新

```bash
# 1. 进入项目根目录
cd /Volumes/lexar/code/d/work/oh/napi_generator

# 2. 运行 Wiki 生成工具 (TODO: 如有)
./scripts/generate-wiki.sh

# 3. 或手动更新对应章节
# - 修改 src/cli/* 的描述 → 更新 04_NAPI_Reference.md
# - 修改构建配置 → 更新 06_Build_System.md
# - 添加安全考量 → 更新 07_Security_Review.md
```

## 文档结构

```
wiki/
├── README.md              # 本文件，使用指南
├── SUMMARY.md             # 全站导航，新人阅读路线
├── index.md              # 项目首页/概览
├── 01_Overview.md        # 项目定位、边界、核心能力
├── 02_Directory_Structure.md  # 目录结构与模块职责
├── 03_Architecture.md    # 架构说明（含 Mermaid 图）
├── 03_CodeMap.md         # 代码导航图（功能→文件索引）
├── 04_NAPI_Reference.md # 对外 N-API 文档
├── 05_Internal_API.md    # 内部 API 与模块接口
├── 05_AttackSurface.md   # 攻击面分析（安全研究）
├── 06_Build_System.md    # GN Targets 与编译产物
├── 07_Security_Review.md # 安全风险评审
├── 08_Troubleshooting.md # 常见问题与定位
├── _work/                # 工作文档
│   ├── ASSESSMENT.md     # 项目评估
│   ├── NOTES.md          # 代码证据记录
│   └── PLAN.md           # 任务计划
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md    # 关键配置项
```

## 阅读建议

### 新人路线

1. **5 分钟概览**: `index.md` → `01_Overview.md`
2. **10 分钟结构**: `02_Directory_Structure.md` → `03_Architecture.md`
3. **30 分钟深入**: `04_NAPI_Reference.md` (选择感兴趣的模块)
4. **实践**: 查看 `examples/napitutorials/` 示例代码

### 按角色阅读

| 角色 | 推荐阅读顺序 |
|------|-------------|
| N-API 开发者 | 01 → 02 → 04 → examples |
| SA 服务开发者 | 01 → 02 → 03 → h2sa 相关章节 |
| 工具贡献者 | 02 → 03 → 05 → 06 |
| 安全审计 | 05 → 07 → 03 → 04 |

## 生成信息

- **生成时间**: 2026-02-07
- **代码版本**: 基于当前仓库 `master` 分支
- **生成工具**: Sisyphus Wiki Agent
- **证据追溯**: 所有关键结论均标注代码证据路径

## 反馈与贡献

- 发现文档错误？请提交 Issue
- 缺少重要章节？请提交 PR
- 有更好的示例？请贡献到 `examples/`

---

[返回 SUMMARY.md](SUMMARY.md)
