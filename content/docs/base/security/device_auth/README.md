# 设备互信认证模块 Wiki

> 文档版本：4.0.2
> 生成时间：2026-02-06
> 所属仓库：`base/security/device_auth`
> 模块名：`@ohos/device_auth`

---

## 文档概述

本文档是 OpenHarmony 设备互信认证模块的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建配置，并提供安全风险分析。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✓ 已覆盖 | 定位、核心功能、运行环境 |
| 目录结构 | ✓ 已覆盖 | 模块职责划分 |
| 架构设计 | ✓ 已覆盖 | 组件图、数据流、时序图 |
| N-API 接口 | ✓ 已覆盖 | JS API 清单、调用链 |
| Inner API | ✓ 已覆盖 | C/C++ 模块接口 |
| GN 构建 | ✓ 已覆盖 | targets、依赖、产物 |
| 安全评审 | ✓ 已覆盖 | 攻击面、风险点 |
| 常见问题 | ✓ 已覆盖 | 构建/运行/调试 |

### 文档结构

```
wiki/
├── README.md              # 文档首页（本文档）
├── SUMMARY.md             # 全站导航
├── 01_Overview.md         # 项目概览
├── 02_Architecture.md     # 架构设计
├── 03_API_Reference.md    # N-API 参考
├── 04_Inner_API.md        # Inner API 参考
├── 05_Build_Config.md     # 构建配置
├── 06_Security_Review.md  # 安全风险评审
├── 07_Troubleshooting.md    # 常见问题
└── appendix/
    ├── Callgraphs.md      # 关键调用链
    └── Config_Flags.md    # 配置开关
```

### 阅读建议

**新人入门**：
1. `01_Overview.md` - 了解项目定位
2. `02_Architecture.md` - 理解整体架构
3. `03_API_Reference.md` - 学习 API 使用

**开发者**：
1. `03_API_Reference.md` - 接口调用
2. `04_Inner_API.md` - 模块集成
3. `05_Build_Config.md` - 构建配置

**安全相关**：
1. `06_Security_Review.md` - 安全风险分析

### 证据来源

所有关键结论均可在仓库中找到直接证据：
- **文件路径**：`wiki/03_API_Reference.md` 中标注了接口定义位置
- **符号名**：函数、类、宏定义均标注完整
- **代码片段**：关键逻辑配有最小必要代码示例

### 如何更新文档

当代码变更影响 Wiki 内容时，请同步更新相关文档：

1. **API 变更**：更新 `03_API_Reference.md` 或 `04_Inner_API.md`
2. **架构变更**：更新 `02_Architecture.md` 和 `appendix/Callgraphs.md`
3. **构建变更**：更新 `05_Build_Config.md` 和 `appendix/Config_Flags.md`
4. **安全相关**：更新 `06_Security_Review.md`

### 相关链接

- **代码仓库**：https://gitee.com/openharmony/security_device_auth
- **官方文档**：https://gitee.com/openharmony/docs/blob/master/zh-cn/security/device_auth.md
- **Issue 反馈**：https://gitee.com/openharmony/security_device_auth/issues

---

*本文档由 Wiki 生成工具自动生成，最新更新时间：2026-02-06*
