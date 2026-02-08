# Mechbody Controller Wiki

## 项目概述

OpenHarmony Mechbody Controller（机械设备控制器）是一个系统服务，提供发现和控制兼容机械设备的 API，实现目标设备的无缝连接和精确控制。

**仓库路径**: `//foundation/distributedhardware/mechbody_controller`

## 覆盖范围

本文档覆盖以下内容：

| 分类 | 覆盖范围 |
|------|----------|
| **N-API 接口** | JavaScript/ArkTS API（`@ohos.mechbodyController`） |
| **IPC/SystemAbility** | SA 8550 架构、命令码、回调机制 |
| **内部模块** | Controller/Connect/Motion/Transport 四大模块 |
| **GN 构建** | Targets、依赖、编译产物 |
| **安全机制** | 权限模型、输入校验、信任边界 |

**不包含**：测试代码（fuzztest/unittest）、vendor 南向协议实现细节

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                    # 全站导航
├── 01_Overview.md               # 项目概述
├── 02_Architecture.md           # 架构说明
├── 03_NAPI_Reference.md         # N-API 接口参考
├── 04_Inner_API.md              # 内部模块接口
├── 05_Build_Targets.md          # GN 构建产物
├── 06_Security_Review.md        # 安全风险评审
├── 07_Troubleshooting.md       # 问题排查指南
└── appendix/
    ├── Callgraphs.md            # 关键调用链
    └── Config_Flags.md         # 配置开关
```

## 阅读路线

**新人入门**：
1. `01_Overview.md` → 了解项目定位
2. `02_Architecture.md` → 理解系统架构
3. `03_NAPI_Reference.md` → 学习 API 使用

**开发者进阶**：
1. `04_Inner_API.md` → 内部模块接口
2. `05_Build_Targets.md` → 构建配置
3. `06_Security_Review.md` → 安全注意事项

## 更新方式

本文档基于代码自动生成 + 人工校验。

**更新时机**：
- 新增 N-API 方法时
- 修改 IPC 命令码时
- 变更构建配置时
- 新增安全机制时

**更新步骤**：
```bash
# 1. 进入仓库
cd /path/to/mechbody_controller

# 2. 更新 wiki 内容（修改对应 .md 文件）

# 3. 验证文档链接
git diff wiki/

# 4. 提交更改
git add wiki/
git commit -m "docs: update wiki for [变更说明]"
```

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于当前仓库 HEAD
- **生成工具**: OpenHarmony Wiki Generator
