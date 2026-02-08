# OpenHarmony SDK-JS 工程 Wiki

## 文档覆盖范围

本文档为 `interface/sdk-js` 仓库的工程 Wiki，旨在帮助开发者快速理解：

- 项目定位与目录结构
- API 声明文件组织方式
- 构建工具体系
- GN 构建配置
- 安全风险与最佳实践

## 文档结构

```
wiki/
├── README.md              # 本文档，说明与导航
├── SUMMARY.md            # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_API_Declarations.md # API 声明文件
├── 02_Build_Tools.md     # 构建工具链
├── 03_Build_Configuration.md # GN 构建配置
├── 04_Security_Review.md  # 安全评审
├── 05_Troubleshooting.md  # 常见问题
└── appendix/              # 附录
    └── Callgraphs.md      # 关键调用链
```

## 适用范围

| 角色 | 适用场景 |
|------|---------|
| SDK 开发者 | 新增/修改 API 声明文件 |
| 构建工程师 | 理解 SDK 构建流程 |
| 安全工程师 | 安全审计与风险评估 |
| 工具开发者 | 开发/维护构建工具 |

## 排除范围

- **测试相关内容**: 本 Wiki 不引用测试代码 (`test/`, `*_test.*`)
- **运行时实现**: 本仓库仅包含声明文件，不包含 C/C++ 实现
- **第三方仓库**: 仅描述本仓库相关工具

## 更新方式

### 何时更新 Wiki

当发生以下变更时，需同步更新 Wiki：

1. 新增/删除/重命名 API 模块
2. 新增/修改构建工具
3. 修改 GN 构建配置
4. 发现新的安全风险
5. 更新工具调用方式

### 更新方式

```bash
# 1. 克隆仓库
git clone <sdk-js-repo>

# 2. 编辑对应文档
#    - 修改 wiki/*.md 文件
#    - 添加证据链接 (文件路径 + 行号)

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for xxx"
git push
```

## 生成信息

| 属性 | 值 |
|------|-----|
| 仓库 | `interface/sdk-js` |
| 生成时间 | 2026-02-06 |
| 生成方式 | 自动扫描 + 人工整理 |
| 文档版本 | 1.0 |

## 相关链接

- **官方仓库**: https://gitee.com/openharmony/interface_sdk-js
- **OpenHarmony**: https://www.openharmony.cn/
- **API 参考**: https://developer.harmonyos.com/
