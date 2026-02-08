# OpenHarmony system_resources Wiki 使用指南

## 概述

本文档为 OpenHarmony `system_resources` 子模块的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、构建系统以及安全考量。

**适用范围**: OpenHarmony Globalization 子系统 - system_resources 部件

## 覆盖范围

| 文档 | 状态 | 说明 |
|------|------|------|
| [README.md](README.md) | ✅ | 本文档，Wiki 使用指南 |
| [SUMMARY.md](SUMMARY.md) | ✅ | 全站导航与阅读路线 |
| [00_Overview.md](00_Overview.md) | ✅ | 项目定位与核心能力 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | ✅ | 目录结构与模块职责 |
| [02_Architecture.md](02_Architecture.md) | ✅ | 架构设计与组件关系 |
| [03_Build_System.md](03_Build_System.md) | ✅ | GN 构建与编译产物 |
| [04_Resources.md](04_Resources.md) | ✅ | 系统资源说明 |
| [05_Security.md](05_Security.md) | ✅ | 安全风险评审 |
| [06_FAQ.md](06_FAQ.md) | ✅ | 常见问题 |

## 不包含内容

- **测试相关**: 本 Wiki 不引用任何测试代码作为证据 (`test/`, `tests/`, `*_test.*`)
- **N-API**: 本模块不提供 JS/N-API 接口
- **IPC/SA**: 本模块不涉及 ServiceAbility 或 IPC 通信
- **运行时逻辑**: 本模块为静态资源，不包含可执行代码

## 文档更新方式

### 何时需要更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. 新增/删除/重命名关键文件或目录
2. 修改 GN 构建配置（新增 target、变更依赖等）
3. 新增系统资源（字体、权限、字符串等）
4. 修改安全相关配置

### 更新流程

```bash
# 1. 克隆仓库
git clone https://gitee.com/openharmony/resources.git
cd resources

# 2. 编辑 wiki 目录下的相应文档
#    使用证据链原则：每个结论需标注文件路径和行号

# 3. 提交变更
git add wiki/
git commit -m "docs: update wiki for [变更描述]"
```

### 证据链原则

所有关键结论必须能够追溯到代码证据：

```
正确示例:
- 系统字体安装在 `fonts/` 目录 [证据: BUILD.gn:28-36]
- 权限定义在 module.json 中定义 [证据: systemres/main/module.json:17-1301]

错误示例:
- 系统字体安装在 fonts 目录 (无证据)
```

## 阅读建议

### 新人阅读路线

1. **概览**: 阅读 [00_Overview.md](00_Overview.md) 了解项目定位
2. **结构**: 阅读 [01_Directory_Structure.md](01_Directory_Structure.md) 熟悉目录布局
3. **构建**: 阅读 [03_Build_System.md](03_Build_System.md) 理解构建配置
4. **资源**: 阅读 [04_Resources.md](04_Resources.md) 了解系统资源
5. **安全**: 阅读 [05_Security.md](05_Security.md) 把握安全要点

### 快速查找

使用 `SUMMARY.md` 作为导航入口，查看所有文档链接。

## 版本信息

| 属性 | 值 |
|------|-----|
| Wiki 生成时间 | 2026-02-06 |
| 适用版本 | OpenHarmony 4.0+ |
| 所属子系统 | Globalization |
| 部件名称 | system_resources |

## 反馈与贡献

如发现 Wiki 内容有误或遗漏，请通过以下方式反馈：

1. 在 [Gitee Issues](https://gitee.com/openharmony/resources/issues) 中提出
2. 提交 Pull Request 修改 `wiki/` 目录下的文档

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Globalization 子系统](https://gitee.com/openharmony/docs/blob/master/en/readme/globalization.md)
- [global_resmgr_standard](https://gitee.com/openharmony/global_resmgr_standard)
